---
title:  "Linux buffer/cache 비우기"
classes: wide
date: 2019-08-07T12:30:50+09:00
categories: [devops, linux]
tags: [linux]
---


리눅스(linux)에서 `top`이나 `free` 명령어로 메모리 상태를 확인할 수 있다.
버퍼와 캐시는 자주 사용하는 파일등의 정보를 저장하여 성능을 빠르게 유지할 수 있는 장점이 있지만, 너무 많아지면 가용 메모리의 부족으로 다른 문제를 야기시킬 수 있다.
그래서 버퍼/캐시의 사용량이 너무 높을 경우 해당 기능을 끄거나, 주기적으로 비움으로 성능을 개선할 수 있다.

아래 명령어로 캐시를 비울 수 있다.
`sync`를 사용하는 이유는 캐시 메모리에 올라간 데이터를 디스크로 옮겨준다. 이 과정을 거치치 않고 캐시를 삭제하면, 데이터가 유실될 수도 있다.


- pagecache 비우기
```bash
sync
echo 1 > /proc/sys/vm/drop_caches
또는
sync
sysctl -w vm.drop_caches=1
```

- dentries, inodes 비우기
```bash
sync
echo 2 > /proc/sys/vm/drop_caches
또는
sync
sysctl -w vm.drop_caches=2
```

- pagecache, dentries, inodes 모두 비우기
```bash
sync
echo 3 > /proc/sys/vm/drop_caches
또는
sync
sysctl -w vm.drop_caches=3
```


### 주기적으로 캐시 비우기
주기적으로 캐시를 삭제하고 싶으면, `crontab -e` 명령어를 사용하면 된다.
```bash
0 3 * * 0 sync && echo 3 > /proc/sys/vm/drop_caches # 매일 오전 3시에 캐시 비우기
```
