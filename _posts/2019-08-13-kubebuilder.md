---
title:  "kubebuilder"
classes: wide
date: 2019-08-08T08:50:12+09:00
categories: [devops, kubernetes, kubebuilder]
tags: [kubernetes, kubebuilder]
---

# Install

## Install kubebuilder
```bash
os=$(go env GOOS)
arch=$(go env GOARCH)

# download kubebuilder and extract it to tmp
curl -sL https://go.kubebuilder.io/dl/2.0.0-beta.0/${os}/${arch} | tar -xz -C /tmp/

# move to a long-term location and put it on your path
# (you'll need to set the KUBEBUILDER_ASSETS env var if you put it somewhere else)
sudo mv /tmp/kubebuilder_2.0.0-beta.0_${os}_${arch} /usr/local/kubebuilder
export PATH=$PATH:/usr/local/kubebuilder/bin
```


## Create a Project

```bash
mkdir namespace-operator
cd namespace-operator

go mod init kangwoo.github.io/namespace-operator

kubebuilder init --domain kangwoo.github.io
```


## Adding a new API

```bash
kubebuilder create api --group tenant --version v1 --kind NamespaceRequest --namespaced false
```

## Adding a new Webhook

```bash
kubebuilder create webhook --group tenant --version v1 --kind NamespaceRequest --defaulting --programmatic-validation
```
