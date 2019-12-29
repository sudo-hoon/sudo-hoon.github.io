---
layout: post
title: "Graph SLAM based on least square"
date: "2019-11-17"

# slug: "example_content"
description: "This post is based on SLAM lecture"
category: 
  - SLAM
# tags will also be used as html meta keywords.
tags:
  - SLAM
  - Robot mapping
comments: true
use_math: true
---

# Least Square Approach to SLAM

- 새로운 SLAM framework인 Graph-based SLAM을 다룬다.
- SLAM problem을 least sqaure로 해결한다.

### Least Square

- overdetermined system에 적용할 수 있는 방법.
- sqaured error sum을 minimize한다.
- a large set of problems에 대한 기본적인 접근법.

## Graph-based SLAM

![image-20191117145854108](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117145854108.png)

- robot이 움직이는 동안 로봇의 pose는 constraint로 연결된다.
  - constraint는 불확실성을 갖는다.
  - constraint는 robot pose간의 relative configuration을 제한한다.
    - 즉, 이 constraint에 맞춰서 robot pose가 제한된다.
    - constraint의 noise가 없다면 robot의 pose는 정확할 것이다.

![image-20191117150126269](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117150126269.png)

- 로봇이 이전 pose와 마주칠때, 이전의 pose와도 constraint가 생성된다.
  - 이전 pose와의 상대적 위치, 즉, loop closing에 해당된다.

### Idea of Graph-based SLAM

- Robot의 움직임을 graph를 이용하여 표현한다.
- __node__ : mapping동안 로봇의 pose에 해당되는 부분
- __edge__ : node 사이의 spatial constraint
- __Goal__ : robot의 움직임에 따라 graph를 그리고, constraint의 error를 최소화하는 node들을 찾는 것.

> 근데 SLAM인데 왜 robot pose 얘기밖에 없는가? mapping은 안하나?  
> robot의 pose를 정확히 알면 mapping은 쉬운 일이다. 이 방법으로 정확히 localizaiton을 하고 이후에 mapping을 한다.

- constraint는 node간의 association으로 결정되는 것?
- graph에서 node는 robot position과 laser measurement에 해당된다.
  - node마다 map을 하나씩 갖고 있고 그 두 node의 local map이 매우 비슷하면 그 노드 간의 constraint를 생성할 수 있다.

![image-20191117152552120](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117152552120.png)

- 위 이미지에서 선으로 이어진 부분이 graph이다.
  - 오른쪽 아래는 node들이 정확히 이어져있다.
  - 위쪽은 두 연속된 node가 constraint로 이어져있는 모습을 볼 수 있다.
    - 앞서 말했듯이, 각 graph의 local map이 비슷하기 때문에 서로 constraint가 형성돼 있다.
    - grid map 형식으로 그려진 지도를 보면 같은 모양의 지도가 그려져있다.
- graph를 다 그리고 나면 node들을 모아서 most likely map을 그릴 수 있다.

![image-20191117152830969](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117152830969.png)

- constraint들을 이용하여 node들을 모으면 위의 지도와 같이 정확한 지도를 표현할 수 있다.
  - 정확히는, 정확한 robot trajectory를 얻을 수 있고, 해당되는 지도를 그릴 수 있다.

## Overall SLAM System

![image-20191117153214393](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117153214393.png)

### Front end :

- Raw data(odometry, lidar)를 이용하여 edge(constraint)를 얻는다.
- Sensor information에 많이 의존하는 part

### Back end :

- constraint를 이용하여 node의 위치를 찾는다.



## Graph

- __n 개의 node__ : $x=x_{1:n}$
- __${x_i}$__ : 2D or 3D transformation (시간 $t_i$에서의 robot의 pose)
- __edge__ : 두 가지 상황에서 발생할 수 있다.

1. Odometry
   - 시간에 따른 robot의 움직임은 node간의 edge를 생성

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117154424419.png" alt="image-20191117154424419" style="zoom:50%;" />

