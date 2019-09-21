---
layout: post
title: "Sparse Extended Information Filter(SEIF)"
date: "2019-09-21"
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

# Sparse Extended Information Filter for SLAM

- 본 강의에서는 SLAM을 moments form에서 canonical form으로 바꾸었을 때의 이점과 그것의 SLAM에 대한 적용에 대해서 다룬다.
- Information matrix가 sparse할 때 갖는 이점과 sparse하게 만드는 방법에 대해 다룬다.

## Motivation

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img1.png){:.aligncenter}

- Correlation matrix를 Information matrix로 변환 했을 때, 많은 부분들이 매우 작은 값을 가진다.
  - 이것을 0으로 appox.한다면 매우 sparse 한 matrix가 된다.
  - 이것이 Sparse Extended Information Filter (SEIF)의 핵심 아이디어.
- SEIF는 IF의 approximation 버전이다.
  - SEIF 에는 sparsificaiton step이 추가로 있다.
  - 즉, 같은 것이 아님.
  - EIF보다 조금 성능이 나쁠 수 있지만 계산에 있어 매우 매우 효율적이다.

## Sparse Information Matrix

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img2.png){:.aligncenter}

- Sparse Information Matrix를 이해하기 위한 정의들
  - Link : information matrix에서 활성화된(non-zero) cross correlation 영역
    - __information matrix이기 때문에 information이라고 해야하나?__
    - 더 어두울수록 강한 link, 어떻게 보면 constraint를 의미한다.
      - 즉, 불확실성이 강할 수록 강한 link를 가진다.
      - 왜냐하면 landmark가 확실하다면 다른 landmark의 불확실성에 영향을 안받으므로.
  - active landmark : 로봇이 관측 중이거나 로봇의 pose와 link가 있는 landmark
    - 즉, information matrix에서 robot pose의 영역에서 활성화된 애들  
  - passive landmark : robot과 직접적인 link가 없는, 즉 관측중이지 않은 landmark

### __Information Matrix__

- __Information matrix의 의미__[^1]

  - 각 off-diagonal 성분은 해당 component간의 partial correlation[^2]을 의미한다.

  ![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img3.png){:.aligncenter}

  - partial correlation : 다른 component들이 given일 때, 해당 components 간의 correlation.
    - 위 그림에서 x, y, z 3가지 random variable이 있을 때, z이 given이라면 $S_z$ 평면에 x와 y를 정사영하는 것을 의미한다.
      - 이때, 둘 사이의 각도 $\phi$가 partial correlation.
  - 즉, link가 약한 곳은 다른 성분들에 대한 충분한 정보가 있다면 uncorrelated된다는 의미이다.
  - diagonal 성분은 해당 component가 얼마나 확실한지를 의미한다.

- Information matrix는 node들(robot, landamark)과 link로 이루어진 그래프의 형태이다.
  
  - _gaussian markov random field와 같다----?_
- link로 직접 연결되지 않은 애들은 conditionally independent하다.
  - 두 노드를 제외한 모든 노드가 given일 때, 두 노드는 independent하다.
    - ex)  (no.1) <-> (no.2) <-> (no.3) 가 있을 때, 2가 given이면 1에 대한 knowledge는 3에 대해 어떠한 정보도 주지 않지만 2에 대해 모른다면 no.1에 대한 정보는 no.3에 대한 정보를 포함한다.
    - conditionally independent : $p(no.1, no.3 \| no.2) = p(no.1\|no.2) * p(no.3\|no.2)$
    - 위의 그림에서 $\phi$가 수직을 의미

- 가까운 landmark는 큰 값을 가진다.
- 대부분의 off-diagonal 원소는 0에 가까운 값을 가진다. 
  - 즉, 대부분 무시 가능하다.
    - conditionally independent하다.
  - __off diagonal 성분을 constant의 개수로 non-zero 값을 가지게 한다!__
    - robot과 ladnamrk간의 link 개수를 유지하는 것이 핵심 idea.

