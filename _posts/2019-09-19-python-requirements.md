---
title:  "파이썬 requirements.txt "
classes: wide
date: 2019-09-09T20:17:00+09:00
categories: [devops, python]
tags: [python]
---

파이썬 환경에서 자신이 사용하고자 하는 파이썬 패키지를 일일히 설치해주는것은 불편한 일이다.
`requirements.txt`을 사용하면 손쉽게 패키지를 설치할 수 있다.

## `requirements.txt` 만들기
자신의 구성 환경을 `requirements.txt`로 만들려면, 다음과 같은 명령어를 실행한다.
```bash
pip freeze > requirements.txt
```

사용자 디렉토리에 설치된 패키지만을 가져오로면 `--user` 옵션을 사용하면 된다.
```bash
pip freeze --user > requirements.txt
```


## `requirements.txt`로 패키지 설치
`requirements.txt`로 지정한 환경에 패키치를 설치하려면, 다음과 같은 명령어를 실행한다.
```bash
pip install -r requirements.txt
```
