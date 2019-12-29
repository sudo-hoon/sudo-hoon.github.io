---
layout: post
title: "Fast SLAM - Grid base"
date: "2019-11-14"

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

# Least Squares

- Graph based approach
- least square : error minimization technique

## Least square?

- unknowns 보다 equation이 많을 때 쓰는 방법.(overdetermined system)
- squared error sum을 최소화한다.

## Problem

- observation ftn $\{f_i(x)\}_{i=1:n}$
  - $x$ : state vector
  - $z_i$ : state x의 measurement
  - $\hat{z_i}=f_i(x)$ : x를 predicted measurement $\hat{z_i}$에 mapping하는 ftn
- 문제 정의
  - 노이즈를 갖는 측정이 n개, 즉,  $z_{1:n}$가 주어졌을 때, state x를 estimation하는 것

- graphical definition

![image-20191109174031742](Least Squares.assets\image-20191109174031742.png)

- $\hat{z}$는 predicted measurement, 즉, 내가 노이즈를 갖고 움직인 predicted position x에 대해서 이전에 알고 있던 landmark 또는 grid와의 거리를 이용해서 얻을 수 있는 값들
- $z$는 real measurement, 즉, 직접 센서를 이용해서 측정한 거리이며 이전에 알고있던 landmark 도는 grid와의 거리를 이용해서 x를 얻을 수 있다.
- 이 둘 사이의 error를 최소화하는 x를 찾는 것이 목표.

## Error function

- $e_i(x)=z_i-f_i(x)$
  - assume : zero mean, normal distribution
- squared error : $\bold{e_i(x)} = e_i(x)^T\Omega_ie_i(x)$
  - $\Omega_i$ : information matrix, gaussian의 error는 information matrix이다.

![image-20191109175201863](Least Squares.assets\image-20191109175201863.png)

- goal : $x^* = argmin_x\bold{e_i(x)} = argmin_x\Sigma_ie_i^T(x)\Omega_ie_i(x)$
  - 어떻게 해결할래?
  - Numerical approach
  - assume : 
    - good initial guess를 한다.
      - global minima 근처에서 initial guess를 한다.
      - local minima에 빠지지 않는다.

## optimization

### Iterative Local  Linearization(newton approach)

1. error term을 current solution 또는 initial guess위치에서 linearization 한다.
2. 1차 미분을 구한다.
3. set to zero 후 linear system을 푼다.
4. 새로운 state, 즉, solution(hopefully closer to minima)
5. iterative

- how linearization?

  - Taylor expansion : $e_i(x+\Delta x)-= e_i + J_i(x)\Delta x$

  - $\bold{e_i(x)} -= (e_i + J_i(x)\Delta x)^T \Omega_i (e_i + J_i(x)\Delta x)$  
    $= e_i(x)\Omega_ie_i(x) + \Delta x^T J_i^T \Omega_i e_i(x)$  
    $+ e_i^T(x)\Omega_i J_i\Delta x + \Delta x^T J_i^T \Omega_i J_i \Delta x$  
    $= e_i^T \Omega_i e_i + \Delta x^T J_i^T \Omega_i J_i \Delta + 2e_i^T(x) \Omega_i J_i \Delta x$
  - let
    - $c_i=e_i^T \Omega_i e_i$
    - $H_i = J_i^T \Omega_i J_i$
    - $b_i = e_i(x)^T \Omega_i J_i$
  - $\bold{e_i(x)}=c_i+2b_i^T\Delta x + \Delta x ^T H_i \Delta x$
    - 즉, $\bold{e_i(x)}$는 $\Delta x$에 대해 quadratic 이다.
  - $F(x) -= \Sigma_i e_i = \Sigma_i [c_i+2b_i^T\Delta x + \Delta x ^T H_i \Delta x]$  
    $= \Sigma_ic_i+2(\Sigma_i b_i^T)\Delta x +\Delta x ^T (\Sigma_i H_i) \Delta x$  
    $= c+2b^T \Delta x + \Delta x^T H \Delta x$
  - $\frac{\partial F}{\partial x} = 2b + 2H\Delta x $

- set to 0

  - $H\Delta x = -b$

- __solution__

  - $\Delta x^* = -H^{-1}b$
  - 즉, linearization한 error function에서 error가 최소화하는 x의 값은 linearization에 사용했던 x에 다가 $\Delta x^* = -H^{-1}b $를 더한 값이다!

- Algorithm

  1. Jacobian J를 계산한다.
2. $H = \Sigma_i J_i^T \Omega_i J_i$ ,  $b = \Sigma_i e_i(x)^T \Omega_i J_i$
  3. $\Delta x^* = -H^{-1}b$를 계산한다.
  4. $x <- x + \Delta x^*$를 update한다.

## Example : Odometry Calibration

- Goal : Systematic odometry error를 없애고 싶다.

  - bias, 바퀴 크기 차이로 발생하는 error들
  - 노이즈와 다른, deterministric하게 발생하는 error를 제거한다.

- assume : Ground truth odometry $u_t^*$ 를 얻을 수 있다.

  - 
  - $u_t^*$ : training data set

