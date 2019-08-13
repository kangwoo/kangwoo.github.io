---
title:  "Kubernets ServiceAccount로 kuebconfig 파일 생성하기"
classes: wide
date: 2019-08-08T08:50:12+09:00
categories: [data science, machine learning, tensorflow, keras]
tags: [machine learning, tensorflow, keras]
---

## 서비스계정(ServiceAccount) 생성
```bash
kubectl create serviceaccount super-man
```

## ClusterRole 또는 Role Binding
```bash
kubectlcreate clusterrolebinding cluster-admin:super-man --clusterrole=cluster-admin --serviceaccount=default:super-man
```


## Kubernets ServiceAccount로 kuebconfig 파일 생성하기
```bash
# your server name goes here
server=https://localhost:6443
# the name of the service account
name=SERVICE_ACCOUNT_NAME
# the name of the namespace
namespace=default

token_name=$(kubectl -n $namespace get serviceaccount $name -o jsonpath='{.secrets[].name}')
ca=$(kubectl -n $namespace get secret/$token_name -o jsonpath='{.data.ca\.crt}')
token=$(kubectl -n $namespace get secret/$token_name -o jsonpath='{.data.token}' | base64 --decode)
namespace=$(kubectl -n $namespace get secret/$token_name -o jsonpath='{.data.namespace}' | base64 --decode)

echo "
apiVersion: v1
kind: Config
clusters:
- name: default-cluster
  cluster:
    certificate-authority-data: ${ca}
    server: ${server}
contexts:
- name: default-context
  context:
    cluster: default-cluster
    namespace: ${namespace}
    user: ${name}
current-context: default-context
users:
- name: ${name}
  user:
    token: ${token}
" > kubeconfig

```