## Measurement와 motion이 information에 주는 영향

### Measurement 1

 ![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img4.png){:.aligncenter}

- 로봇의 첫 위치는 확실하므로 진한 색, 즉, 큰 값을 갖는다.
- robot의 포즈를 정확히 알기 때문에 m1가 m2는 conditionally independent, 서로가 정보를 주지 않음.
- 그러므로 link가 없으므로 하얀색

### Measurement 2

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img5.png){:.aligncenter}

### Measurement 3

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img6.png){:.aligncenter}

- Measurement는 landmark간의 link를 발생시키지 않는다.

### Motion

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img7.png){:.aligncenter}

- robot이 다음 pose를 취하며 불확실성이 강해졌기 때문에 첫 cell이 옅어졌다.

- robot이 다음 pose를 취해서 $x_t$가 과거가 되었을 때, 그것은 더 이상 사용되지 않는다. 
  - 왜냐하면 marginalize 되기 때문이다.(online slam)
- 그 node가 제거되면서 그 node($x_t$)의 이웃된 노드들은 모두 connect된다
  - $x_t$-> $x_{t+1}$가 되면서 이전의 known 노드는 marginalize되고 불확실성은 더 해진다.
  - m1과 m2 사이에 link가 생긴다.
    - robot pose와 connection되어 있던 link가 옅어지며 m1과 m2사이에 propagtion된다.
    - 그러면 information matrix가 계속 fill-in 돼서 sparse하지 않아지는 것 아닌가?
      - -> 계속 모션을 할 수록 link들이 약화되므로 sparse해진다.

### Sparsification

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img8.png){:.aligncenter}

- 약화된 링크를 0으로 만드는것(끊는것)을 sparsification이라 한다.
  - 로봇  pose와 m1간의 link를 없애면?
    - m1과 m2, 로봇 포즈와 m2간의 link가 강해진다.(링크가 propagation 된다.)
    - m2를 알면 m1과 로봇 포즈간에는 conditionally independent하다.
    - 어떻게 sparsification을 할 것인가?

### Link

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img9.png){:.aligncenter}

- SEIF  SLAM은 active landmark 와 passive landmark를 나눈다.
  - Active landamrk : 현재 관측하는 landmark
    - active landmark의 개수를 고정, 즉, 관측의 개수를 고정한다.
  - passive landmark : 나머지 전부
    - robot pose와 link를 갖지 않는다.
  - was active, now passive : active였던 landmark지만 새로운 motion으로 passive가 된 것.
    - 현재의 active landmark들과 link를 갖는다.
- 다음 measurement에서는 m3와 robot사이에 link가 생기고 다음 motion에서는 m2와 m3간에 link가 생긴다.
  - m1과 $x_{t+1}$ 사이에 link가 없으므로 m1과 m3 사이에는 link가 생기지 않는다.
- active landmark의 개수가 커질 수록 EIF와 비슷해진다.
  - 즉, 모든 관측이 active landmark = EIF
  - active landmark의 수를 정해 놓고 관측을 할 때마다 sparsification을 통해 active landmark의 수를 조절한다.

## SEIF SLAM Algorithm

----

1. Motion Update
2. Measurement update
3. Update of the state estimate
4. Sparsification

----

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img10.png){:.aligncenter}

- EIF에서 추가적인 과정, update of the state estimate가 필요.
  - non-linear function에 input으로 mean이 필요하다.
    - $$\mu =  \Omega^{-1}\xi $$ 를 통해 구하면 계산 시간이 오래 걸린다.
    -  그것을 parameterizaiton해서 구하지말고 mean 값을 이 단계에서 계산하여 사용한다.

### Prediction Upate Step

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img11.png){:.aligncenter}

- 대부분의 과정이 EKF와 같다.

  - 하지만 SEIF의 prediciton은 EKF보다 느리다.
  - 계산이 오래걸리는 마지막 5번 단계의 계산을 constant time complexity로 만드는 것이 중요.
    - Sparsification을 한 Information matrix을 이용! 

