---
title:  "Kubernetes 커맨드 라인 툴"
classes: wide
date: 2019-12-02T20:21:00+09:00
categories: [devops, kubernetes]
tags: [kubernetes, tools]
---

## 소개글
쿠버네티스 클러스터를 자주 사용하는 사람이라면, 반복적으로 명령을 입력하는데 불편함을 느낄것입니다.
이러한 불편함을 줄이기 위한 방법은 `kubectl`에 CLI 도구들을 사용하는것입니다.
이 문서에서는 `kubectl`의 사용을 도와줄 몇가지 도구들을 소개하겠습니다.

## kubectl-aliases : 별칭 주기
매번 `kubectl`이나 다른 명령어를 입력할 수 있지만, 상당히 불편함을 느낄것입니다.
이런 불편함을 줄이는 가장 쉬운 방법은 별칭(alias)를 지정해서 사용하는 것입니다.
```
alias k='kubectl'
alias kg='kubectl get'
alias kgpo='kubectl get pod'
...
```
위의 예제처럼 별칭을 지정해 놓으면, 보다 효율적으로 명령어를 입력할 수 있습니다.
자주 쓰는 명령어는 [kubectl-aliases](https://github.com/ahmetb/kubectl-aliases)에 정의되어 있습니다. 참고하시기 바랍니다.

## Shell Autocompletion : 쉡 자동완성 하기
`kubectl`의 명령어를 일일이 입력하고 있다면, 쉘 자동 완성을 설정하는게 좋습니다.
쉘에 `kubectl` 입력하고 [Tab]키를 눌러서 자동완성을 시도하면, 추천 단어 목록을 보여주거나, 추천 단어가 1개일 경우 자동으로 완성이 됩니다.
`kubectl`이 대한 쉘 자동 완성을 설정하는 방법은 [설치 문서](https://kubernetes.io/docs/tasks/tools/install-kubectl/#enabling-shell-autocompletion)를 참고하시기 바랍니다.

### 자동 완성 사용하기 
예를 들어서 `g`를 입력하고 `tab`을 누르면, `get`으로 자동 완성됩니다.
```
$ kubectl g [Tab]
$ kubectl get
```

`get p`를 입력하고 `tab`을 누르면, `p`로 시작하는 단어 목록을 보여줍니다.
```
$ kubectl get p [Tab]
persistentvolumeclaims             podsecuritypolicies.policy
persistentvolumes                  podtemplates
poddisruptionbudgets.policy        policies.authentication.istio.io
pods                               priorityclasses.scheduling.k8s.io
podsecuritypolicies.extensions    
```


## kubectx & kubecns : 컨텍스트와 네임스페이스 쉽게 변경하기
쿠버네티스의 컨텍스트를 여러개 사용하고 있거나, 네임스페이스를 여러개 사용하고 있다면, 이 도구들이 도움이 될것입니다.
`kubectx`는 컨텍스트를 쉽게 변경할 수 있도록 도움을 줍니다.
이 도구를 사용하면, `kubectl config use-context greentea` 같은 긴 명령어를 사용하지 않아도 됩니다. 
그리고 `kubens`는 기본 네임스페이스를 변경할 수 있도록 도와줍니다.
이 두 도구 모드 [Tab] 완성을 지원합니다. 그 뿐만 아니라, `fzf`를 설치하면 대화식 메뉴를 제공하기 때문에, 더욱 편리하게 사용할 수 있습니다.

`kubectx` 와 `kubens`에 대한 설정 방법은 [설치 문서](https://github.com/ahmetb/kubectx)를 참고하시기 바랍니다.

### kubectx 사용하기
`kubectx` 명령을 실행하면, 컨텍스트 목록을 보여줍니다.
```
$ kubectx
coffee
greentea
```

컨텍스트를 변경하기 위해서는, 컨텍스트 명을 입력하면 됩니다.
```
$ kubectx greentea
Switched to context "greentea".
```

만약 `fzf`가 설치되어 있으면, `kubectx` 명령을 실행하면 대화식 메뉴를 보여줍니다.
```
$ kubectx
> coffee
  greentea
  2/2
```

### kubens 사용하기
`kubens` 명령을 실행하면, 네임스페이스 목록을 보여줍니다.
```
$ kubens
kube-system
kube-public
istio-system
default
```

네임스페이스를 변경하기 위해서는, 네임스페이스 명을 입력하면 됩니다.
```
$ kubens kube-system
Context "greentea" modified.
Active namespace is "kube-system".
```

만약 `fzf`가 설치되어 있으면, `kubens` 명령을 실행하면 대화식 메뉴를 보여줍니다.
```
$ kubectx
> kube-system
  kube-public
  istio-system
  default
  4/4
```

## kube-ps1 : 컨템스트와 네임스페이스 이름을 쉘 프로프트에 보여주기 
쿠버네티스의 컨텍스트나 네임스페이스를 여러개 사용하고 있을때, 현재 어떤 컨텍스트와 네임스페이스를 사용하고 있는지 헷갈리는 경우가 많습니다.
`kube-ps1`은 현재 사용하고 있는 쿠버네티스 컨텍스트 및 네임스페이스를 쉘의 프롬프 문자열에 보여줍니다.
이 도구를 사용하면, `kubectl config current-context` 같은 긴 명령어를 사용하지 않아도 됩니다.
그리고, 컨텍스트와 네임스페이스를 보는게 불편하다면, `kubeoff` 명령어를 실행해서 `kube-ps`을 비활성화 시킬수도 있습니다.
물론, 다시 컨텍스트와 네임스페이스를 보고 싶다면 `kubeon` 명령어를 실행하면 됩니다.
`kube-ps1``에 대한 설정 방법은 [설치 문서](https://github.com/jonmosco/kube-ps1)를 참고하시기 바랍니다.

`kube-ps1`의 설치가 완료되면, 셀의 프롬프트가 다음처럼 표시됩니다.
```
(⎈ |[context]:[namespace]) $
(⎈ |greentea:kube-system) $
```

## stern : 여러개의 팟(pod)이나 컨테이너의 로그를 쉽게 보기
`kubectl`은 기본적으로 팟이나 컨테이너의 로그를 테일링해서 볼 수 있는 기능을 제공하고 있습니다.
하지만, 팟이 여러개이거나 하나의 팟에 여러개의 컨테이너가 있을 경우는 로그른 테일링 해서 볼 수가 없습니다.

`stern`는 쿠버네티스트의 여러 팟이나, 팟 내의 여러 컨테이너의 로그를 테일링 할 수 있도록 해줍니다. 
그리고 대상별로 출력되는 로그가 색상별로 표현되기 때문에 쉽게 구별할 수 있습니다.
`stern``에 대한 설정 방법은 [설치 문서](https://github.com/wercker/stern)를 참고하시기 바랍니다.


`stern ginger`이라는 명령어를 실행하면, `ginger`이라는 이름을 가진 팟에 속한 컨테니이너들의 로그를 모두 보여줍니다.
이 쿼리는 정규식이기 때문에 팟 이름을 쉽게 필터링할 수 있으며, 정확한 이름을 지정하지 않아도 됩니다.
다시 말해성 `ginger`로 시작하는 이름을 가진 모든 팟들의 로그를 보여주게 되는것입니다.
```
$ stern ginger
hello gingersnap istio-proxy 2019-12-02T09:05:41.739784Z	info	watchFileEvents: "/etc/certs": MODIFY|ATTRIB
hello gingersnap istio-proxy 2019-12-02T09:05:41.739842Z	info	watchFileEvents: "/etc/certs/..2019_11_06_05_09_24.409981738": MODIFY|ATTRIB
...
```

그리고 아래처럼, 레이블을 가지고 쿼리할 수 있습니다.
```
stern --all-namespaces -l run=cookie
```



## k9s : 터미널 UI
`k9s`는 쿠버네티스 클러스터와 상호 작용할 수 있는 터미널 UI를 제공합니다.
이 도구는 쿠버네티스 리소스들을 쉽게 탐색하고 관리할 수 있도록 도와줍니다.
'k9s'에 대한 설정 방업은 [설치 문서](https://github.com/derailed/k9s)를 참고하시기 바랍니다.

Pod 조회하기
![pods](/assets/img/2019/12/k9s-po.png)

로그 보기
![logs](/assets/img/2019/12/k9s-logs.png)