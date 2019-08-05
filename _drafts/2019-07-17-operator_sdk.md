---
title:  "Operator SDK"
classes: wide
date: 2019-07-17T07:10:10+09:00
categories: [devops, kubernetes]
tags: [kubernetes, operator sdk]
---

## Install the Operator SDK CLI

### Install from Homebrew
```bash
$ brew install operator-sdk
```

### Create an operator
작업할 디렉토리를 생성하고, `operator-sdk`를 사용해서 `operator`를 생성한다.
go 모듈을 사용하기 위해서 `GO111MODULE=on`을 설정하였다.

```bash
$ mkdir -p ~/workspace/colab/
$ cd ~/workspace/colab/
$ export GO111MODULE=on
$ operator-sdk new jupyter-operator --repo jupyter-operator
$ cd jupyter-operator
```


`operator-sdk`를 사용해서 새로운 `operator`를 생성하는 중 다음과 같이 `hg` 실행 파일을 찾을 수 없다는 에러가 발생하였다.

그래서 `hg`도 'brew'를 사용해서 설치하였다. 'hg'는 [Mercurial]<https://www.mercurial-scm.org/> 이라는 크로스 플랫폼 분산 버전 관리 도구의 명령툴이다.

```bash
$ operator-sdk new jupyter-operator --repo jupyter-operator
...
go: finding github.com/operator-framework/operator-sdk master
go: bitbucket.org/ww/goautoneg@v0.0.0-20120707110453-75cd24fc2f2c: hg clone -U https://bitbucket.org/ww/goautoneg . in /Users/lineplus/go/pkg/mod/cache/vcs/59c2185b80ea440a7c3b8c5eff3d8abb68c53dea1f20f615370c924c4150b27f: exec: "hg": executable file not found in $PATH
go: error loading module requirements
Error: failed to exec []string{"go", "build", "./..."}: exit status 1
...


$ brew install hg
```

$ operator-sdk add api --api-version=colab.linecorp.com/v1alpha1 --kind=Jupyter
INFO[0000] Generating api version colab.linecorp.com/v1alpha1 for kind Jupyter.
INFO[0000] Created pkg/apis/colab/group.go
INFO[0003] Created pkg/apis/colab/v1alpha1/jupyter_types.go
INFO[0003] Created pkg/apis/addtoscheme_colab_v1alpha1.go
INFO[0003] Created pkg/apis/colab/v1alpha1/register.go
INFO[0003] Created pkg/apis/colab/v1alpha1/doc.go
INFO[0003] Created deploy/crds/colab_v1alpha1_jupyter_cr.yaml
INFO[0011] Created deploy/crds/colab_v1alpha1_jupyter_crd.yaml
INFO[0011] Running deepcopy code-generation for Custom Resource group versions: [colab:[v1alpha1], ]
INFO[0019] Code-generation complete.
INFO[0019] Running OpenAPI code-generation for Custom Resource group versions: [colab:[v1alpha1], ]
INFO[0036] Created deploy/crds/colab_v1alpha1_jupyter_crd.yaml
INFO[0036] Code-generation complete.
INFO[0036] API generation complete.


$ operator-sdk add controller --api-version=colab.linecorp.com/v1alpha1 --kind=Jupyter
INFO[0000] Generating controller version colab.linecorp.com/v1alpha1 for kind Jupyter.
INFO[0000] Created pkg/controller/jupyter/jupyter_controller.go
INFO[0000] Created pkg/controller/add_jupyter.go
INFO[0000] Controller generation complete.



$ operator-sdk build docker-registry.linecorp.com/kangwoo/jupyter-operator:latest        

operator-sdk generate k8s
 
operator-sdk generate openapi

## 참고 자료
- https://github.com/operator-framework/operator-sdk/blob/master/doc/user/install-operator-sdk.md
- https://github.com/operator-framework/operator-sdk/blob/master/doc/user-guide.md
- https://github.com/operator-framework/operator-sdk/blob/master/doc/operator-scope.md
