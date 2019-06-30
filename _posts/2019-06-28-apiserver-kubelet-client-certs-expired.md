---
title:  "kubernetes 인증서 만료"
classes: wide
date: 2019-06-28T20:51:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes]
---


# 넋두리
한동안 버려두었던 쿠버네티스 클러스터를 사용할이 생겨, `kubectl`를 이용해서 명령어를 날렸다. 그런데 작동하지 않는다.
그동안 소외받았던 서러움을 이해하지 못하는것은 아니지만, 왜 갑자기 안되는 것일까? 
`-v=8` 파라메터를 추가해서 명령어를 실행해 보았지만, 결과는 난데없는 `Unauthorized` 에러.

```bash
$ kubectl get node -v=8
...
I0628 12:01:34.519625   13282 round_trippers.go:411] Response Headers:
I0628 12:01:34.519632   13282 round_trippers.go:414]     Content-Type: application/json
I0628 12:01:34.519638   13282 round_trippers.go:414]     Content-Length: 129
I0628 12:01:34.519653   13282 round_trippers.go:414]     Date: Fri, 28 Jun 2019 03:01:34 GMT
I0628 12:01:34.520370   13282 request.go:942] Response Body: {"kind":"Status","apiVersion":"v1","metadata":{},"status":"Failure","message":"Unauthorized","reason":"Unauthorized","code":401}
I0628 12:01:34.521041   13282 cached_discovery.go:111] skipped caching discovery info due to Unauthorized
...
``` 

"내가 짤렸나"라는 생각이 순간적으로 머리를 스쳐갔지만, 그건 어디까지나 기우일 뿐이겠지.
인증을 담당하는 OIDC Provder가 문제가 생겼나 의심해보았지만, 다른 쿠버네티스 클러스터는 잘 작동했기 때문에 그 문제는 아닌거 같았다.
그렇다면... 이놈의 클러스터가 맛이 갔구나!!!

## 로그 조회
`kubectl`을 사용할 수 없어서, 어쩔 수 없이 마스터 서버에 직접 로그인해서, `docker` 명령어로 api-server의 로그를 조회해 보았다.

```bash
# docker logs fdd294139c9c
...
E0627 12:16:55.724051       1 authentication.go:62] Unable to authenticate the request due to an error: [x509: certificate has expired or is not yet valid, x509: certificate has expired or is not yet valid]
E0627 12:16:55.725487       1 authentication.go:62] Unable to authenticate the request due to an error: [x509: certificate has expired or is not yet valid, x509: certificate has expired or is not yet valid]
E0627 12:16:55.726274       1 authentication.go:62] Unable to authenticate the request due to an error: [x509: certificate has expired or is not yet valid, x509: certificate has expired or is not yet valid]
```

<br/>
덤으로 `kubectl`의 로그도 조회해 보았다.

```bash
# ournalctl -u kubelet -f
...
Jun 28 12:18:14 kube-master002-xxx kubelet[145630]: E0628 00:02:14.684236  145630 server.go:222] Unable to authenticate the request due to an error: x509: certificate has expired or is not yet valid
Jun 28 12:18:20 kube-master002-xxx kubelet[145630]: E0628 00:02:20.928108  145630 server.go:222] Unable to authenticate the request due to an error: x509: certificate has expired or is not yet valid
Jun 28 12:18:24 kube-master002-xxx kubelet[145630]: E0628 00:02:24.683814  145630 server.go:222] Unable to authenticate the request due to an error: x509: certificate has expired or is not yet valid
```

`certificate has expired` 라는 에러 메시지를 보자, 직감적(?)으로 인증서에 문제가 있다는 것을 알 수 있었다.
<br/>
아... 벌써 1년이 지난것인가? 온갖 상념이 내 머리속을 갉아먹기 시작했다. 작년 이때쯤 쿠버네티스에 손을 대기 시작하였고, 그때 설치한 클러스터가 아직도 죽지 않고 살아서 생명을 유지하고 있는것이었다.
미안하다. 그동안 너무 무심했구나.