- __Information matrix calculation__ :

  

  - $\bar{\Omega}_t=\bar{\Sigma} ^{-1}=[G_t\Omega_{t-1}^{-1}G_t^T+R_t]^{-1}  
    =[\Phi_t^{-1}+R_t]^{-1}$  
  - $\Phi_t=[G_t\Omega_{t-1}^{-1}G_t^T]^{-1} =[G_t^T]^{-1}\Omega_{t-1}G_t^{-1}$
  - Matrix inversion lemma : $(R+PQP^T)^{-1}$  
    $=R^{-1}-R^{-1}P(Q^{-1}+P^TR^{-1}P)^{-1}P^TR^{-1}$
    - $Q^{-1} $은 known이고 P가 sparse한 Nx3 matrix이다.   
      (robot pose covariance matrix(3x3)과 Projection matrix)
    - 만약 여기서 $R^{-1} $이 sparse하면 매우 빠르게 전체 연산을 할 수 있다.

  ![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img12.png){:.aligncenter}

  - $R_t^{x-1} + F_x\Phi_tF_x^T$ : 3x3 matrix므로 매우 빠르게 inverse 계산이 가능하다.
  - 위에서 언급한 Matrix inversion lemma와 같이 $\Phi_t$ 가 sparse하면 빠른 연산이 가능하다.
  - $\Phi_t=[G_t\Omega_{t-1}^{-1}G_t^T]^{-1}$

  ![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img13.png){:.aligncenter}

  - 즉, $G_t$는 sparse하다.
  - $\Omega_{t-1}^{-1}$ 역시 sparsification으로 sparse하다.
  - __따라서, $\Phi_t$는 sparse하다__

- $\xi$ 계산은 생략한다.
  
  - __알고리즘에서 사용되지 않는데 왜 계산해야하는가?-----?__
  
- 최종 알고리즘

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img14.png){:.aligncenter}

- parameter 값들은 EKF를 참고한다.

### Measurement Update Step

- EIF와 거의 같다.

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img15.png){:.aligncenter}

![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img16.png){:.aligncenter}



### State Estimate Update Step(Mean recovery)

- Canonical form임에도 불구하고 mean을 구해야 한다.
  - linear motion model, linearized meas. model, sparsification에서 필요하다.
- 하지만 $\mu=\Omega^{-1}\xi$ 로 구하기에는 계산이 비효율적이다.
  - $\Omega$는 Measurement step을 거쳤기 때문에 아직 sparse하지 않다.
  - approximation으로 mean을 구해보자.
    - $\hat{\mu}=argmax(p(\mu))$  
      $=argmax_{\mu}exp(-\frac{1}{2}\mu^T\Omega\mu+\xi^T\mu)$
    - 여기서 $\mu$는 robot의 pose와 active landmark만 사용한다.
      -  local optimization with constant number of dimension

### Sparisification Step

- information matrix를 sparsification한다는 것은 무슨 의미인가?ㅉ
  - 해당 노드들을 conditionally independent하게 만든다는 것
  - $p(a,b,c)$에 대해 c가 given, a와 b가 cond. indep.라면  
     $\tilde{p}(a\|b,c)=p(a\|c)$, $\tilde{p}(b\|a,c)=p(a\|b)$
- __landmark 사이의 link도 제한된다.---?__
  - __그 이유는 한번에 활성화되는 link들은 robot pose와 연결된 애들밖에 없어서?---?__
  - __동시에 active되는 landamrk들만 사이에 link가 존재한다?---?__
