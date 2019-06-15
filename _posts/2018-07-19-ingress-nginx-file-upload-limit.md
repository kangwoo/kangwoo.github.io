---
title:  "Ingress nginx 파일 업로드 크기 제한 늘리기 "
classes: wide
date: 2018-06-15T20:13:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, ingress-nginx]
---

# Ingress NginX : Custom max body size
ingress-nginx를 사용하는 중, 파일 업로드 중 413 에러가 발생하였습니다.
이 경우는 nginx가 허용하는 것보다, 큰 파일이 업로드 되어 에러가 발생한 것이었습니다.

허용하는 최대 파일 크기를 늘리는 방법은 두 가지가 있습니다.
첫번째는 configmap을 에 설정하는 것이고, 두번째는 ingress 에 커스텀 어노테이션을 추가하는 것입니다.
configmap에 설정할 경우는 글로벌하게 적용되고, 커스텀 어노테이션을 사용할 경우는 해당 ingress만 영향을 받습니다.


## ConfigMap
[NGINX Configmap](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#proxy-body-size)에 ```proxy-body-size``` 값을 설정합니다.

- 예제
```yaml
apiVersion: v1
data:
  proxy-body-size: "10m"
kind: ConfigMap
metadata:
  labels:
    app: ingress-nginx
  name: nginx-configuration
```

## Ingress annotation
아래와 같은 어노테이션을 추가하면 됩니다.
```
nginx.ingress.kubernetes.io/proxy-body-size: 10m
```
- 예제
```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/proxy-body-size: 1024m
    nginx.ingress.kubernetes.io/rewrite-target: /
  labels:
    app: kubeflow
  name: kubeflow
  namespace: kubeflow
spec:
  rules:
  - host: kubeflow.xxx.yyy
    http:
      paths:
      - backend:
          serviceName: ambassador
          servicePort: 80
        path: /
```


# 참고 문서
- <https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/annotations/#custom-max-body-size>
