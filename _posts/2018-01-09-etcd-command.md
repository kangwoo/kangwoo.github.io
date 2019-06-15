---
title:  "etcd 명령어"
classes: wide
date: 2018-01-09T20:25:00+09:00
categories: [devops, etcd]
tags: [etcd]
---

# etcd 명령어
## 인증서 기반
TLS 인증서 기반으로 etcd를 설치한 경우, etcdctl을 사용하려면 인증서 정보를 플래그로 넘겨줘야합니다. 
그리고 3 버전의 API를 사용하려면 `ETCDCTL_API=3`를 선언해줘야 합니다.
```bash
ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt \
    --key=/etc/kubernetes/pki/etcd/healthcheck-client.key \
    get foo
```

명령어가 상당히 길기 때문에, `alias`로 지정해 놓고 사용하면 편합니다.
```
alias etcd3ctl="ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/healthcheck-client.crt --key=/etc/kubernetes/pki/etcd/healthcheck-client.key"

```

## 명령어 예제
### 전체 조회
3 버전 부터는 `ls` 명령어가 존재하지 않습니다. 전체 목록 같은 것을 조회하라면 `--prefix` 플래그를 사용하면 됩니다.
아래와 같이 실행하면, 전체 목록을 조회해 볼 수 있습니다.\
```bash
$ etcd3ctl get / --prefix --keys-only

...
/registry/deployments/kube-system/coredns
/registry/deployments/kube-system/metrics-server
/registry/deployments/kube-system/tiller-deploy
/registry/deployments/openebs/maya-apiserver
/registry/deployments/openebs/openebs-admission-server
/registry/deployments/openebs/openebs-localpv-provisioner
/registry/deployments/openebs/openebs-provisioner
/registry/deployments/openebs/openebs-snapshot-operator
/registry/deployments/weave/weave-scope-app
...
```

### 멤버 조회
```bash
$ etcd3ctl member list
67fab7a197a31464, started, etcd-001, https://10.x.x.x:2380, https://10.x.x.x:2379
9ebdbf1241485ebd, started, etcd-002, https://10.y.y.y:2380, https://10.y.y.y:2379
f3aacf2611e12d71, started, etcd-003, https://10.z.z.z:2380, https://10.z.z.z:2379

```

### 엔드포인트  상태
```bash
$ etcd3ctl endpoint health
https://[127.0.0.1]:2379 is healthy: successfully committed proposal: took = 1.13549ms

$ etcd3ctl endpoint status
https://[127.0.0.1]:2379, f3aacf2611e12d71, 3.3.10, 4.6 MB, false, 6, 6880100

```