- Sparsification의 수학적인 표현
  - b가 given일 때
  - $p(a,b,c)=p(a\|b,c)(p(b\|c)p(c)$  
    $\approx p(a\|c)p(b\|c)p(c)$  
    $=\frac{p(a,c)p(b,c)}{p(c)}$
- States of Landmark
  - $m=m^++m^0+m^-$
  - $m^+$ : active, $m^0$ : active to passive, $m^-$ : passive
    - Sparsification은 $m^+$를 $m^0$로 만든다.
    - Sparsification은 passive landmark에는 영향을 주지 않는다.

- __Sparsification 과정__

  - $p(x_t,m\|z_{1:t},u_{1:t})=p(x_t,m^+,m^0,m^-\|z_{1:t},u_{1:t})$  
    z, u를 생략하면

  - $p(x_t,m)=p(x_t,m^+,m^0,m^-)$  
    $=p(x_t\|m^+,m^0,m^-)p(m^+,m^0,m^-)$    
    -> robot의 pose를 계산할 때, passive landmark는 아무런 영향을 주지 않는다.  
    이것을 반영하면

    $=p(x_t\|m^+,m^0,m^-=0)p(m^+,m^0,m^-)$       
    -> Sparsification을 적용하면, 즉, conditional independent로 만들면(given $m^+ and\  m^-=0$)  
    $\approx p(x_t\|m^+,m^-=0)p(m^+,m^0,m^-)$  
    $=\frac{p(x_t,m^+\|m^-=0)}{p(m^+\|m^-=0)}p(m^+,m^0,m^-)$  
    $=\tilde{p}(x_t,m)$

  - $\tilde{p}(x_t,m\|z_{1:t},u_{1:t})\approx \frac{p(x_t,m^+\|m^-=0,z_{1:t},u_{1:t})}{p(m^+\|m^-=0, z_{1:t},u_{1:t})}p(m^+,m^0,m^-\|z_{1:t},u_{1:t})$
    - Gaussian pdf의 곱은 Covariance의 덧셈과 같다.
      - $\tilde{\Omega_t}=\Omega_t^1-\Omega_t^2+\Omega_t^3$
      - $\Omega_t^0 : p(x_t,m^+,m^0\|m^-=0)$ (Conditioning)
      -  $\Omega_t^1 : \int {p(x_t,m^+,m^0\|m^-=0)}dm^0$ (Maginalizing, 나머지도 같음)

- - 

- __Sparsificaiton Algorithm__

  ![](/images/posts/SLAM/2019-09-21-Sparse-Extended-Information-Filter/img17.png){:.aligncenter}

  - __Projection matrix는 어떻게 구하는가?------?__

- __Sparsification의 진정한 의미__

  > 강의에서는 information matrix에서 0에 가까운 크기의 원소를 0으로 approx.하는듯이 처음에 이야기하였는데, 수학적으로는 그 부분을 conditionally independent하다고 approximation을 했고 그로 인해서 그 원소는 0이되고 그 원소와 연결된 다른 원소로 weight가 propagation되는 것 같다.  
  > 즉, conditioning이랑 marginalizing이 propagation을 만든다.



### Conclusion

- Complexity

  - Prediction : constant complexity
  - Measurement : $O(N^2)$
  - State update : nearly constant
    - iteration이 필요하고 발산하는지 확인해야한다.
  - __Sparsification : ??----?__
  - 하지만 Correspondence가 계산 속도에 가장 큰 영향을 줌.

- Linear Memory

  - __왜? 매트릭스가 필요하니 $O(N^2)$아닌가?---?__

- 정확도

  - EKF보다 덜 정확하지만 훨씬 빠르다.
  - Sparsification가 mean recovery에서 approx.

- Active landmark의 수

  - 약 7개만 돼도 EKF와 비슷한 성능을 가진다.
  - 

- 질문
  - 그럼 어떤 landmark를 active시킬지 어떻게 고르지?

    - active로 만드는 알고리즘도 알아야할 듯.

      

[^1]: https://www.quora.com/What-is-the-inverse-covariance-matrix-What-is-its-statistical-meaning  
[^2]: https://en.wikipedia.org/wiki/Partial_correlation#Using_matrix_inversion