2. Measurement

   - 다른 시간에 같은 관측 형태를 가질 때, 그 두 시간에서의 node간에 edge를 생성
   - edge는 virtual measurement를 의미하며, 한 node가 다른 node를 가상으로 관측하는 것을 표현한다.
     - 아래 그림에서 $x_i$와 $x_j$는 conner에 대해 같은 관측을 갖는다.
     - $x_i$는 $x_j$를 가상 관측한다.

   <img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117161509840.png" alt="image-20191117161509840" style="zoom:50%;" />

> measurement edge는 관측을 통해 두 노드 사이가 어떠한 각도로 얼만큼 떨어져있는지에 대한 정보가 있을 것 같다.  
> Edge는 correleration의 의미를 갖는 것으로 보인다.  
> 보면 $x_i$는 map에 대한 관측 정보를 $x_j$와 edge를 통해 표현하고 $x_j$가 관측에 대한 기준이 된다. 이전의 sparse information filter와 비슷한 느낌!

### Transformation

- Homogeneous coordinate가 편리하다.
  - 모든 node는 global frame으로 위치가 표현된다.

- __Odometry-Based Edge__ : $(X_i^{-1}X_{i+1})$
  - 연속적인 시간을 갖는 노드 사이에서 발생한다.
  - i번째 node를 원점으로 변환하고($X_i^{-1}$) i+1번째 node의 위치로 transformation한다.
    - i번째 node에서 i+1번째 노드를 바라보는 frame
- __Observation-Based Edge__ : $(X_i^{-1}X_j)$
  - 개별적인 노드들 사이에서 발생한다.

 ### Edge Information Matrix

- observation은 noise에 의해 영향 받는다.
- 각 edge의 uncertainty는 information matrix로 표현할 수 있다.
  - $\Omega_{ij}$ : i node와 j node 사이의 edge의 불확실성
- $\Omega_{ij}$가 클수록 optimization에서 해당 edge가 더 중요해진다?
- Questions
  - scan matching을 했을 때와 단지 odometry만 입력했을 때, information matrix는 어떻게 다를 것인가?
    - scan-matching을 하면 더 큰 값을 가질 것이다.
      - scan-matching을 하면 robot pose가 더 정확해진다.
      - 이전의 pose만 정확하다면 현재의 pose는 매우 정확할 것이다.
  - 길고 특징이 없는 복도라면 information matrix가 어떻게 생겼을 것 같은가?
    - 복도 쪽으로는 information matrix의 크기가 작고 벽 쪽으로는 클 것 같다..?

> odometry는 관측의 영향 없이 edge를 생성한다.  
> scan-matching은 주변의 관측에 크게 영향을 받는 edge를 생성한다.  
> scan-matching은 큰 information matrix를 생성하지만 주변의 환경에 크게 영향을 받게 된다.

### Pose Graph

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117172010635.png" alt="image-20191117172010635" style="zoom:67%;" />

- 위의 그림을 보면
  - edge는 사실 이전의 pose에서 추정하는 다음 pose라고 생각하면 된다.
    - EKF와 같이 추정된 위치, $z_{ij}$와 불확실성 $\Omega_{ij}$를 갖는다.
    - z는 이전에 언급했던 virtual measurement이다.
  - error 는 추정된 다음 pose인 edge에서 실제 위치 사이의 vector를 의미한다.
    - 이 error가 최소화 된다면? edge로 추정한 위치와 실제 위치가 일치!
- __Goal__ : $\hat{x}=argmin_x\Sigma_{ij}e_{ij}^T(x_i,x_j)\Sigma_{ij}(x_i,x_j)e_{ij}(x_i,x_j)=argmin_x\Sigma_{k}e_{k}^T(x)\Sigma_{k}(x)e_{k}(x)$
  - 위의 error vector를 최소화하는 $x$를 찾는 것이 목표!
    - 즉, error를 최소화하는 node 쌍을 찾는 것이 목표!

### State vector

- $x^T = (x_1^T \ x_2^T \ ... x_n^T)$
  - 각 x는 robot pose를 의미, n은 총 node의 개수

