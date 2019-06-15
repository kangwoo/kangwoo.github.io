---
title:  "Unable to get metrics for resource cpu"
classes: wide
date: 2019-05-29T19:11:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, hpa]
---

# 상황
쿠버네티스 v1.11.x에서 HPA를 사용하려고 했으나, 에러가 발생하였습니다.
## 에러 메시지
```
Warning  FailedGetResourceMetric       3m (x21 over 13m)  horizontal-pod-autoscaler  unable to get metrics for resource cpu: unable to fetch metrics from resource metrics API: the server could not find the requested resource (get pods.metrics.k8s.io)
```

# 해결 방법
위의 에러 메시지는는 metrics-server가 설치되어 있지 않아서 생기는 것입니다.
metrics-server 설치 문서를 참고해서 설치 하시기 바랍니다.
