---
layout: post
title: "Particle Filter"
date: "2019-11-06"
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
> 본 게시글은 Cyrill Stachniss의 [Robot mapping(WS 2013/14)](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 요약 및 정리한 글입니다. 
> 틀린 내용을 지적해주시면 감사드리겠습니다.

# Particle Filters and Monte Carlo Localization

- robot의 pose를 추정하는 알고리즘
- non-parametric 방식으로, 임의의 distribution을 갖는 motion model에도 적용할 수 있다.

----

## Idea

- 임의의 distribution을 sample로 표현하자.
  
  - 로봇의 pose가 gaussian distribution을 갖는다고 하면?
  - 로봇 pose의 평균 주변으로 가우시안의 형태로 sample을 generation한다.
  
- 문제 : 임의의 distribution에 따른 sample을 어떻게 generation할 것인가?

  - gaussian 또는 uniform distibution과 같이 쉽게 생성할 수 없다.
  - 임의의 distribution을 그래프의 형태로는 표현할 수 있지만 생성할 수는 없다.

- __Idea__ :

  1. sample을 잘 아는 distribution(uniform 또는 gaussian)을 통해 생성한다.

  ![image-20191104224333383](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-04-Grid-Maps - 복사본.assets\image-20191104224333383.png)

  2. 생성된 sample들에 weight를 준다.

     - sample x weight이 distribution과 같은 그래프를 갖도록 weight를 준다.

     <img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-04-Grid-Maps - 복사본.assets\image-20191104224430518.png" alt="image-20191104224430518" style="zoom:67%;" />

  3. 해당 sample의 weight만큼 sample을 생성하고 normalize한다.
     - 즉, 5의 weight를 갖는 sample은 5개의 sample을 생성,  
       1의 weight를 갖는 sample은 1개의 sample을 생성(즉, 유지)

  ![image-20191104224508253](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-04-Grid-Maps - 복사본.assets\image-20191104224508253.png)

- 정의
  - proposal distribution $g$ : 우리가 생성할 수 있는, 기초가 되는 distribution
  - target distribution $f$ : 우리가 만들고자하는 distribution
  - weight $w=f/g$ : proposal dist.를 target dist.로 만들기 위한 각 sample의 weight
  - __Importance Sampling__ : 이러한 weight를 갖는 sample을 뽑는 것.
    - Resampling : Proposal distribution의 particle들을 target distribution으로 다시 sampling하므로 이것을 resampling이라 한다.

## Particle Filter Algorithm

1. proposal distribution을 이용하여 particle들을 sampling 한다.

   $$ x_t^{[j]} - \pi(x_t\|...) $$

2. Importance weight를 계산한다.
   $$w_t^{[j]}=\frac{target(x_t^{[j]}}{proposal(x_t^{[j]})}$$

3. 각 particle들의 weight를 기반으로 Resampling한다.

![image-20191106214600038](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-04-Grid-Maps - 복사본.assets\image-20191106214600038.png)

- 2 ~ 6 )
  - proposal distribution $\pi(x_t)$ 를 기반으로 particle을 sampling한다.
  - weight를 계산한다.
  - proposal particle set $\bar{\chi}_t$ 에 sampling한 particle를 추가한다.
- 7 ~ 10 )
  - 기존 J개의 particle을 weight에 맞게 다시 sampling한다.
  - target particle set  $\chi_t$ 에 resampling한 particle을 추가한다.

## Monte Carlo Localization

- Particle filter를 Localization에 사용하는 실질적인 알고리즘.

- 각 particle들은 robot의 pose 정보를 갖는다.

- Proposal distribution은 motion model이다.

  $$x_t^{[j]} - p(x_t\|x_{t-1}u_t)$$

- Weight는 observation model이다.

  $$ w_t^{[j]}=\eta p(z_t\|x_t,m)$$

![image-20191106215448176](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-04-Grid-Maps - 복사본.assets\image-20191106215448176.png)

- 3 )
  - Motion model로 particle을 sampling한다.
    - particle들에 $x_{t+1}^{[j]} = Ax_t^{[j]} + Bu_t + noise$ 를 적용하는 것과 같다.
    - noise는 우리가 생성할 수 있는 gaussian과 같은 distribution을 갖는다.
- 4 )
  - Sensor model로 Importance weight를 한다.
  - 여기서 생길 수 있는 의문
  - target distribution이 non-parametic한 motion noise를 표현해야하는데 그 어디에도 non-parametric motion noise에 대한 이야기가 없다?
    - $$w_t^{[j]}=\frac{target(x_t^{[j]}}{proposal(x_t^{[j]})}$$ 에서 target은 $p(x_t\|u_{t}, z_t)$이다. 
    - $w_t^{[j]}=\frac{target(x_t^{[j]}}{proposal(x_t^{[j]})}=\frac{p(x_t\|x_{t-1},u_{t}, z_t)}{p(x_t\|x_{t-1}u_t)}$
    - Factorization을 적절하게 하면 sensor model만 남는다!

## Resampling

- Particle의 Weight에 기반하여 새롭게 sample을 뽑는 것을 resampling이라 했다.
- 왜 하는가?
  - 한정된 개수의 particle을 갖기 때문에 쓸데 없는 정보를 갖는 particle들의 수를 줄여야한다.
  - 낮은 가능성의 pose를 갖는 particle들을 높은 가능성의 pose를 갖는 particle로 대체한다.
  - 이를 통해 높은 가능성의 pose를 갖는 particle들만 남게된다.
- 그럼 어떻게 Resampling을 하는가?

### Low Variance Resampling

- O(J)
- 추후 내용 추가.



## Summary

- Particle filter는 non-parametric, recursive Bayes filter이다.
- Particle filter Localization은 arbitrary motion noise를 표현할 수 있기 때문에 더욱 성능이 좋다.
- Multi mode를 표현할 수 있기 때문에 더욱 성능이 좋다.
  - EKF는 linearization으로 인한 error, single mode representation

