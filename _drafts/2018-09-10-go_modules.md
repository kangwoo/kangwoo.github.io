---
title:  "Go Modules"
classes: wide
date: 2019-07-02T07:21:10+09:00
categories: [devops, go]
tags: [go]
---

Go 언어의 초창기에는 의존성 관리를 위한 마땅한 도구가 없었습니다. 
하지만 프로젝트를 진행함에 있어 의존성 관리는 꼭 필요한 기능이라서, 시간이 지나면서 다양한 써드파트 도구가 등장하기 시작했습니다.
대표적으로 `vgo`, `dep`, `glide` 등이 있습니다.
이렇게 표준이 없어서, 여러 써드파트 도구가 사용되다가, 
Russ Cox의 제안으로 vgo 프로젝트가 Go의 공식툴로 채택되었고, go 1.11 버전에서 Go Modules가 도입되어 사용할 수 있게 되었습니다.
(GOPATH 안녕~~~)

## GO111MODULE
`go 1.11` 버전에 Go Modules이 등장하면서, 기존 버전에서 사용하는 환경과 공종을 하기 위해서 `GO111MODULE` 이라는 환경변수가 생겼습니다.
이 환경변수에는 세 가지의 값이 올 수 있습니다.


`GO111MODULE`의 값이 `on`인 경우에는 `go` 명령어는 `GOPATH`에 상관없이 Go Modules의 방식으로 작동합니다.
`off`인 경우에는 반대로, Go Modules을 비활성화하고 기존에 사용하던 방식인 `GOPATH`를 이용합니다.
그리고 값이 없거나 `auto`인 경우에는 `GOPATH` 내부에서는 기존 방식대로, 외부에서는 Go Modules 방식대로 작동합니다.


## 명령어 
- go mod init <module-name> : 모듈을 생성합니다.
- go get <module-path>@<module-query> : 특정 버전을 지정해서 모듈을 추가합니다.
- go mod tidy [-v] : go.mod 파일가 소스 파일을 비교하여, import 하지 않은 모듈은 제거하고 import 하였지만 의존성 목록에 추가되지 않은 모듈은 추가합니다.
- go mod vendor : vendor 디렉토리를 생성하고, 모듈들을 복사합니다.
- go mod verify : 모듈의 유효성을 검증합니다.
```
