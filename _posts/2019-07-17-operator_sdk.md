---
title:  "Kubernetes operator-sdk를 이용한 Go Operator 만들기"
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
go 모듈을 사용하기 위해서 `GO111MODULE=on`을 설정하거나, `GOPATH`가 아닌 경로에 디렉토리를 생성해야한다.

```bash
$ mkdir -p ~/workspace
$ cd ~/workspace
$ export GO111MODULE=on
$ operator-sdk new jupyter-operator --repo github.com/kangwoo/jupyter-operator
INFO[0000] Creating new Go operator 'jupyter-operator'.
INFO[0000] Created go.mod
INFO[0000] Created tools.go
INFO[0000] Created cmd/manager/main.go
INFO[0000] Created build/Dockerfile
INFO[0000] Created build/bin/entrypoint
INFO[0000] Created build/bin/user_setup
INFO[0000] Created deploy/service_account.yaml
INFO[0000] Created deploy/role.yaml
INFO[0000] Created deploy/role_binding.yaml
INFO[0000] Created deploy/operator.yaml
INFO[0000] Created pkg/apis/apis.go
INFO[0000] Created pkg/controller/controller.go
INFO[0000] Created version/version.go
INFO[0000] Created .gitignore
INFO[0000] Validating project
go: finding github.com/operator-framework/operator-sdk master
INFO[0108] Project validation successful.
INFO[0108] Project creation complete.
```

### Mercurial 설치
만일 `operator`를 생성하는 중 다음과 같이 `hg` 실행 파일을 찾을 수 없다는 에러가 발생한다면, `hg`를 별도로 설치해야한다.
```bash
$ operator-sdk new jupyter-operator --repo github.com/kangwoo/jupyter-operator
...
go: finding github.com/operator-framework/operator-sdk master
go: bitbucket.org/ww/goautoneg@v0.0.0-20120707110453-75cd24fc2f2c: hg clone -U https://bitbucket.org/ww/goautoneg . in /Users/lineplus/go/pkg/mod/cache/vcs/59c2185b80ea440a7c3b8c5eff3d8abb68c53dea1f20f615370c924c4150b27f: exec: "hg": executable file not found in $PATH
go: error loading module requirements
Error: failed to exec []string{"go", "build", "./..."}: exit status 1
...
```

`hg`도 'brew'를 사용해서 설치할 수 있다. 
참고로 'hg'는 [Mercurial]<https://www.mercurial-scm.org/> 이라는 크로스 플랫폼 분산 버전 관리 도구의 명령툴이다.

```bash
$ brew install hg
```

### 생성한 operator 디렉토리 이동
생성한 operator 디렉토리로 이동한다.
```bash
$ cd jupyter-operator
```

### CR(Custom Resource) 생성
`operator-sdk add api` 명령어를 이용해서, API를 생성한다.

```bash
$ operator-sdk add api --api-version=kangwoo.github.io/v1alpha1 --kind=Jupyter
INFO[0000] Generating api version kangwoo.github.io/v1alpha1 for kind Jupyter.
INFO[0000] Created pkg/apis/kangwoo/group.go
INFO[0003] Created pkg/apis/kangwoo/v1alpha1/jupyter_types.go
INFO[0003] Created pkg/apis/addtoscheme_kangwoo_v1alpha1.go
INFO[0003] Created pkg/apis/kangwoo/v1alpha1/register.go
INFO[0003] Created pkg/apis/kangwoo/v1alpha1/doc.go
INFO[0003] Created deploy/crds/kangwoo_v1alpha1_jupyter_cr.yaml
INFO[0011] Created deploy/crds/kangwoo_v1alpha1_jupyter_crd.yaml
INFO[0011] Running deepcopy code-generation for Custom Resource group versions: [kangwoo:[v1alpha1], ]
INFO[0019] Code-generation complete.
INFO[0019] Running OpenAPI code-generation for Custom Resource group versions: [kangwoo:[v1alpha1], ]
INFO[0036] Created deploy/crds/kangwoo_v1alpha1_jupyter_crd.yaml
INFO[0036] Code-generation complete.
INFO[0036] API generation complete.
```


### Controller 생성
`operator-sdk add controller` 명령어를 이용해서, Controller를 생성한다.
```bash
$ operator-sdk add controller --api-version=kangwoo.github.io/v1alpha1 --kind=Jupyter
INFO[0000] Generating controller version kangwoo.github.io/v1alpha1 for kind Jupyter.
INFO[0000] Created pkg/controller/jupyter/jupyter_controller.go
INFO[0000] Created pkg/controller/add_jupyter.go
INFO[0000] Controller generation complete.
```

### 빌드(Build) 하기
```bash
$ operator-sdk build kangwoo/jupyter-operator:latest   
```
     
### 코드 생성 하기
리소스의 Spec이 변경되었을 경우, 코드를 다시 생성해줘야한다.
```bash
$ operator-sdk generate k8s
$ operator-sdk generate openapi
```


## 참고 자료
- https://github.com/operator-framework/operator-sdk/blob/master/doc/user/install-operator-sdk.md
- https://github.com/operator-framework/operator-sdk/blob/master/doc/user-guide.md
- https://github.com/operator-framework/operator-sdk/blob/master/doc/operator-scope.md
