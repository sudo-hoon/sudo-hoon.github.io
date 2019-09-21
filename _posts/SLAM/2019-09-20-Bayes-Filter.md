---
layout: post
title: "Bayes Filter"
date: "2019-09-20"
# slug: "example_content"
imagefeature: foo.png
description: "This post is based on SLAM lecture"
category: 
  - SLAM
# tags will also be used as html meta keywords.
tags:
  - SLAM
  - Robot mapping
comments: true
use_math: true
show_meta: true
---

> 본 게시글은 Cyrill Stachniss의 [Robot mapping(WS 2013/14)](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 요약 및 정리한 글입니다. 
> 틀린 내용을 지적해주시면 감사드리겠습니다.


# Bayes filter


### state estimation


* State estimation 문제에서 풀고자 하는 문제의 정의는 다음과 같다.
* Goal : $p(x\|z,u)$
* 이때, bayes theorem[^1] 에 기반하여 위의 state estimation을  
  recursive하게 정리할 수 있고  정리된 수식을 recursive bayes filter라고 부른다.
* robot mapping 에서 기본적인 문제 정의는 다음 수식과 같다.
  
  * $bel(x_t) = p(x_t\|z_{1:t}, u_{1:t})$  
  , $x_t$ : t번 째 state, $z_t$ : t번째 관측 값, $u_t$ : t번째 command
* 위 수식을 Marcov assumption과 Law of total probabilty를 이용하여 정리하면 아래와 같다.
* $bel(x_t) = \eta p(z_t\|x_t)\int{p(x_t\|x_{t-1},u_t)bel(x_{t-1})dx_{t-1}}  
     = \eta p(z_t\|x_t) \bar{bel}(x_t)$    
위 수식은 아래와 같은 두 가지 수학적 의미로 나눌 수 있다.


### Prediction step : $\bar{bel}(x_t) = \int{p(x_t\|x_{t-1},u_t)bel(x_{t-1})dx_{t-1}}$


* 바로 직전의 state와 현재의 command가 주어졌을 때, 추정되는 현재의 state  
-> $p(x_t\|x_{t-1},u_t)$ : motion model


### Correction step : $bel(x_t) = \eta p(z_t\|x_t) \bar{bel}(x_t)$


* 현재 추정한 state를 관측 값에 기반하여 수정한 최종적인 예측되는 현재의 state  
-> $p(z_t\|x_t)$ : observation model

----
> $p(z_t\|x_t)$ 는 likelihood function으로 posterior를 기반으로 prior를 추정하는 수학적 form을 가진다. 하지만 우리는 관측 값(prior)는 센서를 통해 얻지만 state(posterior)는 추정 값이다.  이 수식이 갖는 의미는 X_t를 x_t로 예측했을 때, 센서를 통해 얻은 z_t가 나올 가능성이 있는가를 본다. 즉, 관측 값이 $^{bel}$ 를 통해 예측학 $x_t$가 가능성이 적다면 적은 weight를 발생시킬 것이고 가능성이 크다면 큰 weight를 발생시켜 예측 값을 correction한다.

----
* 문제는 정의되었다. 하지만 이 문제를 어떻게 해결할 것인가?     
-> 어떻게 model을 정의할 것인가?


## Motion Model : $p(x_t\|x_{t-1},u_t)$


* 대표적인 motion model은 두 가지 있다.
	1. Odometry-based : wheel encoders를 통해 motion command를 내린다.  
      -> 지면의 불 균일성, 바퀴간의 기압차 등으로 노이즈 발생

	2. Velocity-based : robot의 속도를 통해 motion command를 내린다.   
      -> 속도 센서의 오차 등으로 노이즈 발생   
      -> 일반적으로 Odometry-based motion model이 정확하다.


### Odometry model


* Robot이 $(\bar x, \bar y, \bar\theta)$ 에서 $(\bar x', \bar y', \bar \theta')$로 이동할 때,  
  odometry 정보는 $u = (\delta_{rot1}, \delta_{trans}, \delta_{rot2})$ 로 주어진다.  
  회전, 이동, 회전의 형태로 sequential한 motion command가 정의된다.  


### Velocity model


* Robot이 $(\bar x, \bar y, \bar \theta)$ 에서 $(\bar x', \bar y', \bar \theta')$로 이동할 때,   
  velocity 정보는 $u = (v, w)$ 로 주어진다.   
  회전과 이동이 동시에 motion command로 정의된다.   
  	-> 이러한 정의는 최종적인 robot의 orientation을 제한시킨다.  
  	-> 회전 term에 추가적인 variable을 둠으로써 orientation 회전을 표현한다.  


## Observation Model : $p(z_t\|x_t)$ 


* 한번의 sensor scan(ex. $\theta_1 ~ \theta_2$까지의 거리 측정)은 K번의 측정을 한다.   
  $z_t = {z_T ^1, ...., z_t ^k}$   
  만약에 각 측정이 independent 하다면   
  $p(z_t\|x_t,m) = \prod_{i=1} p(z_t ^i \| x_t, m)$로 표현할 수 있다.   
  즉, 단일 측정 결과들의 곱의 형태로 표현할 수 있다.   
  이러한 단일 측정 결과들을 어떻게 표현할 것인가?   

- 1. beam-endpoint model __-----------? __  
  2. Ray-cast Model
     4개의 정보를 합쳐서 측정 결과를 모델링 한다
     - 1. dynamic obstacle   
       2. obastacle + measurement noise
       3. maximum range
       4. uniform random noise


### Range-bearing[^2] sensor를 이용한 측정 모델


1.  range-finder를 이용한 거리 $z_t ^i = (r_t ^i, \phi _t ^i )^T$
2.  robot의 pose : $(x,y,\theta)^T$
3.  j 번째 feature의 위치 : location $(m_{j,x}, m_{j,y})^T$ 

- Sensor model : $\begin{pmatrix} r_t^i \\ \phi_t^i \end{pmatrix} = \begin{pmatrix} \sqrt{(m_{j,x}-x)^2 + (m_{j,y}-y)^2} \\ atan2(m_{j,y}-y, m_{j,x}-x) -\theta \end{pmatrix} + Q_t$
- $Q_t$는 sensor noise, 미리 modeling 된 노이즈   

----
> [^1]: $P(A\|B) = \frac{P(B\|A)P(A)}{P(B)} = \frac {P(A,B)}{P(B)}$,  https://darkpgmr.tistory.com/119 참고  
> [^2]: horisontal angle, https://en.wikipedia.org/wiki/Bearing\_(navigation) 참고