### Error vecotr

- $e_{ij}(x_i,x_j)=t2v(Z_{ij}^-1(X_i^{-1}X_j))$
  - t2v : homogeneous coordinate to vector form
  - $Z_{ij} $ : measurement (i node에서 관측(추정)하는 j node)
  - $X_i^{-1}X_j$ : i번째 node의 관점에서 보는 j번째 노드의 위치
    - 역함수가 붙는 것이 벡터의 시작점, 일반 매트릭스가 벡터의 끝점으로 보면 쉽다.
    - 즉, 추정된 j번째 위치에서 실제 j번째 위치를 가르키는 벡터, 즉, error vector
- __whole error vecotr : $e_{ij}(x)=t2v(Z_{ij}^{-1}(X_i^{-1}X_j))$__
  - 전체 state vector에서 두 pose 사이의 error를 본 것.
  - 어차피 관련된 node는 $x_i, x_j$뿐이므로 같은 말임.

- if $Z_{ij}=(X_i^{-1}X_j)$ 라면?
  - 즉, 관측된 j번째 노드의 위치가 실제 j번째 노드의 위치와 같다면?
    - 당연히 error = 0

## MInimization Error : Gauss-Newton method

- method
  1. error function 정의 및 linearization
  2. jacobian 계산
  3. 0으로 두고 linear system 계산
  4. 수렴할때까지 반복!

1. error function 정의

   - $e_{ij}(x+\Delta x)=e_{ij}(x)+J_{ij}\Delta x,$ 

2. jacobian 계산

   - $J_{ij}=\frac{\partial e_{ij}(x)}{\partial x}$

     - error function은 $x_i, x_j$에 관한 함수이므로 나머지에 대한 미분은 모두 0이다.

   - $\frac{\partial e_{ij}(x)}{\partial x} = (0 \ ... \frac{\partial e_{ij}(x_i)}{\partial x_i} \ ... \frac{\partial e_{ij}(x_j)}{\partial x_j} ... 0)$

   - $J_{ij} = (0 \ ... \ A_{ij} \ ... \ B_{ij} \ ... \ 0)$

     <img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117175418552.png" alt="image-20191117175418552" style="zoom:50%;" />

   - 즉, 위와 같이 __Jacobian은 매우 sparse 한 matrix가 된다.__

     - 즉, linear system이 sparse하므로 solution을 구하는 것이 쉬워진다.

3. Linear system 계산

   - Linear system의 solution : $\Delta x^* = -H^{-1}b$ (@least sqaures)

   - $b = \Sigma_{ij} e_{ij}(x)^T \Omega_{ij} J_{ij}$
   - $H = \Sigma_{ij} J_{ij}^T \Omega_{ij} J_{ij}$
     - J가 sparse하기 때문에 H도 sparse하다.
   - <img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117180321760.png" alt="image-20191117180321760" style="zoom:80%;" />
   - 위 그림을 보자.
     - Jacobian J는 N x 2 ( N : node 개수, 2 : i번째와 j번째 열만 원소 존재)
     - 그런데, 또 역시 i번째, j번째 행에만 데이터가 존재하므로 2x2의 형태로 2 cell만 값이 존재한다. 
     - $\Omega_{ij}$는 2x2 matrix
     - $e_{ij}$는 2x1 matrix
       - 실제로는, robot pose 표현에 따라 2 x 3(x,y,$\theta$) 또는 2 x 6 이다.
   - <img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117211326286.png" alt="image-20191117211326286" style="zoom:80%;" />
   - 모든 edge에 대해 적용하면, $H$는 sparse matrix가 된다.
     - 따라서 linear system, $\Delta x^* = -H^{-1}b$ 은 효율적으로 풀 수 있다.
   - 수식적으로 정리하면
     - $b_{ij}^T=e_{ij}^T\Omega_{ij}J_{ij}=(0\ ...\ e_{ij}^T\Omega_{ij}A_{ij}...\ e_{ij}^T\Omega_{ij}B_{ij}...\ 0)$
     - $H_{ij}=J_{ij}^T\Omega_{ij}J_{ij}=\begin{pmatrix} A_{ij}^T\Omega_{ij}A_{ij} & A_{ij}^T\Omega_{ij}B_{ij} \\ B_{ij}^T\Omega_{ij}A_{ij} & B_{ij}^T\Omega_{ij}B_{ij} \end{pmatrix}$

