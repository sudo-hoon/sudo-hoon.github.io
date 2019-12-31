---
layout: post
title: "Fast SLAM - Feature-Based SLAM with Particle Filters"
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

# Fast SLAM - Feature-Based SLAM with Particle Filters
- Particle Filter를 SLAM 문제에 어떻게 적용할 것인가?
  - Particle Filter로 localization을 한다.
  - EKF로 Mapping을 한다.

----

### Particle filter

- high dimension으로 갈 수록 많은 particle이 필요하다.
  - 6 dimension 아래까지만 잘 동작한다.

- 3 step
  1. Sampling from proposal
     - $x_t^{[j]} - \pi(x_t\|...)$
     - 알고 있는 distribution으로 sample들을 생성한다.
       - sample들을 robot command로 이동시킨 후의 distribution을 의미한다.
  2. Importance Weights
     - $w_t^{[j]} = \frac{target(x_t^{[j]})}{proposal(x_t^{[j]})}$
     - 위에서 생성한 distribution과 지금의 distribution을 비교하여 weight를 매긴다.
     - 위의 과정에서 proposal distribution으로 target distribution을 표현할 수 있다.
  3. Resampling
     - $w_t^{[j]}$ 의 확률을 가지는 sample i를 생성하는 것을 J번 반복한다.
     - Weight를 sample의 수로 변환하는 과정!

### Particle Represent

- A set of weighted samples
  - $\chi = \{<x^{[i]},w^{[i]}>\}_{i=1, ... , N}$
  - 한 sample은 하나의 가능한 state에 대한 hypothesis이다.
- Feature-based SLAM :
  - $ x = (x_{1:t}, m_{1,x},m_{1,y},....,m_{M,x},m_{M,y})^T$
  - 위와 같은 state에 대해 particle filter를 적용하면?
    - 너무 높은 dimension으로 계산량이 너무 커진다.
    - 아직 이렇게 큰 dimension에 particle fitler를 적용하는 방법은 없다.
- state space의 다른 dimension 사이의 dependencies를 이용하면?
  - grid map에서는 mapping을 할 때, robot의 pose를 알고 있다는 가정을 했다.
  - 로봇의 pose를 알면 mapping은 쉽다.
    - 즉, robot의 pose와 landmark의 위치는 독립적이라는 가정이면 쉽다.
  - 그렇다면 particle filter가 오직 robot의 pose를 가르키는데 사용한다면?
  - particle 하나를 pose에 대한 given으로 두고 mapping을 하는 것이 핵심 아이디어.
    - 파티클 1이 맞는 trajectory다? 그럼 map은 이럴거야
    - 파티클2가 맞으면? 그럼 map은 이럴거야.
    - 이것을 Rao-Blackwellization 이라 한다.

### Rao-Blackwellization

- variable 사이의 dependencies를 이용하기 위한 factorization
  - $p(a,b) = p(b\|a)p(a)$
  - $p(a)$는 known value라고 하자.
  - 만약, $p(b\|a)$가 효율적으로 계산이 가능하다면?
    - a가 pose(particle)을 의미 -> particle이 given이므로 $p(a)$는 known
    - b는 map -> $p(map\|pose)$ 을 계산하는 것이 관건!
  - 그렇다면 $p(pose, map)$ , 즉, SLAM을 빠르게 계산할 수 있다.
- $p(x_{0:t}, m_{1:M} \| z_{1:t}, u_{1:t})=$  
  $p(x_{0:t}\|z_{1:t},u_{1:t})p(m_{1:M}\|x_{0:t},z_{1:t})$
- $p(x_{0:t}\|z_{1:t},u_{1:t}) $ : path posterior
    - 이 부분은 particle filter로 계산이 된다!
  - $p(m_{1:M}\|x_{0:t},z_{1:t})$ : map posterior
    - 그럼 이 부분은 어떻게 구할 것인가?

### 핵심 아이디어



![1571532380618](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571532380618.png)

- 이전의 state는 화살표로 연결된 다음 state에 error를 전파한다.

![1571532534210](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571532534210.png)