## 인증서 만료일
인증서 만료일을 확인하기 위해서 마스터 서버에 접속하였다. 쿠버네티스 관련 인증서가 있는 `/etc/kubernetes/pki` 디렉토리로 이동해서, 인증서 만료일을 조회해 보았다.
```bash
# cd /etc/kubernetes/pki

# openssl x509 -in apiserver.crt -noout -dates
notBefore=Jun 14 06:54:21 2018 GMT
notAfter=Jan 23 08:05:46 2020 GMT

# openssl x509 -in apiserver-kubelet-client.crt -noout -dates
notBefore=Jun 14 06:54:21 2018 GMT
notAfter=Jun 14 06:55:09 2019 GMT

# openssl x509 -in apiserver-etcd-client.crt -noout -dates
notBefore=Jun 14 06:47:00 2018 GMT
notAfter=Jan 23 08:05:17 2020 GMT

```
한 놈, 두 놈... 그래 범인은 `apiserver-kubelet-client.crt` 였구나.
흠 근데 인증서 만료된지 시간이 꽤 지났음에도, 클러스터에서 돌아가는 서비스에는 문제가 없었는것을 보면 신기할 노릇이었다.
`apiserver.crt` 인증서의 만료일이 특이해서 곰곰히 생각해보니, 저 때쯤 쿠버네티스 클러스터를 `1.10.x`에서 `1.11.x`로 업그레이드 했었던 사실이 떠올랐다.
아마 업그레이드 할때 `apiserver.crt`가 자동으로 갱신 되었던거 같다. 다른 클러스터의 `apiserver.crt` 인증서를 살펴보니, 기한이 1년이었다.
`apiserver.crt`를 자동으로 갱신해 줄거면, `apiserver-etcd-client.crt`도 자동으로 갱신해 줄 것이지... 뭔가 문제가 있어 의도적으로 저렇게 한것인지, 단순히 아무 의도도 없이 무시한것이지는 알 수 없지만 말이다.


## 인증서 생성
`apiserver-kubelet-client` 인증서만 다시 생성하면 되지만, 하는김에 `apiserver` 인증서와 `apiserver-etcd-client.crt`도 같이 생성하기로 했다.
<br/>
우선 인증서 파일을 삭제하였다. 아 혹시 모르니 삭제하기전에 백업을 해두자.
```bash
# cd /etc/kubernetes/pki
# mv apiserver.key apiserver.key.old
# mv apiserver.crt apiserver.crt.old
# mv apiserver-kubelet-client.key  apiserver-kubelet-client.key.old
# mv apiserver-kubelet-client.crt  apiserver-kubelet-client.crt.old
# mv apiserver-etcd-client.key apiserver-etcd-client.key.old
# mv apiserver-etcd-client.crt apiserver-etcd-client.crt.old

```
<br/>

그런 다음 `kubeadm`을 이용해서 인증서를 다시 생성하였다
```bash
# kubeadm alpha phase certs apiserver --apiserver-cert-extra-sans '10.x.u.z,kube-master.xxx.com'
[certificates] Generated apiserver certificate and key.
[certificates] apiserver serving cert is signed for DNS names [kube-master001-xxx kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local kube-master.xxx.com] and IPs [10.96.0.1 10.x.x.x 10.x.y.z]

# kubeadm alpha phase certs apiserver-kubelet-client
[certificates] Generated apiserver-kubelet-client certificate and key.

# kubeadm alpha phase certs apiserver-etcd-client
[certificates] Generated apiserver-etcd-client certificate and key.
```
<br/>

그리고 `/etc/kubernetes` 디렉토리에 있는, kubelet.conf` 파일과 `admin.conf`을 삭제한 후, 다시 생성하자.
`apiserver` 인증서를 변경하였기 때문에 `controller-manager`와 `scheduler` 의 conf 파일도 다시 만들어야. 한다.
```bash
# cd /etc/kubernetes
# mv admin.conf admin.conf.old
# mv kubelet.conf kubelet.conf.old
# mv controller-manager.conf controller-manager.conf.old
# mv scheduler.conf scheduler.conf.old

# kubeadm alpha phase kubeconfig all
OR
# kubeadm alpha phase kubeconfig admin
# kubeadm alpha phase kubeconfig kubelet
# kubeadm alpha phase kubeconfig controller-manager
# kubeadm alpha phase kubeconfig scheduler

```
<br/>

이제, 마스터 서버에 있는 `apiserver`와 `kubelet`을 재시작한다.
`kubelet`이 재시작할 때, `apiserver`를 실행시켜주기 때문에, `docker` 명령어로 죽여버렸다.
```bash
# docker stop fdd294139c9c
# systemctl restart kubelet

```
<br/>

그리고 `controller-manager`와 `scheduler`도 재시작해야 한다.
<br/>

이제 `kubectl`을 실행해 보면, 정상적으로 작동하는 것을 확인해 볼 수 있다.


## 마무리
- 아마 인증서가 동시에 만료되었으면 다른 에러가 발생했을거 같다.
- 인증서마 만료되기 전에 미리미리 갱신하자.
