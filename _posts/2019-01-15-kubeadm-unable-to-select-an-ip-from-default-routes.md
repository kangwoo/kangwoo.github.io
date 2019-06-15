---
title:  "kubeadm : unable to select an IP from default routes"
classes: wide
date: 2019-01-15T06:23:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, kubeadm]
---

# 상황
쿠버네티스 1.13 버전을 kubeadm을 이용해서 설치하려고 했으나, IP를 찾을 수 없어서 에러가 발생하였습니다.
```
$ kubeadm init --config=kubeadm-config.yaml
unable to select an IP from default routes.
 
$ kubeadm init --config=kubeadm-config.yaml --v 10
I0114 20:04:41.384877 49609 interface.go:384] Looking for default routes with IPv4 addresses
I0114 20:04:41.384895 49609 interface.go:389] Default route transits interface "eth0.100"
I0114 20:04:41.386764 49609 interface.go:196] Interface eth0.100 is up
I0114 20:04:41.386833 49609 interface.go:244] Interface "eth0.100" has 1 addresses :[fe80::e642:4bff:fe1f:28b0/64].
I0114 20:04:41.386857 49609 interface.go:211] Checking addr fe80::e642:4bff:fe1f:28b0/64.
I0114 20:04:41.386873 49609 interface.go:224] fe80::e642:4bff:fe1f:28b0 is not an IPv4 address
I0114 20:04:41.386892 49609 interface.go:384] Looking for default routes with IPv6 addresses
I0114 20:04:41.386903 49609 interface.go:389] Default route transits interface "eth1.100"
I0114 20:04:41.389546 49609 interface.go:196] Interface eth0.100 is up
I0114 20:04:41.389592 49609 interface.go:244] Interface "eth0.100" has 1 addresses :[fe80::e642:4bff:fe1f:28b0/64].
I0114 20:04:41.389608 49609 interface.go:211] Checking addr fe80::e642:4bff:fe1f:28b0/64.
I0114 20:04:41.389622 49609 interface.go:221] Non-global unicast address found fe80::e643:4bff:fe1f:38b0
I0114 20:04:41.389634 49609 interface.go:400] No active IP found by looking at default routes
unable to fetch the kubeadm-config ConfigMap: unable to select an IP from default routes.
```

네트워크 인터페이스(network interface)에서 IP를 찾을 수 없기 때문에 발생하는 에러였습니다. 해당 서버는 loopback 인터페이스에 IP가 바인딩 되어 있었는데, kubeadm이 사용하는 코드에서는 loopback 인터페이스를 확인하지 않았습니다.
아마 IPv4 BGP over IPv6 ([rfc5549](https://tools.ietf.org/html/rfc5549))을 사용한거 같은데, 자세한 사항은 모르겠습니다.

이 문제에 대한 [이슈](https://github.com/kubernetes/kubeadm/issues/1156)는 등록되어 있고, 
해결 방법으로 [PR](https://github.com/kubernetes/kubernetes/pull/69578)이 올라가 있지만 반영되지 않았습니다.

이 문제를 해결하기 위한 방법은, 네트워커 설정을 변경하거나, kubeam 코드를 패치하면 됩니다.
이 문서에는 간단히 kubeam 코드 패치를 선탁하였습니다.

## kubeadm 빌드하기
소스 코드 빌드 과정은 Docker 컨테이너 안에서 일어나기 때문에, 별도의 golang 환경을 구축할 필요는 없습니다.

kubeadm 빌드하는 과정은 간단합니다.

우선 쿠버네티스 소스를 받습니다.
```bash
git clone https://github.com/kubernetes/kubernetes.git
cd kubernetes
```
태그로 버전을 확인한 후, 본인이 원하는 버전으로 체크아웃 합니다.
```bash
git tag
git checkout v1.13.6
```

```staging/src/k8s.io/apimachinery/pkg/util/net/interface.go``` 파일을 열어서, 아래와 같은 코드를 추가해 줍니다. 
(https://github.com/kubernetes/kubernetes/pull/69578/files 을 참고하시기 바랍니다.)
```golang
			klog.V(4).Infof("Default route exists for IPv%d, however interface %q does not have global unicast addresses. Checking loopback interface", uint(family), route.Interface)
			loopbackIP, err := getIPFromInterface("lo", family, nw)
			if err != nil {
				return nil, err
			}
			if loopbackIP != nil {
				klog.V(4).Infof("Found active IP %v on loopback interface", loopbackIP)
				return loopbackIP, nil
			}

```

빌드 스크립트를 실행서, kubeadm을 빌드합니다.
```bash
build/run.sh make kubeadm KUBE_BUILD_PLATFORMS=linux/amd64
```

빌드가 정상적으로 끝나면, ```_output/dockerized/bin/linux/amd64/kubeadm``` 에 파일을 생성합니다.

생성한 파일을 서버로 이동해서, 설치하시면 됩니다.


# 참고 문서
- https://github.com/kubernetes/kubeadm/issues/1156
- https://github.com/kubernetes/kubernetes/pull/69578
- https://kubernetes.io/docs/setup/release/building-from-source/
- https://github.com/kubernetes/kubernetes/tree/master/build/


