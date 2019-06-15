---
title:  "쿠버네티스 메트릭 서버 인증 실패"
classes: wide
date: 2019-05-31T20:51:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, metrics-server]
---

쿠버네티스 v1.11.x에서 metrics-server를 설치하였으나, 한개의 마스터 서버에서만 정상적으로 작동하는 문제가 발생하였습니다. (3개의 마스터 서버로 HA 구성 상태)

```kubectl top node``` 명령어를 사용하면, 약 1/3 확률로 정상 응답을 하고, 나머지는 아래와 같이 권한이 없다는 메시지가 나옵니다.

```
F0531 10:41:33.286003 52081 helpers.go:119] error: You must be logged in to the server (Unauthorized)
```
그래서 kube-apiserver를 로드 밸런서 없이, 마스터 서버 아이피로 직접 연결해서 테스트해보았는데, 단 1개의 마스터 서버만  정상 응답하고, 나머지는 권한이 없다는 메시지가 나왔습니다.

metrics-server는 [어그리게이션 레이어]([https://kubernetes.io/docs/tasks/access-kubernetes-api/configure-aggregation-layer])를 사용하는데, 
kube-apiserver와 metrics-server 간에 인증서로 상호 연동을 합니다.
문제는 해당 쿠버네티스 클러스터를 설치하는 과정에서, 이 인증서를 마스터 서버마다 따로 생성을 해버려서, 한군데만 정상적으로 작동한다는 것이였습니다.
(kubeadm을 이용해서 설치하였는데, 해당 인증서를 복사하지 않고 설차하여서, 자동으로 생성된 경우입니다.)
그래서, 인증서를 다시 생성한 후, 각 마스터 서버에 복사하고, kube-api-server를 재시작 하였고, metrics-server를 재시작 해서 문제를 해결하였습니다.

쿠버네티스 버전이 1.11이기 때문에 kubeadm.k8s.io/v1alpha1 형식으로 파일을 만들어야 했습니다. 
로드 밸런서 도메인 이름과, IP 주소, 각 마스터 서버의 IP 주소를 apiServerCertSANs 에 추가하여 kube-config.yaml 파일을 생성하였습니다.
```
apiVersion: kubeadm.k8s.io/v1alpha1
kind: MasterConfiguration
api:
  advertiseAddress: xx.xx.xx.xx
...
apiServerCertSANs:
  - lb.xx.xx.xx
  - lb.xx.xx.xx
  - ma.xx.xx.xx
  - ma.xx.xx.xx
  - ma.xx.xx.xx

```

그리고 ```kubeadm```을 실행해서 front-proxy 인증서를 생성하였습니다.

```bash
kubeadm alpha phase certs front-proxy-ca --config kube-config.yaml
kubeadm alpha phase certs front-proxy-client --config kube-config.yaml

```
위의 명령을 실행하면 해당 파일들이 생성됩니다.
- front-proxy-ca.crt
- front-proxy-ca.key
- front-proxy-client.crt
- front-proxy-client.key

생성한 파일들을, 나머지 마스터 서버에 복사하고, kube-api-server를 재시작합니다.

모든 마스터 서버의 작업이 끝났으면, metrics-server를 재시작합니다.
