---
title:  "Spring Boot 시작시 /dev/./urandom을 사용하는 이유"
classes: wide
date: 2018-02-06T13:15:00+09:00
categories: [devops, spring boot]
tags: [spring boot, java, linux]
---

SpringBoot 실행 예제를 보면 `-Djava.security.egd=file:/dev/./urandom` 플래그를 간혹 볼 수 있습니다.
이 플래그가 왜 필요한지에 대해서 간단히 설명해 보겠습니다.
<br/>
스프링 부트를 이용해서 웹 애플리케이션을 만들 때, 기본적으로 톰캣을 이용하게 됩니다.
이 톰캣은 자바의 `SecureRandom` 클래스를 사용해서, 세션 아이디 같은 것을 생성하게 됩니다.
리눅스(linux) 환경의 경우, `SecureRandom` 클래스는 안전한 난수 생성을 위해서 `/dev/random`을 사용합니다.
<br/>
리눅스에서는 `/dev/random`과 `/dev/urandom`이라는 두 개를 난수발생기(PRNG : pseudo-randum number generator)를 제공 합니다.
이 난수 발생기는 디바이스 드라버에서 발생하는 입력 신호 등을 이용해서 난수를 발생시킵니다.
즉, 키보드나 마우스 클릭 같은 디바이스의 입줄력 신호를 엔트로피 풀(Entropy pool)에 저장하고, 난수를 생성할때 엔트로피 풀에서 필요한 크기 만큼 가져다 사용하게 됩니다.
<br/>
문제는 이 엔트로피 풀에 저장되어 있는 데이터가 부족할 때 발생합니다.
`/dev/random` 경우 엔트로피 풀에 필요한 크기 만큼의 데이터가 부족할 경우, 블록킹(blocking) 상태로 대기하게 됩니다.
이런 경우 애플리케이션이 행(hang)에 걸린것 처럼 멈처버리는 현상이 발생합니다.
이 문제를 해결하기 위해서 `/dev/urandom`을 사용한 것입니다.
`/dev/urandom` 같은 경우에는 엔트로피 풀에 있는 데이터가 충분하지 않아도, 난수를 생성해버립니다. (하지만 `/dev/random`에 비해서는 난수의 무작위성이 떨어집니다.)
그래서 난수로 인한 애플리케이션의 블록킹 사태를 막기 위해서 `/dev/urandom`를 사용하기도 합니다.
<br/>
그런데 여기서 한가지 의문 사항이 드는데, 왜 `/dev/urandom`이 아니라 `/dev/./urandom`을 사용했을까요?
그 이유는 [자바 버그](https://bugs.java.com/bugdatabase/view_bug.do?bug_id=6202721)로 인해서 입니다. Java5 이휴의 특정 버전에서는 `/dev/urandom`을 설정하면, `/dev/random`로 인식해 버리는 버그가 있습니다.

