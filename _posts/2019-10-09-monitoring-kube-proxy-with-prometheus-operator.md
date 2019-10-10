---
title:  "Prometheus Operator로 kube-proxy 모니터링 하기"
classes: wide
date: 2019-10-09T17:41:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, kube-proxy]
---

## kube-proxy 모니터링 하기
`kube-proxy`도 `/metrics`라는 매트릭 엔드 포인트를 제공한다.
하지만 기본 설정값이 `127.0.0.1:10249`이기 때문에, 외부에서 접근이 안된다.

그래서 `prometheus`에서 수집하려고 하면, 접근이 안되서 문제가 발생한다.

## 설정 변경하기

```bash
$ kubectl -n kube-system edit cm/kube-proxy 
## Change from
    metricsBindAddress: 127.0.0.1:10249
## Change to
    metricsBindAddress: 0.0.0.0:10249
```

물론 `0.0.0.0:10249`로 변경하고, 모든 곳에서 다 접근이 가능하기 때문에,
보안이 취약한 곳이라면 사용하지 않는것이 좋다.

설정을 변경하고, `kube-proxy`를 재시작하면 적용이 된다.

```bash
kubectl -n kube-system delete pod -l k8s-app=kube-proxy 
```
