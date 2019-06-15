---
title:  "flannel : failed to find IPv4 address"
classes: wide
date: 2019-05-31T20:51:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, flannel]
---

`flannel`을 설치하였으나, 아이피를 찾을 수 없다는 에러가 발생하고 작동하지 않는 문제가 발생하였습니다.

```bash
$ kubectl -n kube-system get pod -lapp=flannel
NAME                    READY     STATUS             RESTARTS   AGE
kube-flannel-ds-2vnhj   1/1       CrashLoopBackOff   5          1m12s
kube-flannel-ds-b6mqq   1/1       CrashLoopBackOff   5          1m12s
kube-flannel-ds-f2bpz   1/1       CrashLoopBackOff   5          1m12s
```

```bash
$ kubectl -n kube-system logs kube-flannel-ds-2vnhj
I0601 19:31:31.628591       1 main.go:475] Determining IP address of default interface
E0601 19:31:31.630621       1 main.go:193] Failed to find any valid interface to use: failed to find IPv4 address for interface eth0.100
```

`flannel`은 기본적으로 `eth0` 인터페이스에서 아이피를 찾게 되는데, 해당 서버는 `bond0`에 IP가 할당 되어 있었습니다.
이 문제를 해결하기 위해서는 실행 플래그에 `--iface=bond0`을 추가해 주면 됩니다.

```bash
$ kubectl -n kube-system edit daemonset kube-flannel-ds
 
...
    spec:
      containers:
      - args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=bond0
        - --iface=eth0
        command:
        - /opt/bin/flanneld
        env:
        - name: POD_NAME
...
```

