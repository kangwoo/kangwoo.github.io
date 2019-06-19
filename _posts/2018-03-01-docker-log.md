---
title:  "Docker 로그 관리"
classes: wide
date: 2018-02-06T13:15:00+09:00
categories: [devops, docker]
tags: [docker, logrotate]
---

도커(docker)는 로깅 드라이버(logging driver) 통해, 로그를 남기게 되어 있습니다.
로깅 드라이버의 기본 값을 `json-file`입니다. 즉, 로그를 json 형식으로 파일로 저장하게 됩니다.

아래 명령어를 실행하면, 해당 도커의 로깅 드라이버가 뭔지 알 수 있습니다.
```bash
$ docker info --format '{{.LoggingDriver}}'
json-file
```
`json-file` 로깅 드라이버를 사용하는 경우, 시간이 지날 수록 로그 파일이 쌓이기 때문에 주기적으로 파일을 삭제해줘야합니다.
주기적으로 파일을 삭제하는 방법은, 도커 데몬의 설정을 변경하거나, `logrotate`를 이용하는 것입니다.

## 도커 데몬 설정 파일 변경하기
`/etc/docker/` 디렉토리에 있는 `daemon.json` 파일에 아래와 같은 내용을 추가해 주면 됩니다.

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "10m",
    "max-file": "3" 
  }
}
```
도커 재시작하면 변경 사항이 반영됩니다.


## logrotate 이용하기
`logrotate`는 로그를 관리하기 위해 사용되는 범용툴입니다. 서버에 설치가 안되어 있다면, 설치가 필요합니다.
아래와 같이 컨테이너 로그를 정리하는 설정 파일을 추가해 주면 됩니다.
```bash
cat > /etc/logrotate.d/container << EOF
/var/lib/docker/containers/*/*.log {
    rotate 100
    copytruncate
    missingok
    notifempty
    compress
    maxsize 100M
    maxage 30
    daily
    dateext
    dateformat -%Y%m%d-%s
    create 0644 root root
}
EOF
```


# 참고 문서
- <https://docs.docker.com/config/containers/logging/json-file/>
