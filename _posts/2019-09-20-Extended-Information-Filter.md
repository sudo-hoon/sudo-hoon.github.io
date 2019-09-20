---
layout: post
title: "Extended Information Filter"
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

# Extended Information Filter

### Canonical Parameterization

- Gaussian distribution은 __moments__, 다시 말해서 $$\mu, \sum$$ 으로 표현된다
- Information space에서 gaussian은 __Information vector $$\xi$$ , Information Matrix, $$\Omega $$__ 로 표현 가능하다.
- Information Matrix, $$\Omega=\Sigma{}^{-1} $$
- Information Vector, $$\xi=\Sigma^{-1}\mu$$

##### Dual representation

- ![적은 노이즈 선형화](C:/Users/hyeji/Documents/img/screenshot.png)

- ![적은 노이즈 선형화](C:/Users/hyeji/Documents/img/screenshot.png)

- ![적은 노이즈 선형화](C:/Users/hyeji/Documents/img/screenshot.png)

##### 왜 Parameterization을 하는가?

- IF는 KF보다 numerically stabe하다.
- Prediction step은 KF가 더 빠르고 Correction step은 IF가 더 빠르다.
  - 상황에 따라 IF와 KF를 선택해서 사용할 수 있다.
  - 강의에서는 IF가 KF보다 조금 더 빠르다고 한다.
- 불확실성이 강한 상황에서 IF를 사용하면 sparse한 Information matrix를 얻을 수 있다.
  -> 즉, 계산이 효율적이다.



### Kalman Filter Algorithm to Information Filter Algorithm

- 기존의 Kalman filter algorithm에서 Parameterization을 하면 쉽게 구할 수 있다.

  -----------------------------

  1. $$\bar{\Omega_t}=(A_t\Omega_{t-1}^{-1}A_t^T+R_t)^{-1}$$
  2. $$\bar{\xi_t}=\bar{\Sigma_t}(A_t\Omega_{t-1}^{-1}\xi_{t-1}+B_tu_t)$$
  3. $$\Omega_t=C_t^TQ_t^{-1}C_t+\bar{\Omega_t}$$
  4. $$\xi_t=C_t^TQ_t^{-1}z_t+\bar{\xi_t}$$

  -------------------------------

- Prediction step : 1, 2

- Correction step : 3, 4

- __Complexity__

  |                | Kalman Filter  | Information Filter |
  | :------------: | :------------: | :----------------: |
  |   Prediction   |   $$O(n^2)$$   |   $$O(n^{2.4})$$   |
  |   Correction   | $$O(n^{2.4})$$ |     $$O(n^2)$$     |
  | Transformation | $$O(n^{2.4})$$ |   $$O(n^{2.4})$$   |

###### 내가 어떤 filter를 사용하느냐에 따라 matrix가 sparse할 수 있기 떄문에 cost가 매우 낮아진다.



### From EKF Algorithm to EIF Algorithm

|                      EKF                      |                             EIF                              |
| :-------------------------------------------: | :----------------------------------------------------------: |
| $$\bar{\Sigma_t}=G_t\Sigma_{t-1}G_t^T + R_t$$ |   $$\bar{\Omega_t}=(A_t\Omega_{t-1}^{-1}A_t^T+R_t)^{-1}$$    |
|       $$\bar{\mu_t}=g(u_t,\mu_{t-1})$$        | $$\bar{\xi_t}=\bar{\Omega_t}g(u_t,\Omega_{t-1}^{-1}\xi_{t-1})$$ |

- 이것 역시 EKF에 parameterization을 함으로써 쉽게 구할 수 있다.

  --------------

  1. $$\mu_{t-1}=\Omega_{t-1}^{-1}\xi_{t-1}$$

  2. $$\bar{\Omega_t}=(G_t\Omega_{t-1}^{-1}G_t^T+R_t)^{-1}$$

  3. $$\bar{\mu_t}=g(u_t,\mu_{t-1})$$

  4. $$\bar{\xi_t}=\bar{\Sigma_t}(A_t\Omega_{t-1}^{-1}\xi_{t-1}+B_tu_t)$$

  5. $$\Omega_t=C_t^TQ_t^{-1}C_t+\bar{\Omega_t}$$

  6. $$\xi_t=C_t^TQ_t^{-1}z_t+\bar{\xi_t}$$

  -------------------

  

  