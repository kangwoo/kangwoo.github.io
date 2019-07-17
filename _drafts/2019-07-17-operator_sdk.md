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
$ operator-sdk new jupyter-operator2 --repo jupyter-operator
...
go: finding github.com/operator-framework/operator-sdk master
go: bitbucket.org/ww/goautoneg@v0.0.0-20120707110453-75cd24fc2f2c: hg clone -U https://bitbucket.org/ww/goautoneg . in /Users/lineplus/go/pkg/mod/cache/vcs/59c2185b80ea440a7c3b8c5eff3d8abb68c53dea1f20f615370c924c4150b27f: exec: "hg": executable file not found in $PATH
go: error loading module requirements
Error: failed to exec []string{"go", "build", "./..."}: exit status 1
...


$ brew install hg
```

## 참고 자료
- https://github.com/operator-framework/operator-sdk/blob/master/doc/user/install-operator-sdk.md

