---
title:  "오차 함수 (error function) / 손실 함수 (loss function)"
classes: wide
date: 2019-07-02T07:21:10+09:00
categories: [devops, machine learning]
tags: [machine learning, data science, loss function]
mathjax: true
---

## 오차 함수 (error function) / 손실 함수 (loss function) 
오차는 학습 목표값과 실제 값 간의 차이를 의미한다.

간단히 생각해보면 오차함수는 단순히 (목표 값 - 실제 값)이다. 언뜻 보기엔 충분한 합리적인 오차함수 이지만, 
전체 노드의 오차를 구하기 위해 합을 계산해보면 그 값이 0이 발생할 수도 있다.
즉, 양의 오차와 음의 오차가 서로를 상쇄할 수 있는 것이다.

이러한 문제점을 없애기 위해서, 절댓값 손실(absolute loss, L1 Loss)을 취하거나, 
두 값의 차이에 제곱을 하는 제곱 손실(Squared loss, L2 Loss)방식을 사용할 수 있다.

절댓값을 사용하는 방식은, 최저점 근처에서 기울기 작아지지 않기 때문에 오버슈팅할 가능성이 높지만 이상치(outlier) 덜 민감하다.
이에 반해서 제곱오차 방식은 최저점에 접근함에 따라 이굴기가 작아지므로 목표물을 오버슈팅할 가능성이 낮지만, 이상치에 민감하다.


### 평균 제곱 오차(MSE, Mean Squared Error)
목표값과 예측값의 차이를 제곱한 값의 편균이다.


L(Y, hat{y}) = \frac{1}{n} \sum_{i=1}^n (y_i - \hat{y}_i)^2

텐서플로에서 표현하면 다음과 같다.
```python
loss = tf.reduce_mean(tf.square(y - y_pred))
```
### 교차 엔트로피 오차(CEE, Cross Entropy Error)
교차 엔트로피는 다음과 같이 정의된다.

H(p, q) = - \sum_x p(x) \log q(x) 

```python
loss = -y*tf.log(y_pred) - (1-y)*tf.log(1-y_pred)
loss = tf.reduce_mean(losgg)
```

텐서플로에서 표현하면 다음과 같다.
```python
loss = tf.nn.sigmoid_cross_entropy_with_logits(labels=y, logits=y_pred)
loss = tf.reduce_mean(loss)
```


# Refrences
- https://rdipietro.github.io/friendly-intro-to-cross-entropy-loss/