- 문제 정의 : 

  - $u_i^`=f_i(\bold{x})=\begin{pmatrix} x_{11}& x_{12} & x_{13} \\ x_{21}& x_{22} & x_{23} \\ x_{31}& x_{32} & x_{33} \end{pmatrix}u_i $

  - $\bold{x}$ : bias error
  - $f_i(x)$ : bias error인 x를 넣었을 때, corrected odometry를 return하는 function
  - $u_i$ : biased-odometry
  - $u_i^`$ : unbaised-odometry
  - state vector
    - $\bold{x} = (  x_{11}\ x_{12}\  x_{13}\ x_{21}\ x_{22}\  x_{23}\  x_{31}\ x_{32}\  x_{33})^T$
  - error vector
    - $\bold{e_i} = u_i^*-\begin{pmatrix} x_{11}& x_{12} & x_{13} \\ x_{21}& x_{22} & x_{23} \\ x_{31}& x_{32} & x_{33} \end{pmatrix}u_i$
  - jacobian of error function :
    - $J_i=\frac{\partial e_i(x)}{\partial x}=-\begin{pmatrix} u_{i,x}&u_{i,y} & u_{i,\theta} & & & & & & \\ & & & u_{i,x}&u_{i,y} & u_{i,\theta} & & & \\ & & & & & & u_{i,x}&u_{i,y} & u_{i,\theta} \end{pmatrix}$
      - $\bold{u_i} = (u_{i,x}, u_{i,y},u_{i,\theta})$
      - __Jacobian이 x에 독립이다!__
        - __즉, error를 최소화 하는 문제는 linear problem이다.__
        - __$\bold{e_i}$가 linear이다!. (not approximation)__
          - __iteration을 할 필요가 없다. direct solution을 얻을 수 있다!__

- Q : 

  - odometry가 perfect하다면?
    - $\bold{x} $는 단위 행렬의 형태를 갖는다.
  - calibration problem을 풀려면 최소한 얼마의 measurement가 필요한가?
    - 3개. unknown odometry가 x, y, $\theta$ 로 3개이므로.
    - 많을수록 좋다.
  - $H$ 는 왜 symmectric 인가?
    - $H = \Sigma_i J_i^T \Omega_i J_i$
      - Jacobian도 symmetric, information matrix도 symmetric이므로
  - measurement function의 구조가 $H$의 구조에 어떻게 영향을 주는가?
    - error 가 없는 원소에 해당되는 jacobian은 0이다.
    - error가 없을수록 sparse해진다.

### Linear system을 어떻게 효율적으로 계산할 것인가?

- Linear system : $H\Delta x = -b$
  - 이것을 풀려면 Matrix inversion이 필요하다!
    - cubic complexity
  - 효율적인 방법
    1. Cholesky factorization
    2. QR decomposition
    3. Iterative methods
       - conjugate gradients

- Cholesky Decomposition
  - $A$ : Symmetric and Positive definite
  - problem : $Ax = b$
  - cholesky decomposition
    - $A = LL^T$를 얻을 수 있다.
      - $L$ : lower triangular matrix
  - $Ax=LL^Tx=Ly=b$ 을 보면
    - $Ly=b$
    - $L^Tx=y$
    - 위 두 가지 linear system을 풀어서 x를 구할 수 있다.
      - $L$은 triangular matrix이므로 쉽게 풀 수 있다.

## 이 방법을 state estimation에 어떻게 적용할 것인가?

- __linear system이 gaussian이라면 linear square는 Maximum likelihood 문제가 된다.__



### General state estimation

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-Least-square.assets\image-20191117114117191.png" alt="image-20191117114117191" style="zoom:50%;" />

- markov assumption, independence를 적용하면 위와 같이 정리할 수 있다.
- log를 취하면

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-Least-square.assets\image-20191117114319473.png" alt="image-20191117114319473" style="zoom:50%;" />

- 모든 term이 gaussian이라고 가정한다면?

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-Least-square.assets\image-20191117114434427.png" alt="image-20191117114434427" style="zoom:50%;" />

- log gaussian은 위와 같이 표현되며 이전까지 전개했던 $e_i(x)$와 같은 error function의 형태를 갖는다.
- 이것을 각 term에 적용하면

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-Least-square.assets\image-20191117114634960.png" alt="image-20191117114634960" style="zoom:50%;" />

- 정리하면

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-17-Least-square.assets\image-20191117114732760.png" alt="image-20191117114732760" style="zoom:50%;" />

- 결국,  inititial state error, odometry error, observation error를 최소화하는 문제가 된다.
- __gauss-newton approach를 통해 위의 문제를 풀 수 있다.__

> 그런데, particle filter의 경우를 보면 robot의 state가 실제로 multi-modal이여야하는 상황인데 gaussian이면 표현할 수가 없어서 발산하는 경우가 있다. 이렇게 가정하고 문제를 푸는 것이 정확할 수 있는가?  
> 한 가지 해결책을 고려해보면 particle마다 다른 형태의 gaussian distribution을 가짐으로써 여러 파티클들이 여러 mode를 표현하는 방식으로 할 수는 있을 것 같다.



## summary

- squared error function을 minimize하는 방법.
- gauss-newton은 non-linear problems을 iterative하게 풀 수 있다.
- linearization을 통해 linear problem으로 바꿀 수 있다.
- minimize squared error function = maximize the log likelihood of indep. gaussian