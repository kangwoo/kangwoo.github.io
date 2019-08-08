---
title:  "Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA"
classes: wide
date: 2019-07-02T07:21:10+09:00
categories: [data science, machine learning]
tags: [machine learning, data science, tensorflow]
mathjax: true
---

##  경고 메시지
텐서플로(TensorFlow) 라이브러리를 사용해서, 머신러닝 코드를 실행하다 보면 아래와 같은 경고 메시지가 출력되는 경우가 있다.

```bash
I tensorflow/core/platform/cpu_feature_guard.cc:141] Your CPU supports instructions that this TensorFlow binary was not compiled to use: SSE4.1 SSE4.2 AVX AVX2 FMA
```

굳이 직역을 해보자면, '이 텐서플로 바이너리는 당신의 CPU가 제공하는 명령어(instruction)들을 사용하도록 컴파일되어 있지 않습니다.' 라는 뜻일 것이다.

## 해결 방법
해결 방법은 옵션을 추가해서 텐서플로를 다시 빌드 하면 된다.

예를 들어 `SSE4.1`을 지원하는 CPU라면 `--copt=-msse4.1`을 추가하면 된다.


## 참고 자료
- <https://stackoverflow.com/questions/47068709/your-cpu-supports-instructions-that-this-tensorflow-binary-was-not-compiled-to-u>
- <https://stackoverflow.com/questions/41293077/how-to-compile-tensorflow-with-sse4-2-and-avx-instructions>
- <https://github.com/lakshayg/tensorflow-build>