- Sparse Summary
  - $edge_{ij}$는 $b_{ij}$의 i번째, j번째 원소와 $H_{ij}$의 ii, ij, ji, jj 원소에만 영향을 준다.
  - Resulting system은 sparse하다.
  - system은 각 edge의 contribution을 모두 더해서 계산할 수 있다.

### Algorithm

- 문제 정의 :

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117212921157.png" alt="image-20191117212921157" style="zoom:75%;" />

- __Algorithm__

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117213013379.png" alt="image-20191117213013379" style="zoom:67%;" />

- 알고리즘은 간단하게 3가지 과정으로 구성된다.
  1. graph $x$로 linear system 구성하기
  2. 구성된 linear system solve하기
  3. $x$를 $\Delta x$만큼 증가시키고 수렴될 때까지 반복..
- __BuildLinearSystem :__

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191117213050893.png" alt="image-20191117213050893" style="zoom:67%;" />

- __SolveSparse__ : 
  - Sparse Cholesky decomposition
  - Conjugate gradients
- Question
  - 그냥 linear system을 풀게되면 어떻게 되는가?
    - 모든 pose가 상대적인 위치로 표현되기 때문에 무수히 많은 solution이 나온다.
    - 즉, determinant가 0이 된다.
      - $dx_1$이 되는 constraint를 하나 추가하면 된다.
      - 즉, $H=H+\begin{pmatrix}1 & 0 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & 0\end{pmatrix}$
  - 이 해결책은 Prior, $p(x_0)$의 역할과 큰 관련이 있다.
- __Role of the Prior__
  - $e(x_0)=t2v(X_0)=0$이라면?
    -  즉, $x_0$의 위치가 fix되어있다면?
    - $X_0$ 는 identity matrix가 된다.
    - 즉, reference frame에서 원점이다. 
    - 이것으로 hessian을 구해보면 0th 원소만 값이 존재한다.
      - 위에서  $\begin{pmatrix}1 & 0 & 0 \\ 0 & 0 & 0 \\ 0 & 0 & 0\end{pmatrix}$ 처럼
- __Fixing a subset of varaibles__
  - 만약 한 로봇 포즈를 정확히 알고 있고 다른 포즈들을 optimization해야할 때는 어떻게 해야하는가?
    - 해당 포즈에 대응되는 원소의 row, column vector들을 제거한 후 optimization한다.
  - hessian은 linearization point에서의 information matrix에 해당된다.
    - 로봇 포즈가 fix 돼있다는 것은 conditioning을 의미한다.
    - 이때의 information matrix는 수학적으로 다음과 같이 표현된다.
      - 우리 시스템은 가우시안으로 가정하고 있다.
    - <img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-GraphSLAM-based-on-Least-square.assets\image-20191130103328287.png" alt="image-20191130103328287" style="zoom:67%;" />
  - Hessian을 inverse하면 linearization point에서의 Covariance matrix를 얻을 수 있다.
    - diagonal 성분에 해당되는 pose의 global uncertainty를 알 수 있다.
- __Relative Uncertainty__
  - i가 고정되어 있을 때, 다른 node들 과의 상대적인 불확실성을 알고 싶다면?
    - i 번째 raw와 column을 지우고(즉, i번째 노드를 고정) inverse한다!
    - 이때의 j,j 원소의 값은 i번째 노드가 고정되어 있을 때, j번째 노드의 불확실성이다.

### 

## Summary

- Gauss-newton error minimization을 통해 Graph based SLAM의 back end optimization을 할 수 있다.

- $H$ matrix는 sparse하다.

  - 효율적인 계산이 가능하다.

    