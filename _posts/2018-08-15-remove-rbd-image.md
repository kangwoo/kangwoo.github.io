---
title:  "Ceph rbd 이미지 삭제하기"
classes: wide
date: 2018-08-15T15:13:00+09:00
categories: [devops, ceph]
tags: [kubernetes, ceph, rbd, ops-log]
---

# 상황
ceph의 물리적 저장 용량이 부족해서, 쿠버네티스의 PV를 삭제하였습니다.
PV 를 삭제하면 용량을 확보할 수 있을 줄 알았는데, 그렇지 않았습니다.
(PV의 설정을 잘못한건가???)

PV는 삭제되었지만, ceph에는 그대로 파일들이 남아있었습니다.
그래서 수작업으로 파일들을 삭제하였습니다.

# 작업 로그
## 용량 보기
```bash
$ ceph osd lspools
0 rbd,1 kube-xxx,2 kube-yyy,3 kube-zzz,

$ rados df
pool name KB objects clones degraded unfound rd rd KB wr wr KB
kube-xxx 203236277 50236 0 0 0 2656421 182868220 147679538 5831075965
kube-yyy 0 0 0 0 0 0 0 0 0
kube-zzz 181152993 44417 0 0 0 19325662 908304090 32869011 5467331110
rbd 0 0 0 0 0 0 0 0 0
total used 1212089217 94653
total avail 1112637423
total space 2324726640
```
## rbd 이미지 목록 보기
```bash
$ rbd list -p kube-zzz

kubernetes-dynamic-pvc-234b756d-25e7-11e9-b73c-0a580af40203
kubernetes-dynamic-pvc-3782458d-64d0-11e9-b73c-0a580af40203
kubernetes-dynamic-pvc-49fd82c0-6635-11e9-b73c-0a580af40203
kubernetes-dynamic-pvc-615d1d51-25e7-11e9-b73c-0a580af40203
kubernetes-dynamic-pvc-a62f7ee0-56b2-11e9-b73c-0a580af40203
kubernetes-dynamic-pvc-cb7d70f3-4081-11e9-b73c-0a580af40203
```

### rbd 이미지 삭제 하기
```bash
$rbd -p kube-zzz status kubernetes-dynamic-pvc-cb7d70f3-4081-11e9-b73c-0a580af40203
Watchers: none

$ rbd -p kube-zzz rm kubernetes-dynamic-pvc-cb7d70f3-4081-11e9-b73c-0a580af40203
Removing image: 100% complete...done.

```


# 참고 문서
 - <http://docs.ceph.com/docs/jewel/man/8/rbd/>
