---
title:  "쿠버네티스에 메트릭 서버(metrics-server) 설치하기"
classes: wide
date: 2019-05-30T20:34:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, metrics-server]
---

쿠버네티스 v1.11부터 heapster가 deprecated 되었습니다 (자세한 내용은 [문서](https://github.com/kubernetes-retired/heapster/blob/master/docs/deprecation.md)를 참고 바랍니다.)
그래서 HPA(horizontal pod autoscaler)나 kubectl top 명령어를 사용하라면 metrics-server를 사용해야 합니다.

# Metrics Server 란?
Metrics Server는 클래스터 전체의 리소스 사용 데이터를 어그리게이션합니다.
각 노드에 설치된 kublet을 통해서 노드나 컨테인너의 CPU나 메모리 사용량 같은 메트릭을 수집합니다.


# 배포 방법

## 요구사항
Metrics Server를 배포하려면, 쿠버네티스 클러스터에 어그리게이션 레이어가 활성화되어 있어야합니다.
대부분은 기본적으로 활성화되어 있습니다. 혹시 직접 활성화해야하는 경우라면, 아래 링크를 참조하세요.
[https://kubernetes.io/docs/tasks/access-kubernetes-api/configure-aggregation-layer]


## 배포
Metrics Server git 저장소([https://github.com/kubernetes-incubator/metrics-server])를 복제(clone)하고, 다음과 같이 배포하세요.

```bash
git clone https://github.com/kubernetes-incubator/metrics-server.git
cd metrics-server
kubectl apply -f deploy/1.8+/
```

kubectl을 이용해서 적용하면, v1beta1.metrics.k8s.io 라는 apiservce가 생성되고, metrics-server 라는 디플로이먼트와 서비스가 생성됩니다.

설치가 잘 진행되었다면, 다음과 같이 apiservice를 확인할 수 있습니다.
```bash
kubectl get apiservices | grep metrics

v1beta1.metrics.k8s.io                 2019-05-31T06:24:16Z
```

그리고 디플로이먼트와 서비스도 확인할 수 있습니다.
```bash
kubectl -n kube-system get deploy,svc | grep metrics-server

deployment.extensions/metrics-server               1         1         1            1           1h
service/metrics-server               ClusterIP   10.96.106.172   <none>        443/TCP         1h

```


```kubectl top node``` 명령어를 사용하면, 노드의 사용현황을 볼 수 있습니다.

```
$ kubectl top node
NAME                           CPU(cores)   CPU%      MEMORY(bytes)   MEMORY%
kube-node-001                  9736m        24%       99265Mi         38%
kube-node-002                  12060m       30%       115793Mi        44%
kube-node-003                  12349m       30%       117894Mi        45%
kube-master-001                248m         0%        20110Mi         7%
kube-master-002                289m         0%        7035Mi          2%
kube-master-003                268m         0%        7087Mi          2%
```


### Troubleshooting

metrics-server 의 포드에 다음과 같은 에러 로그가 있을 경우, 파라메터를 추가해야합니다.
```bash
E0531 08:27:57.249840       1 manager.go:111] unable to fully collect metrics: [unable to fully scrape metrics from source kubelet_summary:kube-xxxx: unable to fetch metrics from Kubelet kube-xxx (kube-xxx): Get https://kube-xxx:10250/stats/summary/: dial tcp: lookup kube-xxx on xx.xx.xx.xx:x: no such host]
```
```bash
E0531 08:34:42.750007       1 manager.go:111] unable to fully collect metrics: [unable to fully scrape metrics from source kubelet_summary:kube-xxxx: unable to fetch metrics from Kubelet kube-xxx (xx.xx.xx.xx): Get https://xx.xx.xx.xx:10250/stats/summary/: x509: cannot validate certificate for xx.xx.xx.xx because it doesn't contain any IP SANs]
```
```dial tcp: lookup kube-xxx on xx.xx.xx.xx:x: no such host``` 에러인 경우에는 ```kubelet-preferred-address-types``` 파라메터를,
```x509: cannot validate certificate for xx.xx.xx.xx because it doesn't contain any IP SANs```인 경우에는 ```kubelet-insecure-tls``` 파라메터를 사용하면 된다.

metrics-server-deployment.yaml 파일을 편집해서, ```image: k8s.gcr.io/metrics-server-amd64:v0.3.3``` 밑에다 아래 파라메터를 추가하면 됩니다.
```
        command:
          - /metrics-server
          - --kubelet-preferred-address-types=InternalIP
          - --kubelet-insecure-tls
```
