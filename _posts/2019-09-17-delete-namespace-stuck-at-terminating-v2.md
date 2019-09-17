---
title:  "쿠버네티스 네임스페이스가 삭제되지 않을 때 강제 삭제하기"
classes: wide
date: 2019-09-09T20:17:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, helm, kubeapps]
---

# 문제
가끔식 문제가 발생하여, 네임스페이스(namespace)를 삭제할때, 상태만 `Terminating`으로 변하고, 계속 기다려도 삭제가 되지 않는 경우가 있다.

이럴 경우에는 네임스페이스의 `finalizers`를 제거해 주면 된다. (하지만 정상작으로 삭제될때까지 기다리는게 가장 좋다)

# 해결 방법
`foo`라는 네임스페이스가 있다고 가정한다.

다음과 같은 명령어로 네임스페이스 정의 내역을 json 파일로 저장한다.
```bash
$ kubectl get namespace foo -o json > foo.json
```

`foo.json` 파일을 영어서 `finalizers` 부분에 있는 `kubernetes` 값을 삭제하고, 저장한다.

그런 다음 쿠베 프락시를 실행한다. 쿠버네티스 api를 호출할 예정인데, 인증 토큰이 필요하다. 
`kubectl proxy`를 이용하면, 저장되어 있는 인증토큰을 자동으로 이용한다.

```bash
$ kubectl proxy
Starting to serve on 127.0.0.1:8001
```

다른 터미널을 열어서 쿠버네티스 api를 호출한다. 다음과 같이 api를 호출하면 변경된 `finalizers` 부분이 쿠버네티스에 반영된다.
```bash
curl -k -H "Content-Type: application/json" -X PUT --data-binary @foo.json http://127.0.0.1:8001/api/v1/namespaces/foo/finalize

```
