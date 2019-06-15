---
title:  "디스크 사용량"
classes: wide
date: 2018-01-10duT20:25:00+09:00
categories: [devops, linux]
tags: [linux, disk]
---

# df : 디스크의 용량 확인
### 디스크의 용량을 보기 좋게 출력해 준다.
```bash
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda2       100G  6.6G   94G   7% /
devtmpfs         16G     0   16G   0% /dev
tmpfs            16G   12K   16G   1% /dev/shm
tmpfs            16G 1004K   16G   1% /run
tmpfs            16G     0   16G   0% /sys/fs/cgroup
/dev/vdc1       3.0T  528G  2.5T  18% /mnt
tmpfs           3.2G     0  3.2G   0% /run/user/0
tmpfs           3.2G     0  3.2G   0% /run/user/10000
...
```
### 현재 디렉토리를 포함하는 파티션의 용량을 출력해 준다.
```bash
$ df -h .
Filesystem      Size  Used Avail Use% Mounted on
/dev/vdc1       3.0T  528G  2.5T  18% /mnt
```


# du : 디렉토리 및 파일 용량 확인
### 현재 디렉토리에 있는 디렉토리 및 파일 용량을 보기 좋게 출력해 준다.
```bash
$ du -sh *
41G     01DAR7DQAV4TX9XMQA53H8P2KN
38G     01DCJ573XV9BS33MW7728Y4YR1
38G     01DCQYKT2QCZHF9J00H2W2GV62
38G     01DCXR075DH20RB24RAY92M129
1.6G    01DCZN81WMAWP6RCD79XKV8T0J
13G     01DCZNJW2WSNXC6GYNFY1AEH9M
1.7G    01DCZW3S4K9BM13KS623FDTZ26
0       lock
31G     wal
```

### 현재 디렉토리에 있는 디렉토리 및 파일 중에서 용량이 큰 순서대로 10를 출력해 준다.
```bash
$ du -sh * | sort -rh | head -n 10
```
