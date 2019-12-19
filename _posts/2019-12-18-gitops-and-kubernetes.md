---
title:  "GitOps and Kubernetes"
classes: wide
date: 2019-12-18T20:20:37+09:00
categories: [devops, kubernetes]
tags: [kubernetes, gitops, argocd]
---

# GitOps 스타일의 지속적 배포

쿠버네티스에 애플리케이션을 배포하는 방법은 여러가지가 있습니다. 

가장 간단한 방법으로는, 사람이 직접 `kubectl` 를 실행해서 매니페스트를 클러스터에 적용하는 것입니다. 물론 사람이 실행하기 때문에 번거롭게, 실수가 쉽게 발생한다는 문제가 있습니다. 그래서 보통은 자동화 도구를 사용합니다. `Spinnaker`, `Jenkins X`, `Tekton`, `Argo Workflow`, `Argo CD` 와 같은 CI/CD 도구들을 사용해서 배포하는 것입니다.

이 문서에서는 요즘 사용되고 있는 GitOps 스타일의 방법을  사용하여,  CI/CD 파이프라인을 만드는 방법에 대해서 설명하겠습니다.

 블루/그린 배포, 카나리아 분석, 멀티 클라우드 배포 등의 고급 기능을 사용하려면, `Spinnaker`가 더 좋은 선택지가 될 수 있지만, 간단히 사용하기에 `Argo CD` 로도 충분하다고 생각합니다. 이 문서에는 `Argo CD` 를 사용합니다.

# GitOps