- robot의 pose 는 particle로 인해 known이라는 가정
- landmark들 사이에는 robot pose로 연결되어 있다.
- 그런데, 로봇의 pose가 known이면?
  - map들간의 연결고리가 끊어진다!
- landmark들은 서로 conditionally independent!
- $p(x_{0:t}, m_{1:M} \| z_{1:t}, u_{1:t})=$  
  $p(x_{0:t}\|z_{1:t},u_{1:t})p(m_{1:M}\|x_{0:t},z_{1:t})$  
  $p(x_{0:t}\|z_{1:t},u_{1:t})\prod_{i=1}^M p(m_i\|x_{0:t},z_{1:t})$  
  - 왜 계산이 편한가?
    - High dimensional(2xM) EKF를 한번 계산하는 것 보다 low dimensional(2x2) EKF를 여러번 계산하는 것이 효율적이기 때문에.
  - 2x2 dimensional kalman filter로 풀 수 있다!

### Modeling robot's path 

- 각 Sample은 어떻게 modeling되는가?
  - $p(x_{0:t}\|z_{1:t}, u_{1:t})$

- particle은 과거의 trajectory를 수정하지 않는다. 즉, 새로운 위치만을 계속 update.
- 즉, 따라서 과거의 정보를 담고 있을 필요가 없다.
  - online slam
  - 현재의 위치 정보만을 포함.

## Fast SLAM

- Particle filter와 EKF를 이용한 SLAM 알고리즘을 보자.

- proposed by Montemerlo et al. in 2002

![1571533452500](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-06-Feature-based-SLAM-FastSLAM.assets\1571533452500.png)

- 각 landmark는 2x2 EKF에 의해 표현된다.
- 각 particle은 M개의 individual EKFs를 포함한다.

![image-20191106232531696](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-06-Feature-based-SLAM-FastSLAM.assets\image-20191106232531696.png)

- Odometry model로  partcle들을 이동시킴.
- Proposal distribution에 맞게 particle을 위치 시킨다.

![image-20191106232638078](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-06-Feature-based-SLAM-FastSLAM.assets\image-20191106232638078.png)

- particle이 sensor info.가 given일 때, map의 위치를 잘 맞추었는가를 통해 importance weight를 매긴다.

- particle 1은 어느정도 잘 맞지만 2, 3은 안맞다.(low likelihood)

- 이 정보들을 바탕으로 weight를 매긴다!

![image-20191106233002189](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-06-Feature-based-SLAM-FastSLAM.assets\image-20191106233002189.png)

- EKF를 통해 map을 update한다. (small tiny EKF update)
- 모든 sample에 대해 map update를 해야한다.
  - sample수 x landmark 수 만큼의 EKF update

### key step of Fast SLAM 1.0

1. motion model에 따라 sample 생성 (particle에 motion model 적용)
   - $x_k^{[k]} - p(x_t\|x_{t-1}^{[k]},u_t)$
   - 이때, noise term이 proposal distribution
   - proposal distribution을 무엇으로 쓰느냐에 따라 성능도 달라질까?
2. importance weight 계산
   - $w^{[k]} = \|2\pi Q\|^{-\frac{1}{2}}exp(-\frac{1}{2}(z_t-\hat{z}^{[k]})^TQ^{-1}(z_t-\hat{z}^{[k]})) $
   - z 가 실제 관측, $\hat{z}$ 가 EKF에서의 예측 값
3. update belief of observed landmarks (EKF update rules)
4. Resampling

### Part1

![1571575822194](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571575822194.png)

- 1~4 : motion model
- 5 : association
- 7 : 로봇에서 landmark까지 나온 거리와 로봇의 위치를 바탕으로 절대 좌표 계산
- 9 : sensor space에서의  noise를 절대 좌표 space로 옮김.

![1571576414547](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571576414547.png)

- robot pose update는 particle filter를 이용한다.
  - particle은 robot pose와 landmark의 state정보를 포함한다.
