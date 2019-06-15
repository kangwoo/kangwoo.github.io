---
title:  "docker 명령어"
classes: wide
date: 2018-06-15T20:13:00+09:00
categories: [devops, docker]
tags: [docker, container]
---

## 도커 이미지 관련 명령어
- docker login [repository] : 저장소(repository)에 로그인한다. 저장소 주소를 적지 않으면 Docker Hub repository 로 로그인한다.
- docker create [image] : 해당 이미지로부터 새로운 컨테이너를 생성한다.
- docker pull [image] : 이미지를 저장소로부터 가져온다.
- docker push [image] : 이미지를 저장소에 올린다.
- docker tag [source] [target] : 원본 이미지 새로운 태그를 부여한다.
- docker search [term] : 해당 단어로 저장소에 있는 이미지를 검색한다.
- docker images : 로컬 시스템에 저장되어 있는 이미지 목록을 보여준다.
- docker history [image] : 해당 이미지의 히스토리를 보여준다.

## 도커 컨테이너 관련 명령어
- docker ps : 현재 실행중인 컨테이너 목록을 보여준다.
- docker run [image] : 해당 이미지로 도커 컨테이너를 실행한다.
- docker start [container] : 도커 컨테이너를 시작한다.
- docker stop [container] : 도커 컨테이너를 중지한다. (SIGTERM -> SIGKILL)
- docker stop $(docker ps -q) : 현재 작동하는 모든 도커 컨테이너를 중지한다.
- docker kill [container] : 도커 컨테이너를 강제로 중지한다. (SIGKILL)
- docker inspect [container] : 컨테이너의 상세 정보를 보여준다.
- docker rm [container] : 중지된 도커 컨테이너를 삭제한다.
- docker rm $(docker ps -a -q) : 중지된 모든 도커 컨테이너를 삭제한다.
- docker exec -iㅓ [container] [command] : 대상 도커 컨테이너에 명령어를 실행한다.

## 기타 명령어
- docker info : 도커 상세 정보를 보여준다.
- docker version : 도커 버전을 보여준다.
