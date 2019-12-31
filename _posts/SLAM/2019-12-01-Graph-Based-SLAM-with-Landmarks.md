---
layout: post
title: "Graph-Based SLAM with Landmarks"
date: "2019-12-01"

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

# Graph-Based SLAM with Landmarks

- 

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-12-01-.assets\image-20191201134801311.png" alt="image-20191201134801311" style="zoom:67%;" />

- node : robot pose와 landmark의 location
- Edge : landmark에 대한 관측과 Odometry measurements
  - 이전의 virtual observation은 더 이상 사용하지 않는다.
  - 이때, 이 edge에 포함된 error를 최소화하는 것이 목적!
- 같은 feature를 바라보는 node간의 relation을 이용하여 constraint를 만들 수 있다.
  - 중간에 feature를 바라보는 두 node들은 feature를 marginalizing함으로써 서로 간의 direct constraint를 얻을 수 있다.
  - 이때, 두 노드 사이의 상대적인 거리, 각도 정보를 얻을 수 있다.

- 



## Landmarks Observation

- Expected Observation
  - $\hat{z_{ij}}(x_i,x_j)=R_i^T(x_j-t_i)$
  - $e_{ij}(x_i,x_j)=\hat{z_{ij}}-z_{ij}= R_i^T(x_j-t_i)-z_{ij}$
- Only Bearing Observation
  - $\hat{z_{ij}}(x_i,x_j)=atan\frac{(x_j-t_i).y}{(x_j-t_i).x}-\theta_i$
  - $e_{ij}(x_i,x_j)=atan\frac{(x_j-t_i).y}{(x_j-t_i).x}-\theta_i-z_j$

## The Rank of the Matrix H

- $H_{ij}=J_{ij}^T\Omega J_{ij}$
  - $J_{ij}$ :  2x3 matrix​
  - 따라서, $H_{ij}$는 최대 2의 rank를 갖는다.
  - Bearing only case에는 $J_{ij}$는 1x3 matrix
  - 따라서 bearing only에서 $H_{ij}$는 최대 1의 rank를 갖는다.



# Where is the robot?

-  로봇이 landmark를 관측했을 때, 로봇은 그 landmark를 기준으로 circle 위에 존재한다.
  - landmark와의 각도를 알 수 없기 때문에.
  - 1-dimensional solution space를 갖는다.
    - $H_{ij}$가 rank가 2이기 때문에 하나의 freedom을 갖게 된다.
    - 더 많은 관측이 필요하다.
- 관측이 한번 더 생기면?
  - circle 사이의 교점을 통해서 unique solution을 찾을 수 있다.
- 만약 Bearing only라면?
  - 첫 관측을 통한 로봇의 위치는 x,y space 어디든 존재할 수 있다.
    - 모든 가능한 로봇의 위치에서의 heading은 알 수 있다.
    - 즉, 2-dimensional solution space를 갖는다.



## Rank

- landmark-based SLAM에서 system은 under-determined
- H의 rank는 constraint의 rank의 sum과 같거나 적다.
  - 이때 constraint는 landmark를 향한 constraint를 의미한다.

- unique solution을 찾기 위해서는 system이 full rank여야한다.
  - 얼마나 많은 2D landmark observation이 필요한가?
    - 2개 이상
  - 얼마나 많은 bearing only observation이 필요한가?
    - 3개 이상



## Under-Determined Systems

- full rank system이 보장되지 않는다.
  - landmark가 2번 이상 관측되리라는 보장이 없다.
  - robot이 odometry를 가지지 않을 수도 있다.(움직이지 않을 수 있다.)
- damping factor를 넣어서 강제로 full rank로 만들자.
- $(H+\lambda I)\Delta x=-b$
  - $\lambda$의 의미 : 
    - system을 positive definite로 만든다.
  - gauss newton의 weighted sum이며 이것이 steepest descent를 만든다.
    - steepest gradient 와 gauss newton approach 사이의 weighted sum??
  -  $\lambda$ : 처음에는 작게 잡고 linear system solve를 할때 에러가 이전보다 증가하면 $\lambda$를 증가시킨다.
    - 아니라면 $\lambda$를 점점 작게 줄인다.



## Bundle Adjustment

- 다른 위치에서 관측한 2D image로 3D reconstruction을 하는 것.
  - landmark를 기준으로 camera의 위치를 파악할 떄 사용한다.
- reprojection error를 최소화한다.
  - 내가 카메라에 관측될 것이라 기대하는 feature의 위치와 실제 feature의 위치 사이의 에러