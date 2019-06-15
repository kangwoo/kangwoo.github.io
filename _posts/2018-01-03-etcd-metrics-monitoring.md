---
title:  "etcd metrics monitoring"
classes: wide
date: 2018-01-03T20:13:00+09:00
categories: [devops, etcd]
tags: [etcd]
---

# etcd 모니터링 하기 

## Metrics endpoint
각각의 etcd 서버는 클라이언트 포트의 `/metrics` 경로를 이용해서 메트릭을 출력해 줍니다.
(클라이언트 기본 포트는 2379 입니다.)
```bash
$ curl http://127.0.0.1:2379/metrics

...
tcd_disk_wal_fsync_duration_seconds_count 4.9931728e+07
# HELP etcd_grpc_proxy_cache_hits_total Total number of cache hits
# TYPE etcd_grpc_proxy_cache_hits_total gauge
etcd_grpc_proxy_cache_hits_total 0
# HELP etcd_grpc_proxy_cache_keys_total Total number of keys/ranges cached
# TYPE etcd_grpc_proxy_cache_keys_total gauge
etcd_grpc_proxy_cache_keys_total 0
# HELP etcd_grpc_proxy_cache_misses_total Total number of cache misses
# TYPE etcd_grpc_proxy_cache_misses_total gauge
etcd_grpc_proxy_cache_misses_total 0
# HELP etcd_grpc_proxy_events_coalescing_total Total number of events coalescing
# TYPE etcd_grpc_proxy_events_coalescing_total counter
etcd_grpc_proxy_events_coalescing_total 0
# HELP etcd_grpc_proxy_watchers_coalescing_total Total number of current watchers coalescing
# TYPE etcd_grpc_proxy_watchers_coalescing_total gauge
etcd_grpc_proxy_watchers_coalescing_total 0
# HELP etcd_mvcc_db_total_size_in_bytes Total size of the underlying database physically allocated in bytes.
# TYPE etcd_mvcc_db_total_size_in_bytes gauge
etcd_mvcc_db_total_size_in_bytes 1.0145792e+07
# HELP etcd_mvcc_db_total_size_in_use_in_bytes Total size of the underlying database logically in use in bytes.
# TYPE etcd_mvcc_db_total_size_in_use_in_bytes gauge
etcd_mvcc_db_total_size_in_use_in_bytes 4.77184e+06
...
```

하지만 TLS 인증 기반으로 etcd를 설치하였을 경우, 단순 메트릭을 보기 위해서도 인증서가 필요합니다. 
이런 불편한 때문에 v3.3.0 부터 별도의 `/metrics` 엔드포인트(endpoint)를 지정할 수 있는 기능이 생겼습니다.
바로 `--listen-metrics-urls` 플래그 입니다.
이 플래그를 이용해서, 별도의 엔드포인트를 지정할 수 있습니다.

`--listen-metrics-urls=http://0.0.0.0:2381` 플래그를 etcd 실행 할때 추가하면 `2381` 포트로 `/metrics`과  `/health` 를 조회해 볼 수 있습니다.
또 다른 방법으로는 환경 변수나 설정 파일에 `ETCD_LISTEN_METRICS_URLS` 값을 설정하는 것입니다.

## Health endpoint
`/health` 엔드포인트를 이용하면 etcd 서버의 상태를 알 수 있습니다.
```bash
$ curl http://127.0.0.1:2379/health
{"health":"true"}
```

## Prometheus
프로메테우스 설정 파일에, etcd 스크랩 설정을 추가하면, 프로메테우스가 etcd의 메트릭 엔드포인트에 접근하여, 메트릭 데이터를 수집해 갑니다.

```
scrape_configs:
  - job_name: etcd
    static_configs:
    - targets: ['x.x.x.x:2381','y.y.y.y:2381','z.z.z.z:2381']
```


# 참고 문서
- <https://github.com/etcd-io/etcd/blob/master/Documentation/op-guide/monitoring.md>
