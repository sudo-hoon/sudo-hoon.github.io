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

> 본 게시글은 Cyrill Stachniss의 [Robot mapping(WS 2013/14)](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 요약 및 정리한 글입니다. 
> 틀린 내용을 지적해주시면 감사드리겠습니다.

# Fast SLAM - Grid-Based SLAM with Particle Filters
- Particle Filter를 SLAM 문제에 어떻게 적용할 것인가?
  - Particle Filter로 localization을 한다.
  - Grid map으로 mapping하자.

----

## Rao-Blackwellization for SLAM

- $p(x_{0:t}, m_{1:M} \| z_{1:t}, u_{1:t})=p(x_{0:t}\|z_{1:t},u_{1:t})p(m_{1:M}\|x_{0:t},z_{1:t}) $ 

- Feature-based 역시 각 particle당 각 map을 갖고 있다.

- 각 particle은 robot의 trajectory를 갖고 있다.
- 각 particle은 known pose mapping을 한다.
- Grid-based Fast SLAM은 known pose 가정이 잘 맞지 않는다.
  - feature base는 update할 때 서로 correlation으로 map이 조정되지만 grid는 cell간의 독립성이 가정돼있어서 그게 안된다.
  - pose의 정확도를 높여야한다!

### Scan matching을 통한 pose correction

- $x_t^*=argmax_{x_t}{p(z_t\|x_t,m_{t-1})p(x_t\|u_t,x_{t-1}^*)}$
  - robot motion이 주어졌을 때, 가장 적절한 robot pose    
                                              \+    
    센서 값이 지도에 가장 잘 matching되게 하는 robot pose 
  - Scan matching을 통해 최적화하자!

__Scan matching__

- scan matching은 __locally consistent pose correction__을 한다.
  - 왜냐하면 내가 관측할 수 있는 영역 내에서만 scan matching할 수 있으므로.
- scan-matching을 통해 pose를 correction하고 그것을 FastSLAM의 input으로 사용하자.
  - 즉, object function에서 robot의 pose를 scan matching의 결과를 사용한다.  
    $$p(x^*_{0:t}, m_{1:M} \| z_{1:t}, u_{1:t})=p(x^*_{0:t}\|z_{1:t},u_{1:t})p(m_{1:M}\|x^*_{0:t},z_{1:t}) $$ 

__Graphical represent for mapping with improved odometry__

![image-20191116102546226](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-14-Grid-SLAM-FastSLAM.assets\image-20191116102546226.png)

- $u_1...u_k$를 motion command set으로 묶어서 한번에 움직인다.
- 움직인 후, scan matching을 통해 현재 최적의 pose state를 찾는다.
  - 초기의 pose와 현재 최적의 pose 사이의 움직임을 한번의 motion command, $u_1'$으로 표현한다.
- $u_1'$을 particle filter의 robot command로 대체하여 particle filter를 적용한다.
- 반복한다.

## Summary fast SLAM 1.0 with grid mapping

- scan matching을 통해서 높은 성능을 가질 수 있다.
- scan matching을 통해 높은 노이즈를 갖는 motion command를 high quality로 만들 수 있다.
  - 즉, proposal distribution의 quality를 높일 수 있다.
- 하지만 ad-hoc solution으로, 근본적인 해결책은 아니다.
  - proposal distribution의 quality를 높이기 위해서 중간에 이것 저것 끼워 넣은 느낌.



# fast SLAM 2.0

- 이전의 문제는?
  - grid mapping 알고리즘은 motion noise에 취약하다.
  - scan matching을 통해 high quality proposal distribution을 얻었지만 근본적인 해결책은 아니다.
- 목표
  - 더 정확한 Sampling
  - 더 정확한 mapping
  - 더 적은 파티클 사용

## Improved proposal distribution

$$x_t^{[k]} - p(x_t \| x^{[k]}_{1:t-1}, u_{1:t}, z_{1:t})$$

- 이전의 proposal distribution : $x_t^{[k]} - p(x_t \| x^{[k]}_{1:t-1}, u_{1:t})$ 에서 measurement가 추가되었다.
- measurement를 이용하여 더 정확한 proposal distribution을 생성하겠다!
- particle filter의 3 요소를 새롭게 정리해보자.
  - Proposal distribution sample generation
  - Importance weight calculation
  - Resampling

### Proposal distribution generation

- $x_t^{[k]} - p(x_t \| x^{[k]}_{1:t-1}, u_{1:t}, z_{1:t})= \frac{p(z_t\|x_T,m^{[i]})p(x_t\|x_{t-1}^{[i]},u_t)}{p(z_t\|x^{[i]}_{t-1},m^{[i]},u_t)}$ 
  - 분모 부분을 marginalizing form으로 바꾸면 아래와 같이 된다.

![image-20191116110314910](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-14-Grid-SLAM-FastSLAM.assets\image-20191116110314910.png)

- 위 수식에서 분모 부분을 보자. (위 그림에서 분포는 x에 대한 분포이다.)
  - local 부분은 sensor model로, x의 likelihood는 뾰족함과 동시에 여러 곳에 multimodal로 존재한다.
    - 센서로 측정했을 때 추측하는 robot의 위치는 다양할 수 있다.
    - 예를들어 같은 방이 여러개 있을 때, 센서로 추측하는 로봇의 위치는 어느 방일지 알 수 없다.
    - 따라서, sensor로 추측하는 robot의 위치는 지역적으로만 limit돼 있다.
  - global 부분은 motion model로, x의 distribution은 넓지만 single mode이다.
    - 로봇이 motion command로 주었을 때, 로봇의 위치는 순간이동될 수 없다.
    - 예를 들어서 5cm 움직이라는 command가 있을 때, 갑자기 robot의 위치가 5m로 이동할 확률은 거의 없다.
    - 따라서, motion command로 얻어지는 robot의 distribution은 globally limit돼 있다. 
- $tau$부분을 보자.
  - 앞서 말한 좁지만 multi mode인 locally limit과 넓지만 single mode인 globally limit의 곱이다.
  - 둘을 곱하면? ->  좁고 single mode인 distribution의 분포를 가질 것이다!
- 위 수식을 $tau$로 정리하면

![image-20191116110332684](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-14-Grid-SLAM-FastSLAM.assets\image-20191116110332684.png)

- proposal distribution을 이쁘게 정리했지만 이것을 어떻게 generation할 것인가?
  - 이전에는 proposal distribution을 gaussian으로 사용했기에 쉽게 생성할 수 있었지만 위 수식으로는 바로 생성할 수 없다.
- $\tau(x_t)$가 앞서 말한 듯이 좁고 single mode인 distribution을 가질 것이라고 예측 가능하다.
  - 그럼 gaussian approximation을 통해 생성하자!

![image-20191116113624431](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-14-Grid-SLAM-FastSLAM.assets\image-20191116113624431.png)

- scan matching으로 maximum point를 찾고 그 주변으로 sample들을 생성한다.
  - sample들($x_j$)에 $\tau(x_j)$ 값을 이용하여 평균, 분산을 계산하여 approximation을 한다.
  - 획득된 gaussian을 바탕으로 particle에 motion command를 내린다.

- __Proposal distribution sample generation 은 gaussian approximation으로 해결한다.__

> 이전에 EKF에서 robot의 pose를 multi modal로 표현 못해서 particle filter를 사용했는데, 또 gaussian approximation을 하면 로봇의 pose를 multi modal로 표현하지 못하는 것 아닌가?  
> 그렇지는 않다.  
> 왜냐하면 robot의 pose를 gaussian으로 approximation한 것이 아니라 proposal distirbution만 gaussian으로 approximation한 것이고 Importance sampling 과정을 지나면 multi modal이 표현된다.  
> 이전의 Particle filter도 실제로 proposal distribution이 gaussian이었다.

### Importance Weight

![image-20191116113117099](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-14-Grid-SLAM-FastSLAM.assets\image-20191116113117099.png)

![image-20191116113152326](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-14-Grid-SLAM-FastSLAM.assets\image-20191116113152326.png)

- 수식 전개는 위 과정을 따라가면 쉽게 가능하다.
  - 마지막 $\Sigma$ 부분은 Proposal distribution을 gaussian approximation하는 과정에서 획득된다.
- particle의 weight는 이전의 weight 에 $\Sigma \tau(x_j)$의 곱이다.
  - $\Sigma \tau(x_j)$가 모든 particle마다 다른 값을 갖나??
  - $\tau$는 particle 마다 다르다.

### Resampling

![image-20191116114728339](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-14-Grid-SLAM-FastSLAM.assets\image-20191116114728339.png)

- 이전 resampling의 문제

  - 각 particle들은 이전의 trajectory를 갖고 있다.
    - 즉, 자신의 조상 particle들을 갖고 있다.
  - 그런데, resampling을 하다보면 결국 같은 조상 particle로 부터 나온 particle만 남을 수 있다.
    - 중요한 정보를 갖는 particle을 잃을 수도 있다.(particle depletion)
  - 위 그림 처럼 가장 오른쪽 4개의 particle들은 가장 왼쪽 2번째 행의 particle을 같은 조상으로 갖는다.
  - 다양성을 잃을 수 있다.
    - 만약, 그 조상 particle이 문제가 있더라도 수정이 안된다.

- Resampling은 particle filter가 converge하기 위해서 꼭 필요하다.

  - 하지만 너무 자주하면 오히려 정보를 잃는다.
  - 따라서 꼭 필요할때만 하자!

- __Selective Sampling__

  - particle들 간의 weight가 너무 극명하게 차이날 때만 resampling을 하자.
    - particle들 간의 weight가 비슷할 때는 resampling을 통해서 오히려 가능성이 있는 particle을 잃는다.

- __Effective Particle__

  - $n_{eff}=\frac{1}{\Sigma_i(w_t^{[i]})^2}$
  - Effective particle : the inverse variance of the normalized particle weights
    - particle들 간의 weight 차이가 없다 -> $n_{eff} = \# of particles$
    - particle들 간의 weight 차이가 커진다 -> $n_{eff}$가 작아진다.

  - $n_{eff}$가 전체 particle 개수의 1/2일 때, resampling을 하는 방식으로 selective하게 resampling을 하자.

![image-20191116115547700](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-14-Grid-SLAM-FastSLAM.assets\image-20191116115547700.png)

- 그림으로 이해해보자.
  - 왼쪽 아래 그래프를 보면 시간이 갈 수록(움직일 수록) $n_{eff}$ 가 작아진다.
  - 그러다가, loop closing을 하는 순간,  $n_{eff}$ 가 매우 작아진다.
    - loop closing을 할때, 이전의 trajectory와 closing이 안되는 particle들은 weight가 매우 작아지기 때문에.
  - 이때, threshold 아래로 $n_{eff}$가 작아졌을 때, resampling을 진행한다.
  - known areas를 지날때는 weight의 변화가 크게 없다.
  - 약 3500 second 에서도 loop closing이 발생했을 때, 똑같은 상황이 발생한다.



### Gaussian approximation에 대한 고민

- proposal distribution을 개선시키려고 observation도 model에 추가했다.
  - 이전의 proposal distribution은 큰 motion noise의 영향이 너무 컸다.
    - 새로운 proposal dis.는 거기에 sensor model이 곱해짐으로써 noise의 분산이 굉장히 작아졌다.
  - 게다가, observation으로 인해서 multi modal의 가능성이 생김으로써 표현력이 좋아진다.
  - 그런데, approximation으로 인해서 표현력이 다시 감소하는 상황이 발생한다.
    - loop closing을 할 때, proposal distribution이 multi-modal일 가능성도 충분히 있다.
    - 실제 distribution이 multi modal일 때, approximation은 실패하고 diverge할 수 있다.
- __해결책__

![image-20191116120730675](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-14-Grid-SLAM-FastSLAM.assets\image-20191116120730675.png)

- sample들을 odometry로만 움직여서 proposal distribution을 gaussian approx.하지말고 노이즈를 주고 odometry를 입력하자.
  - 그렇다면, 각 particle마다 다른 mode의 gaussian distribution을 가지며 multi modal이 표현된다.

## Summary

- FastSLAM은 grid map에서도 적용될 수 있다.
- feature-base와 달리 motion noise의 영향을 크게 받는다.
  - 따라서, proposal distribution을 개선시키는 것이 중요하다.
- selective resampling으로 particle depletion을 막을 수 있다.
- 위 과정들을 통해 적은 수의 particle로도 좋은 성능을 낼 수 있다.