- 근데 그럼 sensor model은 gaussian을 가정하는 것인가?
  - 즉, landmark의 위치는 mono modal distribution을 가정하는 것 같다.

![1571576428183](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571576428183.png)



### Weight가 왜 sensor model인가?

- __Importance sampling principle__

![1571577512879](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571577512879.png)

- $p(x_{1:t}\|z_{1:t-1},u_{1:t})=p(x_t\|x_{t-1},u_t)p(x_{1:t-1}\|z_{1:t-1},u_{1:t-1})$
- $p(a,b)=p(b|a)p(a)$
- $p(b|a) = p(x_t\|x_{1:t-1},u_t)=p(x_t\|x_{t-1},u_t) $
- $p(a) = p(x_{1:t-1}\|z_{1:t-1},u_{1:t-1})$
- proposal은 단순히 factorization
- target은 bayes rule

![1571577973515](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571577973515.png)

- ![1571578063656](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571578063656.png)
- marginalization form으로 표현
- 내가 어디있는지 알고 랜드마크가 어디있는지 알면?
  - 이전의 모든 포즈와 이전의 모든 관측을 없앨 수 있다.
- 밑에꺼에서 관측이 이전꺼까지 밖에 없으니까 현재 위치도 이전 위치까지만 본다.
- $p(z_t\|x_t,m_j)$ : 로봇 포즈를 알고 landmark 위치를 알 때의 observation
  - $N(z_t;\hat{z}^{[k]},Q_t)$ : predicted observation과 sensor noise cov.가 주어졌을 때의 sensor distribution
- 오른쪽꺼도 보면 landmark 역시 gaussian distribution?

![1571578660296](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571578660296.png)

- 왼쪽 distribution은 sensor space의 노이즈를 절대좌표로 옮김. 오른쪽 꺼는 sensor space
- 가우시안의 곱은 covariance의 합
- 결과는 그래도 jacobian을 썼으니 approx.임.
- gaussian 간의 convolution이므로 위와 같이 된다?? 확인 필요.



## Data Association

![image-20191114213123912](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-06-Feature-based-SLAM-FastSLAM.assets\image-20191114213123912.png)

- 각 파티클은 다른 data association을 만든다!

  - 더 좋은 data association을 하는 particle들이 더 잘 살아남는다.

  - 위 그림처럼 association의 관점에서 multi modal state를 갖는다.

- data association 알고리즘을 따로 적용하지 않고 resampling 과정을 통해 data association을 한다.
  - association은 random하게 적용한다.
  - likelihood가 너무 낮으면 새로운 association을 생성한다.

![image-20191114214446369](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-06-Feature-based-SLAM-FastSLAM.assets\image-20191114214446369.png)

- EKF와 FastSLAM 비교
  
  - Motion noise가 증가할 수록 EKF는 approximation error로 인해서 성능이 떨어진다.
  - FastSLAM에서 robot pose update는 particle filter를 사용하기 때문에 approximation이 없다.
    - 즉, motion noise가 크게 영향을 주지 않는다.
  

![1571750295109](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-10-19-FastSLAM.assets\1571750295109.png)

- particle update에서 landmark의 수에 독립이다.
- sensor update도 관측하는 곳만하므로 landmark 수에 영향 없다.
- Resampling 할 때
  - 최악일 때는 N개의 particle이 가진 M개의 landmark가 복사된다.
- 이거를 데이터 구조를 잘 이용해서 효율적으로 바꾸자!

![image-20191114215849763](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-06-Feature-based-SLAM-FastSLAM.assets\image-20191114215849763.png)

- new particle을 update할 때, 새롭게 모든 데이터를 복사하는 것이 아니라 update안된 부분의 포인팅만 바꾼다.
- update된 값만 새로 설정한다.

![image-20191114220133632](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-06-Feature-based-SLAM-FastSLAM.assets\image-20191114220133632.png)

- landmark에 접근하는데 logarithm complexity(tree algorithm에서 find하는데 걸리는 시간)
  - 기존에는 memory에 직접 indexing하지만 M번 접근해서 복사해야했다.
- 메모리도 매우 적게 차지한다.