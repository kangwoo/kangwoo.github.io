---
title:  "Ingress status에 값이 없을 경우"
classes: wide
date: 2019-11-11T20:31:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, ingress, argocd]
---

## Ingress status에 값이 없다.
`argocd`를 이용해서, 리소스들을 동기화 했는데, `ingress` 부분 계속 `Processing`이라고 나오이고 끝날 생각을 안한다.
`ingress` 의 상태를 보니 다음과 같았다.
```
$ kubectl get ingresses dobby -o yaml
...
status:
  loadBalancer: {}
```

뭔가 저기 `status`에 값이 할당되어야 할거 같은 느낌이 들었지만, 왜 안되어 있는지 이유는 알지 못했다.
그래서 사용중인 `nginx-ingress-controller`의 설정을 살펴보았다.

```
...
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.0
          args:
            - /nginx-ingress-controller
            - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-service=$(POD_NAMESPACE)/ingress-nginx
            - --annotations-prefix=nginx.ingress.kubernetes.io
...
```  

`--publish-service` 플래그를 사용해서, `ingress-nginx` 서비스의 IP를 읽어와서 업데이트 해주는거 같은데,
불행히도 해당 `ingress-nginx` 서비스가 `LoadBalancer`이 아니라서 IP가 존재하지 않는다.
그래서 `--publish-service` 플래그를 삭제하고, `--publish-service` 플래그를 사용해서 직접 명시해주었다.

```
...
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.25.0
          args:
            - /nginx-ingress-controller
            - --default-backend-service=$(POD_NAMESPACE)/default-http-backend
            - --configmap=$(POD_NAMESPACE)/nginx-configuration
            - --tcp-services-configmap=$(POD_NAMESPACE)/tcp-services
            - --udp-services-configmap=$(POD_NAMESPACE)/udp-services
            - --publish-status-address=10.xx.yy.zz
            - --annotations-prefix=nginx.ingress.kubernetes.io
...
```  

변경사항을 반영하고, `ingress` 의 상태를 조회를 해보니, 정상적으로 상태값이 반영되었다.
```
$ kubectl get ingresses dobby -o yaml
...
status:
  loadBalancer:
    ingress:
    - ip: 10.xx.yy.zz
```

`ingress` 의 상태값이 반영된 후, `argocd`도 정상적으로 작동하였다.


## 참고
- https://github.com/kubernetes/ingress-nginx/blob/master/docs/examples/static-ip/README.md