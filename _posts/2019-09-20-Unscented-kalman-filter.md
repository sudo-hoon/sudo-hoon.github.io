---
layout: post
title: "Unscented Kalman Filter"
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

# Unscented Kalman Filter

- Kalman filter는 linear model에 대해 최적이다.
- EKF는 linearization을 통해 비선형 모델을 kalman filter에 적용한다.
  - mean을 기준으로만 선형화를 하기 때문에 mean과 먼 곳은 매우 부정확하다.
- 왜 선형이어야 하는가?
  - __recursive한 filter equation을 closed form으로 정리하기 위해서는 state가 gaussian 분포를
    유지해야하기 때문이다. -----?__
- Unscented kalman filter(UKF)는 model을 linearization을 하는 것이 아닌 비선형 시스템을 통과한
  distribution을 gaussian으로 approximation한다.


### Goal

- EKF 보다 더 정확한 approximation을 하는 것
  

## Unscented Transform

- 방식 :

  1. sigma point를 계산한다.

  2. non-linear system에 통과시켜 sigma point를 transform 한다.

  3. transformed & weighted sigma point들로 부터 gaussian parameter($$\mu, \sum$$)을 계산한다.

     ![weighted sum between prediction and measurement]()

  ![weighted sum between prediction and measurement]()

  ![weighted sum between prediction and measurement]()
  

### Sigma points, $$\chi^{[i]}$$

- state의 distribution에 기반한 임의의 샘플
  - 해당 샘플의 확률에 해당되는 weight, $$w^{[i]}$$를 갖는다.
  - continuous prob. distribution에서 discrete한 sample을 뽑는 것과 같다.
    

##### Sigma point의 성질

1. $$\sum_i w^{[i]}=1$$
2. $$\mu=\sum_i w^{[i]}\chi^{[i]}$$
3. $$\sum = \sum_i w^{[i]}(\chi^{[i]}-\mu)(\chi^{[i]}-\mu)^T$$

- w는 평균의 역할을 하기 때문에 총 합이 1이다.
- 2, 3번의 $$\mu, \sum $$ 를 보면 마치 pmf(probabilty mass function)의 평균, 분산과 같다.

- 위 조건을 만족하면 되기 때문에 무수히 많은 sigma point와 weight가 가능하다.
  

##### Sigma point의 결정

- 강의에서 제안하는 방식은 다음과 같다.
  - $$\chi^{[0]} = \mu$$
  - $$\chi^{[i]}=\mu+(\sqrt{(n+\lambda)\sum})_i\ \ for\ \ i=1, 3..., n-1$$
  - $$\chi^{[i]}=\mu-(\sqrt{(n+\lambda)\sum})_i\ \ for\ \ i=2,4 ..., n$$
  - n : dimension, $$\lambda$$ : scailing parameter
    - $$\chi^{[0]}$$는 이전의 state의 평균이 그대로 non linear system을 통과한 값이다.
    - $$\chi^{[i]} $$는 평균을 중심으로 분산과 scailing parameter에 비례해서 떨어진 대칭되는 값이 선택된다.
      - 분산이 큰 방향일 수록 멀리 떨어진 값이 선택된다.
      - scailing parameter에 따라 optimization이 가능하다.
      - 마치 PCA에서 principal axis를 구하는 것과 같다.
        

#### Squart root의 계산

- Sigma point를 구할 때, square root가 필요하다.
  - $$A=BB$$를 성립하는 B를 A의 square root라 한다.[^1]



##### 1. eigen value decomposition[^2]

- $$\sum = SS$$ -> $$\sqrt{\sum}=S$$
-  ![weighted sum between prediction and measurement]()
- $$SS=(VD^\frac{1}{2}V^{-1})(VD^\frac{1}{2}V^{-1}) = VDV^{-1}=\sum$$
- Covariance matrix는 real-valued symmetric matrix이므로 항상 eigen value decomposition이 가능하며 square root는 orthogonal matrix이다.
  

##### 2. Cholesky Matrix Square Root[^3]

- $$\sum = LL^T$$ -> $$\sqrt{\sum}=L$$
  - 왜 L이 square root인가? : https://math.stackexchange.com/questions/2767873/why-is-the-square-root-of-cholesky-decomposition-equal-to-the-lower-triangular-m
- numerically stable solution이며 일반적으로 UKF에서 이 방식이 더욱 쓰인다.

##### 굳이 principle axis에 놓인 sample을 선택 안해도 된다는데 그래도 최적의 결과가 나오는가?



#### Weight의 계산

- $$w_m^{[i]}$$ : weight for mean, $$w_c^{[i]}$$ : weight for covariance

  1. $$w_m^{[0]}=\frac{\lambda}{n+\lambda} $$, $$w_c^{[0]}=w_m^{[0]}+(1-\alpha^2+\beta) $$

  2. $$w_m^{[i]}=w_c^{[i]}=\frac{1}{2(n+\lambda)}$$ 

- k >= 0

- $$\alpha$$ : (0 : 1]

- $$\lambda = \alpha^2(n+k) - n$$

- $$\beta = 2$$

  - k와 $$\alpha$$ 는 sigma point이 mean으로 부터 얼마나 떨어져있는가를 의미한다.
  - $$\beta$$ 값은 2가 gaussian의 경우에 가장 최적이다.

- __어떻게 최적의 parameter 값을 결정하는가?__

  

#### Gaussian approximation

- non-linear system을 통과한 sigma point들과 weight를 이용하여 gaussian approximation을 한다.
- $$\mu'=\sum_{i=0}^{2n}w_m^{[i]}g(\chi^{[i]})$$
- $$\sum'=\sum_{i=0}^{2n}w_c^{[i]}(g(\chi^{[i]})-\mu')(g(\chi^{[i]})-\mu')^T$$



##### Examples

 ![weighted sum between prediction and measurement]()

 ![weighted sum between prediction and measurement]()

 ![weighted sum between prediction and measurement]()

- 

### UKF Algorithm

#### Prediction

 ![weighted sum between prediction and measurement]()

#### Correction

 ![weighted sum between prediction and measurement]()



### UKF vs EKF

 ![weighted sum between prediction and measurement]()

 ![weighted sum between prediction and measurement]()

 ![weighted sum between prediction and measurement]()



- linear model의 경우에는 EKF와 UKF가 같다.
- EKF 보다 더 좋은 approximation이다.
- 같은 complexity class이지만 EKF보다 조금 느리다.
- 여전히 gaussian distribution의 한계를 갖는다.
- EKF vs UKF
  - EKF : non-linear system을 linearization
  - UKF : non-linear system을 통과한 output을 gaussian approximation
    - 둘 다 시스템을 통과한 결과가 gaussian distribution을 유지한다는 점은 같다.



[^1]: https://en.wikipedia.org/wiki/Square_root_of_a_matrix
[^2]: https://darkpgmr.tistory.com/105
[^3]: 