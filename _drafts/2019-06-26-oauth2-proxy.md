---
title:  "Identity Provider (WIP)"
classes: wide
date: 2018-02-06T13:15:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, oauth2-proxy, ingress-nginx]
---

oauth2-proxy를 ingress-nginx에 연동해서 사용할 경우 

[2019/06/26 08:49:13] [oauthproxy.go:679] Error loading cookied session: Cookie "_oauth2_proxy" not present

에러가 발생

nginx에서도 다른 에러 발

9.131 Safari/537.36" 608 0.178 [datalab-system-oauth2-proxy-4180] 172.22.32.5:4180 0 0.178 502 c5a96bcd69b3c80b5b8f3e1627e6631e
2019/06/26 08:49:22 [error] 1769#1769: *15032 upstream sent too big header while reading response header from upstream, client: 172.22.96.0, server: datalab.linecorp-dev.com, request: "GET /oauth2/callback?code=xK2BWt&state=87dec78ea6a339a05cdf3d3d0e9260d3:/ HTTP/1.1", upstream: "http://172.22.32.5:4180/oauth2/callback?code=xK2BWt&state=87dec78ea6a339a05cdf3d3d0e9260d3:/", host: "datalab.linecorp-dev.com"

https://andrewlock.net/fixing-nginx-upstream-sent-too-big-header-error-when-running-an-ingress-controller-in-kubernetes/


kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-configuration
  namespace: kube-system
  labels:
    k8s-app: nginx-ingress-controller
data:
  proxy-buffer-size: "16k"


# Day 2
## Istio : End User Authentication
### JWT이 있을 경우만 서비스에 접근할 수 있도록 테스트 하였다.
우선 기존에 띄웠던 http-echo 서비스를 제거하고, istio proxy가 포함된 상태로 서비스를 다시 띄웠다.
기록을 위해서 주입된 yaml 파일을 생성한 후, 서비스를 실행하였다.
```bash
$ kubectl -n test delete -f http-echo.yaml

$ istioctl kube-inject -f http-echo.yaml -o http-echo-injected.yaml

$ kubectl -n test delete -f http-echo-injected.yaml

```

### End User Authentication를 위한 Policy 생성
```yaml
apiVersion: "authentication.istio.io/v1alpha1"
kind: "Policy"
metadata:
  name: example
  namespace: test
spec:
  targets:
  - name: http-echo
  origins:
  - jwt:
      issuer: "https://xxx.xxx.com"
      jwksUri: "https://xxx.xxx.com/jwks"
  principalBinding: USE_ORIGIN
```
정책(Policy)를 생성하였지만, http-echo 서비스에 접근이 되었다. 
뭐가 문제인것일까...?
혹시 몰라서 Policy 사용 활성화 여부도 체크해 보았지만 문제 없었다.
```bash
$ kubectl -n istio-system get cm istio -o jsonpath="{@.data.mesh}" | grep disablePolicyChecks
disablePolicyChecks: false
```
istio 관련 로그를 봐도 알 수가 없다...

수 많은 삽집 끝에 문제의 이유를 찾아낼 수 있었다.
`http-echo` 서비스의 포트에 이름(name)을 지정하지 않았서이다. 포트의 이름을 `http`로 설정하니 정상 작동하였다.
(포트의 이름을 지정하지 않고, Policy의 targets에 port를 명시하면 될까? 다음에 테스트 해보자..)
```yaml
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: http-echo
  name: http-echo
  namespace: test
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
      name: http
  selector:
    app: http-echo
  type: ClusterIP
```


```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: http-echo
  name: http-echo
  namespace: test
spec:
  replicas: 1
  selector:
    matchLabels:
      app: http-echo
  template:
    metadata:
      labels:
        app: http-echo
    spec:
      containers:
        - image: mendhak/http-https-echo
          imagePullPolicy: Always
          name: http-echo
          ports:
            - containerPort: 80
              protocol: TCP
          resources: {}
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: http-echo
  name: http-echo
  namespace: test
spec:
  ports:
    - port: 80
      protocol: TCP
      targetPort: 80
      name: http
  selector:
    app: http-echo
  type: ClusterIP

```

## RBAC
```yaml
apiVersion: "rbac.istio.io/v1alpha1"
kind: RbacConfig
metadata:
  name: default
  namespace: test
spec:
  mode: 'ON_WITH_INCLUSION'
  inclusion:
    services:
      - "mins.test.svc.cluster.local"

---
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRole
metadata:
  name: member
  namespace: test
spec:
  rules:
    - services: ["*"]
      paths: ["*"]
      methods: ["*"]

---
apiVersion: "rbac.istio.io/v1alpha1"
kind: ServiceRoleBinding
metadata:
  name: member-group-datalab
  namespace: test
spec:
  subjects:
    - properties:
        request.auth.claims[groups]: "datalab"
  roleRef:
    kind: ServiceRole
    name: "member"

```



## 어노테이션 기반 주입 테스트
sidecar.istio.io/inject: "true"