GitOps 라는 용어는 [Weaveworks](https://www.weave.works/blog/gitops-operations-by-pull-request)에서 만들었습니다.

GitOps의 핵심은 Git 저장소에 저장된 쿠버네티스 매니페스트 같은 파일을 이용하여, 배포를 선언적으로 한다는 것입니다. 즉, Git에 저장된 매니페스트가 쿠버네티스 클러스터에도 똑같이 반영된다는 것입니다.

이러한 방법은 이해하기 쉬운 운영 모델을 제공하며, Git을 사용하기 때문에 보안 및 감사 기능도 기본으로 제공됩니다. 그리고 재해로부터 쉽게 복구할 수 있습니다. 무엇보다도 큰 장점은 개발자 친화적이라는 것입니다. 

이런 선언적 스타일은 쿠버네티스와 잘 어울립니다.  

이미 아시고 계신분들도 있지만,  쿠버네티스의 주요한 개념 중 하나는 선언적 시스템이라는 것입니다. 어떠한 리소스를 생성하라 명령하는 것이 아니라,  사용자는 매니페스트를 정의하고, 시스템은 그 상태를 유지하기 위해 노력한다는 것입니다. 이런 점이 상당히 유사하기 때문에 잘 어울린다고 볼 수 있습니다.

앞서 설명한 바와 같이 GitOps란 Git 저장소에 있는 내용을,  쿠버네티스 클러스터에 그대로 반영해주는것입니다. 이것을 그림으로 표현하자면 아래와 같습니다.

Git 저장소에 있는것을 쿠버네티스 클러스터에 동기화 합니다.CI / CD 파이프라인

![GitOps%20and%20Kubernetes/Untitled.png](/assets/img/2019/12/gitops-000.png)

일반적으로 많이 사용하는 CI/CD 파이프라인을 대략적으로 그린다면, 다음과 같을 것입니다.

![GitOps%20and%20Kubernetes/Untitled%201.png](/assets/img/2019/12/gitops-001.png)

개발자가 소스 코드를 작성하고,  Git 저장소에 올립니다. 그러면 Jenkins, CircleCI 같은 CI 툴에 의해서 테스트와 빌드 같은 작업이 실행된 후, 생성한 컨테이너 이미지를 컨테이너 저장소에 업로드 합니다. 

그런 다음 CI/CD 툴에서 업로드된 컨테이너 이미지의 정보를 참조하여, 대상 서버에 배포를 하는 것입니다.

GitOps는 이러한 파이프라인의 배포 부분에서 약간 다르게 작동합니다.

컨테이너 이미지를 컨테이너 저장소에 업로드 한 후, 매니페스트가 저장되어 있는 Git 저장소를 가져옵니다. 그리고 매니페스트의 특정 부분(예를 들면 이미지 태그)을 업데이트 한 후, Git 저장소에 올리고 작업을 종료하게 됩니다.

![GitOps%20and%20Kubernetes/Untitled%202.png](/assets/img/2019/12/gitops-002.png)

매니페스트가 정의되어 있는 Git 저장소가 변경되면, Git 저장소의 내용과 쿠버네티스 클러스터를 동기화 해주는 에이전트가 변경 내역을 쿠버네티스 클러스터에 반영해 주게 되는 것입니다.

이 문서에는 편의를 위해서, Git 저장소의 내용과 쿠버네티스 클러스터를 동기화 해주는 역할을 하는 에이전트를 GitOps 오퍼레이터(Operator)라고 부르도록 하겠습니다.

이러한 과정을 간단히 코드로 표현하면  다음과 같습니다.

### 새로운 컨테이너 이미지를 빌드하고 푸시하기

    docker build -t example/hello:v2.0 .
    docker push example/hello:v2.0

### 매니페스트를 수정하고 git 저장소에 푸시하기

    git clone https://github.com/example/hello-config.git
    cd hello-config
    
    kubectl patch --local -f config-deployment.yaml -p '{"spec":{"template":{"spec":{"containers":[{"name":"hello","image":"example/hello:v2.0"}]}}}}' -o yaml
    
    git add . -m "Update hello to v2.0"
    git push

이렇게 매니페스트가 저장되어 있는 git 저장소가 업데이트가 되면, GitOps Operator가 해당 내용을 쿠버네티스 클러스터에 반영 즉, 동기화 해주는 것입니다.

# GitOps Operator

GitOps 오퍼레이터에 대해서 좀 더 알아 보도록 하겠습니다.  앞서 살펴본 봐와 같이 GitOps 오퍼레이터는 Git 저장소 있는 매니페스트를 쿠버네티스 클러스터에 반영해 주는 역할을 합니다.

다음과 같이, 쿠버네티스의 `CronJob` 을 이용해서 간단히 구현해 볼 수도 있습니다.

    apiVersion: batch/v1beta1
    kind: CronJob
    metadata:
      name: gitops-cron-job
      namespace: gitops
     spec:
       schedule: "*/10 * * * *"
       backoffLimit: 0
       jobTemplate:
         spec:
           template:
             spec:
               containers:
               - name: gitops-operator
                 image: gitops/operator:latest
                 command: [sh, -e, -c]
                 args:
                 - git clone http://github.com/gitops/hello.git /tmp/hello
                   find /tmp/hello -name '*.yaml' -exec kubectl apply -f {} \;

10초 마다 주기적으로, Git 저장소의 내용을 가져와서 쿠버네티스 클러스터에 적용 하고 있는 것입니다.

이런 식으로 직접 구현해서 사용할 수도 있지만, 보안이나 모니터링 등 여러 측면에서 불편하기 때문에 기존에 만들어진 GitOps 오퍼레이터를 사용할 것입니다.

널리 알려진 GitOps 오퍼레이터는 `Weavework`에서 만든 `Flux`와 `Intuit`에서 만든 `ArgoCD`가 있습니다. 이 문서에서는 `Argo CD`를 사용합니다.

![GitOps%20and%20Kubernetes/Untitled%203.png](/assets/img/2019/12/gitops-003.png)

# Argo CD

`Argo CD`는 GitOps스타일의 배포를 지원하는 CD 도구입니다. 원하는 설정 사항을 변경하여 Git에 푸시하면, 자동으로 쿠버네티스 클러스터의 상태가 Git에 정의된 상태로 동기화 됩니다.

즉, 지정한 대상 환경에 애플리케이션을 원하는 상태로 자동으로 배포하는 것입니다. 

그뿐만 아니라, 멀티 클러스터 관리/배포 기능도 가지고 있습니다. 그리고 SSO 연동과 멀티 테넌시를 지원하고, RBAC을 사용할 수도 있는 등 여러가지 장점을 가지고 있습니다.

![GitOps%20and%20Kubernetes/argocd-ui.gif](/assets/img/2019/12/argocd-ui.gif)

출처 : [https://argoproj.github.io/argo-cd/](https://argoproj.github.io/argo-cd/)

# GitOps 구성하기

### Git 저장소

`Argo CD`를 이용해서, GitOps 스타일의 CI/CD 파이프라인을 구성하는 방법에 대해서 알아보겠습니다.

이 예제에서는 두 개의 Git 저장소를 사용합니다.

- app 저장소 : 애플리케이션 소스 코드를 저장하고 있습니다.
- config 저장소 : 쿠버네티스 배포 용 매니페스트를 저장하고 있습니다.

물론 애플리케이션 소스 코드와 매니페스트를 단일 저장소에 저장할 수도 있습니다. 하지만 서로 다른 곳에 저장하는 것이 더 좋기 때문에 분리하는 것을 추천합니다.

app 저장소와 config 저장소를 분리하는 가장 큰 이유는, 용도와  생명 주기가 다르기 때문입니다.  app 저장소는 실제 개발자가 주로 사용하며, 애플리케이션 소스코드를 저장하고 있고, config 저장소는 주로 CI/CD 툴 같은 자동화 시스템에서 주로 사용하기 때문입니다.

참고할 예제 소스 코드 저장소는 [https://github.com/kangwoo/hello-go](https://github.com/kangwoo/hello-go) 이고,  매니페스트 저장소는 [https://github.com/kangwoo/hello-go-deploy](https://github.com/kangwoo/hello-go-deploy) 입니다.

GitOps를 구성하기에 앞서, 먼저 결정해야 할 사항이 하나 있습니다. 그것은 바로 매니페스트를 어떻게 만들지 입니다. 기존에 사용하던 쿠버네티스트의 매니페스트를 그대로  사용해도 됩니다. 예를 들면 `deployment.yaml`, `service.yaml`, `ingress.yaml` 등등 기존에 사용하던 형태 그대로 만들어도 됩니다. 하지만, 배포 환경이 여러개가 된다는 등의  환경 별로 파일을 각자 만들어줘야하는 경우가 생길 수 있습니다. 물론 환경별로 파일을 따로 따로 만들 수도 있지만, 상당히 번거롭습니다.  중복되는 내용이 더 많을 것이기 때문입니다. 그래서  템플릿 같은 것을 사용하면 좀 더 편하게 만들 수 있습니다. 바로 `Kustomize`나 `Helm` 등의 툴을 이용하는것입니다. 다행히도 `Argo CD` 에서는 `Kustomize`나 `Helm`, `Ksonnet` 등을 지원하기 때문에, 별다른 노력 없이 해당 툴들을 사용할 수 있습니다. 예제에서는 `Kustomize` 를 사용하도록 하겠습니다.

우선 app 저장소에 소드 코드를 올립니다.  

 CI 툴을 이용해서, 해당 저장소에서 소스 코드를 클론한다음, 컨테이너 이미지를 빌드하고 푸시하는 파이프라인을 만듭니다. CI 툴은 Jenkins나 CircleCI, Tekton등 아무거나 사용해도 무방합니다. 여기서 중요하게 다를 부분은 GitOps 부분이기 때문에, 컨테이너 이미지를 빌드해서 푸시하는 것에 대해서는 자세히 다루지 않겠습니다.

go 로 작성된  Hello를 출력하는 main.go가 있고, 컨테이너 이미지 빌드를 위한 Dockerfile 이 있습니다. 그리고 젠킨스에서 CI 파이프라인을 정의한 Jenkinsfile 이 있습니다.

main.go

    package main
    
    import (
      "fmt"
      "net/http"
    )
    
    func main() {
      http.HandleFunc("/", hello)
      http.ListenAndServe(":8080", nil)
    }
    
    func hello(w http.ResponseWriter, r *http.Request) {
      fmt.Fprintf(w, "Hello, %s", r.URL.Path[1:])
    }

Dockerfile

    FROM golang:alpine as builder
    RUN mkdir /build
    ADD . /build/
    WORKDIR /build
    RUN go build -o main .
    
    FROM alpine
    ENV USER_UID=1001 \
        APP_DIR=/app 
    RUN mkdir -p ${APP_DIR} && chown ${USER_UID}:0 ${APP_DIR} && chmod ug+rwx ${APP_DIR}
    
    USER ${USER_UID}
    COPY --from=builder /build/main ${APP_DIR}/
    WORKDIR ${APP_DIR}
    EXPOSE 8080
    CMD ["./main"]

Jenkinsfile

    def podLabel = "worker-${UUID.randomUUID().toString()}"
    
    podTemplate(label: podLabel, containers: [
      containerTemplate(name: 'docker', image: 'docker', command: 'cat', ttyEnabled: true),
      containerTemplate(name: 'tools', image: 'argoproj/argo-cd-ci-builder:v1.0.1', command: 'cat', ttyEnabled: true),
    ],
    volumes: [
      hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
    ]) {
      node(label) {
        def myRepo
        stage('Checkout') {
          myRepo = checkout scm
        }
    
        def gitCommit = myRepo.GIT_COMMIT
        def shortGitCommit = "${gitCommit[0..7]}"
        def imageTag = shortGitCommit
    
        stage('Image Build') {
          container('docker') {
            sh "docker build . -t kangwoo/hello-go:${imageTag}"
          }
        }
    
        stage('Image Push') {
          container('docker') {
            sh "docker push kangwoo/hello-go:${imageTag}"
          }
        }
    
        stage('Deploy to dev') {
          steps {
            withCredentials([usernamePassword(credentialsId: 'my-git', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PWD')]) {
              container('tools') {
                sh "git clone https://${GIT_USER}:${GIT_PWD}@github.com/kangwoo/hello-go-deploy.git"
                sh "git config --global user.email '${GIT_USER}@mycompany.com'"
    
                dir("hello-go-deploy") {
                  sh "cd ./overlays/dev && kustomize edit set image kangwoo/hello-go:${imageTag}"
                  sh "git commit -am 'Publish new version ${imageTag} to dev' && git push || echo 'no changes'"
                }
              }
            }
          }
        }
    
      }
    }

`Jenkinsfile`에서  중요하게 봐야할 부분은 `stage('Deploy to dev')` 입니다. 

특정 이미지 태그를 만들어서, 컨테이너 이미지를 빌드하고 푸시한 다음, 해당 스테이지가 실행됩니다.  매니페스트가 정의되어 있는 Git 저장소를 클론하고, `kustomize edit set image` 명령어를 실행해서, 사용할 이미지 정보를 업데이트 해줍니다. 그런 다음 `git` 명령어를 이용해서 Git config 저장소에 푸시합니다. 

변경된 매니페스트가 Git 저장소에 푸시되면,  `Argo CD`가  변경된 점을 파악해서, 쿠버네티스 클러스터와 동기화 해줍니다.

참고로 `git`과 `kustomize` 명령어를 쉽게 사용하기 위해서, `argoproj/argo-cd-ci-builder:v1.0.1` 이미지를 사용하였습니다.

그 다음, 매니페스트를 만를고, config 저장소에 올립니다.

매니페스트는 `kustomize`를 사용해서 만들었습니다. 간단히 구조를 살펴보면, `base`와 `overlays` 디렉토리를 가지고 있습니다.

    .
    ├── base
    │   ├── deployment.yaml
    │   ├── kustomization.yaml
    │   └── service.yaml
    └── overlays
        ├── dev
        │   ├── deployment-patch.yaml
        │   └── kustomization.yaml
        └── prod
            ├── deployment-patch.yaml
            └── kustomization.yaml

`base` 디렉토리에는 리소스를 정의한 파일이 있습니다.  바로 `deployment.yaml`과 `service.yaml` 파일 입니다. 이 파일들에는 쿠버네티스의 `Deployment`와 `Service`를 생성하기 위한 명세가 담겨 있습니다. 그리고, `kustomization.yaml` 라는 파일도 존재하는데, 이 파일은 `kustomize` 에서 사용하는 파일로서, 기본적인 메타 정보와 어떠한 리소스들을 사용할지에 대한 정보가 담겨 있습니다.

`overlays` 디렉토리는 다시 `dev`와 `prod` 디렉토리로 나누어 집니다. 개발과 프로덕션 환경으로 사용하기 위해서 두 개로 나눈것입니다.  `dev`와 `prod` 디렉토리에는 각각 메타 정보를 담긴 `kustomization.yaml` 파일과, 환경별로 패치할 내용이 담긴 deployment-patch.yaml 파일이 존재합니다. 예를 들면, 개발 환경에 반영될때에는, `base` + `overlays/dev` 가 합쳐진 결과가 반영이 되는 것입니다.

`Argo CD` 는 `kustomize` 를 지원하기 때문에, `overlays/dev` 같이 해당 디렉토리를 지정해주면, 합쳐진 결과가 자동으로 쿠버네티스 클러스터에 동기화 됩니다.

### Argo CD

Git 저장소에 있는 내용을 쿠버네티스 클러스터에 자동으로 동기화 하기 위해서 `Argo CD` 에 설정을 추가하겠습니다. 

`Argo CD` 웹 화면에 접속한 후, 로그인을 한다음, `New App`을 클릭합니다.

![GitOps%20and%20Kubernetes/Untitled%204.png](/assets/img/2019/12/gitops-004.png)

그리고 아래 값들을 입력한 후, `CREATE` 버튼을 클릭하면 애플리케이션이 생성됩니다.

![New App](/assets/img/2019/12/argocd-fields.png)

웹 화면을 사용하지 않고, 직접 CR을 생성해서 사용할 수도 있습니다.

아래처럼 `Application` 리소스를 정의한 후, `Argo CD` 가 설치된 네임스페이스에 해당 리소스를 생성해 주면 됩니다.

    cat <<EOF | kubectl -n argocd apply -f -
    apiVersion: argoproj.io/v1alpha1
    kind: Application
    metadata:
      name: hello-go
    spec:
      destination:
        namespace: default
        server: https://kubernetes.default.svc
      project: default
      source:
        path: overlays/dev
        repoURL: https://github.com/kangwoo/hello-go-deploy.git
        targetRevision: HEAD
      syncPolicy:
        automated: {}
    EOF

애플리케이션이 정상적으로 생성되면, 화면에서 확인할 수 있습니다. 애플리케이션 이름을 클릭하면  다음과 같은 상세 내용을 볼 수 있습니다.

![GitOps%20and%20Kubernetes/Untitled%205.png](/assets/img/2019/12/gitops-005.png)
