> 본 게시글은 Cyrill Stachniss의 [Robot mapping(WS 2013/14)](http://ais.informatik.uni-freiburg.de/teaching/ws13/mapping/) 강의를 요약 및 정리한 글입니다. 
> 틀린 내용을 지적해주시면 감사드리겠습니다.

# Homework3

## Exercise 1 : Bayes Filter and EKF

### (a) Describe briefly the two main steps of the Bayes filter in your own words.

- Prediction step과 Correction step이 있다.
- Prediction step에서는 과거의 state와 현재의 motion command를 바탕으로 새로운 state를 예측한다.
- Correction step에서는 Prediction step에서 예측한 현재의 state을 sensor값에 기반한 state의 likelihood로 correction한다.
  - P(z|x)bel_(x)에서, bel\_(x)에서 x=X 일 때, 가장 큰 확률을 갖는다고 하자. 하지만 센서에서 읽은 값 z=Z에 대해 P(z=Z|x=X)는 굉장히 작은 값일 수 있다. 즉, 이전의 predicted state인 bel\_(x)는 센서를 통해 얻은 likelihood로 조금 더 현실적인 값으로 보정된다.

### (b) Describe briefly the meaning of the following probability density functions : p(xt| ut, xt−1), p(zt| xt), and bel(xt), which are processed by the Bayes filter.

- p(xt|ut, xt-1) : 이전의 state와 현재의 command가 주어졌을 때, 현재의 state에 대한 distribution, bel_(x)
- p(zt|xt) : 현재의 state에 대한 likelihood
- bel(xt) : 이전의 state, 현재의 command, 현재의 measurement가 주어졌을 때, 현재의 state에 대한 distribution

### (c) Specify the (normal) distributions that correspond to the above mentioned three terms in EKF SLAM.
- EKF SLAM 강의자료 참고

#### (d) Explain in a few sentences all of the components of the EKF SLAM algorithm, i. e., µt , Σt , g, Gxt , Gt, Rxt, Rt, h, Ht, Qt, Kt and why they are needed. Specify the dimensionality of these component
- EKF SLAM 강의자료 참고

#### 

## Exercise 2: jacobians

#### (a) Derive the Jacobian matrix Gxt of the noise-free motion function g with respect to the pose of the robot. Use the odometry motion model as in exercise sheet 1:
