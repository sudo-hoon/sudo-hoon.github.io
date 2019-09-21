---
layout: post
title: "Extended Kalman Filter"
date: "2019-09-20"
# slug: "example_content"
description: "This post is based on SLAM lecture"
category: 
  - SLAM
# tags will also be used as html meta keywords.
tags:
  - SLAM 
comments: true
use_math: true
---

> 본 게시글은 Cyrill Stachniss의 [Robot mapping(WS 2013/14)](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 요약 및 정리한 글입니다. 
> 틀린 내용을 지적해주시면 감사드리겠습니다.

# Extended Kalman Filter

## Kalman filter

- Bayes filter의 일종
- motion model과 observation model이 linear model이고 노이즈가 gaussian distribution 일 때,  
  최적의 솔루션이다.

### Linear model

##### motion model : $x_t=A_tx_{t-1}+B_tu_t+\epsilon_t $

##### observation model : $z_t=C_tx_t+\delta_t$

* $A_t$ : command과 노이즈가 없더라도 바람 등으로 인해 state가 변할 수 있음을 표현.
* $B_t$ : 환경의 영향이 없다면 identity matrix, 또는 바람의 영향 같은 것을 고려하여 표현할 수 있다.
* $C_t$ : state를 sensor 값으로 환산, linear 모델이기는 하나 실제 sensor는   
원통 좌표계의 형태로 값을 얻으므로 state가 cartesian coord. 라면 linear할 수 없다.
* $\epsilon_t$ : motion의 noise. kalman filter에서는 independent normal distribution을 가정, $N(0,R_t)$
* $\delta_t$ : sensor의 noise. 역시 independent normal zero mean distribution을 가정, $N(0, Q_t)$  
  

#### Linear motion model

* $p(x_t|u_t, x_{t-1})$  
  $=det(2\pi R_t)^{-1/2}exp(-\frac{1}{2}(x_t-\mu)^TR_t^{-1}(x_t-\mu))$  
  $=det(2\pi R_t)^{-1/2}exp(-\frac{1}{2}(x_t-A_tx_{t-1}-B_tu_t)^TR_t^{-1}(x_t-A_tx_{t-1}-B_tu_t)$
* motion model의 mean :  $\mu_t=E(x_t)=E(A_tx_{t-1}+B_tu_t+\epsilon_t)= A_tx_{t-1}+B_tu_t$
* 즉, 다음 robot의 평균 position은 motion 노이즈 없이 command에 따라 움직였을 때의 위치이다.  
  

#### Linear observation model

- $p(x_t|u_t, x_{t-1})$  
  $=det(2\pi Q_t)^{-1/2}exp(-\frac{1}{2}(z_t-\mu)^TQ_t^{-1}(z_t-\mu))$  
  $=det(2\pi Q_t)^{-1/2}exp(-\frac{1}{2}(z_t-C_tx_t)^TQ_t^{-1}(z_t-C_tx_t))$
  
- motion model의 mean :  $\mu_t=E(A_tx_{t-1}+B_tu_t+\epsilon_t)= A_tx_{t-1}+B_tu_t$   
  

  

### Kalman filter Algorithm  

1. $\bar{u_t}=A_tx_{t-1} + B_tu_t$

2. $\bar{\sum{} _t}=A_t\sum{}_{t-1}A_t^T + R_t$

3. $K_t=\bar{\sum{}_t}C_t^T(C_t\bar{\sum{}_t}C_t^T+Q_t)^{-1}$

4. $\mu_t=\bar{\mu_t}+K_t(z_t-C_t\bar{\mu_t})$

5. $\sum{}_t=(I-K_tC_t)\bar{\sum{}_t} $ 

   

- 1~2 : Prediction step, 3~5 : Correction step
- 각 항의 수학적인 의미  
  
1. 다음 robot의 평균 위치는 노이즈 없는 motion system을 통과한 위치와 같다.
     - 즉, 이상적으로 움직인 위치
  2. 새로운 위치에서의 noise covariance는 이전 covariance의 __linear mapping+ motion noise__
     - 노이즈의 크기는 이전보다 증가한다.
  3.  $K$ (Kalman gain)
     - Perfect sensor($Q_t=0$) : $K_t=C_t^{-1}$ => $\mu_t=C_t^{-1}z_t$, $\sum{}_t = 0$  
       -> $bel(x_t)$ 의 평균은 prediction에 의존하지 않으며 센서에서 읽힌 값 그대로 위치를 추정함.  
       -> Covariance는 0, 즉, 위치를 정확하게 추정한다. (map을 알고 있기 때문에)
     - Poor sensor($Q_t -> \infty$) :  $K_t -> 0$ => $\mu_t=\bar{\mu_t}$, $\sum{}_t = \bar{\sum{}_t}$
       -> $bel(x_t)$ 의 평균과 covariance는 오로지 prediction에만 의존한다.  
  4. robot의 pose의 평균은 예측된 pose의 평균에 센서로 추정한 pose로 보정한다.
     -  $\bar{z_t}=z_t-C_t\bar{\mu_t}$ 이므로  $K_t(z_t-C_t\bar{\mu_t})=K_t(z_t-\bar{z_t})$ 이다.  
       -> 즉,  prediction 값으로 센서 값을 예측하고 실제 센서 값과의 차이를 구한다.  
       -> 그 결과에 kalman gain을 곱함으로 써, 센서의 신뢰도 만큼 보정의 정도가 커진다.  
       -> 또한 kalman gain은 센서 값의 차이를 robot의 pose 값으로 변환한다.  
  5. Correction 된 robot pose의 covariance 값은 항상 prediction보다 작아진다.
     - 센서 값의 불확실성이 작을수록 robot pose의 covariance는 더 작아진다.

![weighted sum between prediction and measurement](https://sudo-hoon.github.io/images/posts/SLAM/2019-09-20-EKF-SLAM/img1.png)



## Extended Kalman filter

- Kalman filter의 한계 : motion model 과 sensor model이 linear model이다.
- 로봇의 pose는 좌표가 cartesian으로 이루어지지만 로봇의 움직임 또는 센서의 측정은 각도를 바탕으로 transforamtion이 이루어지는 경우가 많다.   
  -> 즉, linear transformation이 불가능.

#### nonlinear motion and sensor model

1. motion model : $x_t = g(u_t, x_{t-1}) + \epsilon_t$ 
2. sensor model : $z_t = h(x_{t}) + \delta_t$

- Kalman filter는 linear system + gaussian distribution에 대해 bayes filter의 최적의 solution이다.

- 위와 같이 비선형 시스템을 통과한 gaussian distribution은 형태를 유지하지 못한다.

- 비 선형 시스템일 때, 어떻게 kalman filter를 사용할 것인가?

  - non-linear system을 선형화한다.

- non-linear system을 통과한 평균 위치 값을 중심으로 선형화를 하자!

#### Linearization

- 선형화는 Taylor expansion을 한 후 jacobian을 계산하는 과정으로 이루어진다. 

1. First order Taylor Expansion
   - Prediction :  $g(u_t,x_{t-1}) ~= g(u_t,\mu_{t-1})+\frac{\partial g(u_t,\mu_{t-1})}{\partial x_{t-1}}(x_{t-1}-\mu_{t-1})=g(u_t,\mu_{t-1})+G_t(x_{t-1}-\mu_{t-1})$  
     -> $\mu_{t-1}$ 을 기준으로 expansion
   - Correction : prediction : $h(x_t) ~= h(\bar{\mu_t})+\frac{\partial h(\bar{\mu_t})}{\partial x_t}(x_t-\bar{\mu_t})=h(\bar{\mu_t})+H_t(x_t-\bar{\mu_t})$  
     -> $\bar{\mu_{t}}$ 를 기준으로 expansion
2. Jacobian[^1]
   - 미소 구간을 선형화하여 좌표계를 변환할 때 사용한다.
   - 2차 선형화를 할 때는 hessian까지 계산이 필요하다.

![Jacobian](/images/posts/SLAM/2019-09-20-EKF-SLAM/img2.png)

- 선형화는 mean position에서 멀어질 수록 부정확해지므로 pose의 노이즈가 작을수록 EKF의 성능은 좋아진다.

![low noise](/images/posts/SLAM/2019-09-20-EKF-SLAM/img3.png)



### EKF Model

#### Linearized Motion Model

- $x_t = g(u_t, \mu_{t-1}) + G_t(x_{t-1}-\mu_{t-1})$
- $p(x_t|u_t, x_{t-1})$  
  $=det(2\pi R_t)^{-1/2}exp(-\frac{1}{2}(x_t-\mu)^TR_t^{-1}(x_t-\mu))$  
  $=det(2\pi R_t)^{-1/2}$  
  $exp(-\frac{1}{2}(x_t-(g(u_t, \mu_{t-1}) + G_t(x_{t-1}-\mu_{t-1})))^TR_t^{-1}(x_t-(g(u_t, \mu_{t-1}) + G_t(x_{t-1}-\mu_{t-1})))$

#### Linearized Observation Model

- $z_t = h(\bar{\mu_t})+H_t(x_t-\bar{\mu_t})$

- $p(x_t|u_t, x_{t-1})$  
  $=det(2\pi Q_t)^{-1/2}exp(-\frac{1}{2}(z_t-\mu)^TQ_t^{-1}(z_t-\mu))$  
  $=det(2\pi Q_t)^{-1/2}$  
  $exp(-\frac{1}{2}(z_t-(h(\bar{\mu_t})+H_t(x_t-\bar{\mu_t})))^TQ_t^{-1}(z_t-(h(\bar{\mu_t})+H_t(x_t-\bar{\mu_t}))))$

#### EKF Algorithm

- 직관적으로 g,h는 각 모델의 state의 translation, 즉, 평균의 변화에 해당되고  
  G,H는 각 모델 state의 distribution을 선형 시스템에 통과시키는 것에 해당된다.
- 따라서 KF Algorithm에서 약간의 변화만 주면 쉽게 구할 수 있다.

1. $\bar{\mu_t}=g(u_t,\mu_{t-1})$
2. $ \bar{\sum{} _t}=G_t\sum{}_{t-1}G_t^T + R_t $
3. $K_t=\bar{\sum{}_t}H_t^T(H_t\bar{\sum{}_t}H_t^T+Q_t)^{-1}$
4. $\mu_t=\bar{\mu_t}+K_t(z_t-h(\bar{\mu_t}))$
5. $\sum{}_t=(I-K_tH_t)\bar{\sum{}_t}$

### EKF Summary

1. 비 선형 시스템에서 선형화를 통해 Kalman filter를 사용하는 방법
2. 비 선형성이 크지 않거나 model의 variance가 크지 않을 때, 잘 작동한다.
3. Complexity = $O(K^{2.4}+N^2)$
   - K : dimension of Measurement
     - matrix inversion은 cubic complexity
     - kalman gain을 구할때 matrix inversion
   - N : dimension of State (robot pose + landmark)
     - multiply operation으로 제곱의 복잡도
       - __왜 제곱의 복잡도지? 세제곱 아닌가?-----?__

[^1]: https://wikidocs.net/4053