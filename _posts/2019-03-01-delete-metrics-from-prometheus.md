---
title:  "프로메테우스 시계열 데이터 삭제하기"
classes: wide
date: 2019-03-01T13:43:00+09:00
categories: [devops, prometheus]
tags: [prometheus]
---

프로메테우스([Prometheus](https://prometheus.io/))에서 시계열(Time series) 데이터를 삭제하는 방법에 대해서 알아보겠습니다.

디스크 용량이 부족하거나 필요없는 시계열 데이터가 있을 경우, 데이터를 삭제할 필요가 있습니다.
프로메테우스의 시계열 데이터를 삭제하려면 관리 API를 사용해야합니다. 이 관리 API는 기본적으로 비활성화 되어 있습니다.


### 관리 API 활성화 하기
이 기능을 활성화 하기 위해서는, 프로메테우스를 기동할때 ```--web.enable-admin-api``` 파라메터를 추가해줘야합니다.


### 시계열 데이터 삭제하기
시계열 데이터를 삭제하는 API는 ```delete_series``` 입니다.
아래 명령어를 실행하면, 레이블과 일치하는 시계열 데이터를 모두 삭제할 수 있습니다.
```bash
curl -X POST \
	-g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={foo="bar"}'
```
예를 들어 잡(job)이나 인스턴스(instance)에 해당하는 데이터를 삭제하려면, 다음과 같이 실행하면 됩니다.
```bash
curl -X POST \
	-g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={job="node_exporter"}'

curl -X POST \
	-g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={instance="172.22.0.1:9100"}'
```
전체 시계열 데이터를 모두 삭제하려면, 다음과 같이 실행하면 됩니다.
```bash
curl -X POST \
	-g 'http://localhost:9090/api/v1/admin/tsdb/delete_series?match[]={__name__=~".+"}'
```

참고로 ```delete_series``` API를 호출해도, 데이터가 즉시 삭제되지는 않는다. 실제 데이터는 디스크에 남아있고, 다음번 컴팩션(compaction)이 실행될때 정리된다.
바로 정리 하고 싶다면, 다음과 같이 ```clean_tombstones``` API를 호출하면 된다.
```bash
curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/clean_tombstones'
```
