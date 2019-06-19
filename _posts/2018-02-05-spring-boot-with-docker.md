---
title:  "Spring Boot with Docker
classes: wide
date: 2018-02-06T13:15:00+09:00
categories: [devops, spring boot]
tags: [spring boot, docker]
---


## Containerize
도커에는 이미지 파일 생성을 위해서 사용되는 `Dockerfile` 이라는 파일이 있습니다. 
스프링 부트 프로젝트를 위한 `Dockerfile`을 만들어 보겠습니다.
<br/>
`src/Dockerfile` 파일을 아래 내용을 참조해서 생성합니다.
```
FROM openjdk:8-jdk-alpine

ARG JAR_FILE

RUN mkdir -p /app

WORKDIR /app

ADD ${JAR_FILE} app.jar
ADD entrypoint.sh entrypoint.sh

EXPOSE 8080

ENTRYPOINT ["./entrypoint.sh"]
```
`ENTRYPOINT`에서 자바를 직접 실행시킬 수 있지만, 편의를 위해서 `entrypoint.sh` 파일을 만들어서 실행하도록 구성하였습니다.

`src/entrypoint.sh` 파일도 생성합니다. 기본적인 JVM 메모리 옵션과 DNS 캐시 비활성화 옵션이 포함되어 있습니다.
```bash
#!/bin/sh

set -e

GC_OPTS=${GC_OPTS:="-XX:+UseNUMA -XX:+UseG1GC"}
MEM_OPTS=${MEM_OPTS:="-Xms1024m -Xmx1024m"}
NET_OPTS=${NET_OPTS:="-Dsun.net.inetaddr.ttl=0 -Dsun.net.inetaddr.negative.ttl=0 -Djava.net.preferIPv4Stack=true"}
JVM_OPTS="-server $GC_OPTS $MEM_OPTS $NET_OPTS"

JAVA_OPTS=${JAVA_OPTS:="-server $GC_OPTS $MEM_OPTS $NET_OPTS "}

exec java -Djava.security.egd=file:/dev/./urandom -jar $JAVA_OPTS app.jar $@
```


## gradle docker plugin
그레이들(gradle)에서 도커 이미지를 빌드하기 위해서 [gradle-docker])https://github.com/palantir/gradle-docker) 플러그인을 설정에 추가해 줍니다. 
물론 그레이들을 이용하지 않고, 직접 도커 명령어로 이미지를 빌드 할 수도 있습니다.
<br/>
`buidl.gradle` 파일을 열어서, 플러그인 내용을 추가해 줍니다.
```groovy
plugins {
...
	id 'com.palantir.docker' version '0.22.1'
...
}
```

그리고 `docker` 빌드를 위한 task 내용도 추가해 줍니다.
```groovy
...
docker {
	name "repo-name/${bootJar.baseName}:${project.version}"
	dockerfile file('docker/Dockerfile')
	files 'docker/entrypoint.sh', "build/libs/${bootJar.archiveName}"
	buildArgs(['JAR_FILE': "${bootJar.archiveName}"])
}
...
```

설정이 완료되면, 다음 명령어로 도커 이미지를 빌드 할 수 있습니다.

```bash
./gradlew build docker 
```
<br/>
참고로 직접 도커 명령어로 빌드하려면 아래와 같이 실행하면 됩니다.
```bash
cd docker
docker -t repo-name/image-name:latest build --build-arg JAR_FILE=../build/libs/apps-0.0.1-SNAPSHOT.jar .
```
