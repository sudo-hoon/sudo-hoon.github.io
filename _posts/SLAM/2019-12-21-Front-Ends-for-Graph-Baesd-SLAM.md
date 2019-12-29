---
layout: post
title: "Front-Ends for Graph Baesd SLAM"
date: "2019-12-21"

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

# SLAM Front-Ends

- 3가지 SLAM Front end system에 대해 소개한다.
- 어떻게 constraint를 구할것 이냐?

##  How to get Constraint?

- constraint는 matching observation에 의해 구할 수 있다.

- matching observation의 종류
  1. Dense scan-matching
     - 현재 scan한 raw observation을 지난 raw observation과 비교하여 현재 위치가 이전의 위치와 같은 곳인지 확인하는 방법.
  2. Feature-based matching
     1. 관측된 observation을 landmark로 인식해서 이전의 landmark와 비교하는 것
  3. Descriptor-based matching
     1. feature extractor(SIFT, SURF)를 통해 feature를 뽑아서 matching하는 것

- match를 할 때 어떤 node들을 match할 것인가?
  - 센서 A와 node B1, B2가 있다.
  - B1의 A에 대한 uncertainty range와 B2의 A에 대한 uncertainty range를 구한다.
    - 이것은 A를 fix하고 information matrix의 inverse를 취해서 diagonal 성분을 구하면 된다.
    - A를 fix : Hessian에서 A에 해당되는 row 와 column을 삭제)
  - 이때, sensor range안에 들어오는 node만을 고려해서 matching을 한다.
    - 아래 그림의 상황에서 B1만 matching을 한다.

![image-20191221191911330](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-12-21-Front-Ends-for-Graph-Baesd-SLAM.assets\image-20191221191911330.png)

- Uncertainty를 효율적으로 계산하는 방법?
  - Hessian을 inverse하는 방법은 계산량이 너무 많다.
  - __Dijkstra expansion__을 사용하자.
- __Dijkstra expansion__
  - 이때, loop closing은 무시한다.
  - 시작 pose(uncertainty가 가장 낮은 지점)부터 가장 가까운 node들에게 uncertainty를 증폭 및 전파하는 방식
  - loop closing을 무시하기 때문에 node가 뒤로갈 수록 uncertainty가 매우 커진다.
    - 한번씩 inversion을 하는건가?
- Match 확인
  - 위 방식으로 match를 할 node들을 구했다.



### ICP-Based Approach

- match할 node의 uncertatinty range 내의 한 점에서 observation과 지금 현재의 observation에서 corresponding point들 간의 거리를 비교해서 정확한 위치를 찾는다

- 문제는?
  - 완전히 같은 지역(무한히 긴 복도)를 지나면 잘못된 지역에서 optimization된다.
    - ambiguity in the environment
    - 즉, 여러 minima가 존재할 수 있고 initial guess에 크게 영향 받는다.
  - initial guess에 매우 민감, local minima에 빠지기 쉽다.

- 연속된 map들 사이에서 ICP를 한다.



## vision

- description feature를 추출한다.

### SURF

- 장면(pose)마다 SURF feature를 저장한다.
- camera pose를 구하고 SURF feature의 3D 위치를 구한다.
  - imu로 roll, pitch 구한다.
  - camera 이미지로 x,y,z,yaw를 구한다
- camera pose estimation
  - kd-tree를 이용해서 possible matches를 찾는다.
  - match들을 quality(?)에 따라 sort한다.
  - 2개의 best match를 찾아서 camera pose를 우선 잡는다.
  - 그리고 다른 feature들을 한 이미지에서 다른 이미지로 projection해서 잘 맞는지 본다.
  - 계속해서 가장 잘 맞는 놈을 찾는다.
  - RANSAC과 비슷한 방법이다.
- 위 방법은 3가지 다른 곳에서 다른 방식으로 쓰인다.

1. Visual odometry
   - 이전에 관측된 N개들과 현재의 feature들을 matching한다.
   - bundle adjustment 같은 느낌이다.

2. localization
   - 예측된 odometry 근처에서 map과 현재 관측된 feature들을 match한다.

3. loop closing
   - 모든 map의 feature들과 현재의 feature를 비교한다.



## Robustness

- ICP를 이용한 문제 중 Ambiguity 문제는 심각하다.

- 몇 가지 문제 상황에서 constraint를 만들면 안된다.

- __global Ambiguity__ 

  ![image-20191221201559042](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-12-21-Front-Ends-for-Graph-Baesd-SLAM.assets\image-20191221201559042.png)

  - A의 uncertainty 범위 안에 3개의 모호한 matching이 있다.

- __global Sufficiency__

  - ![image-20191221202100111](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-12-21-Front-Ends-for-Graph-Baesd-SLAM.assets\image-20191221202100111.png)
  - A의 uncertainty 범위안에 A와 매칭되는 것은 유일하다.

- __local Ambiguity__

  - ![image-20191221202136846](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-12-21-Front-Ends-for-Graph-Baesd-SLAM.assets\image-20191221202136846.png)
  - A의 uncertainty 범위안에서 A가 matching될 수 있는 방법이 너무 많다.
  - 길고 연속적인 구조에서 발생하는 문제.



### Global Match Criteria

1. Global Sufficiency : A는 uncertainty 범위 내에서 match되는 부분이 하나 밖에 없다.
2. Local unambiguity : Overlapping match되는 곳이 없다.

- Match로 인정되기 위해서는 위 두 가지 기준을 만족해야함.

![image-20191221203159893](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-12-21-Front-Ends-for-Graph-Baesd-SLAM.assets\image-20191221203159893.png)

- 일단 scan matching 떄린다?
- 그리고 묶는다?
- 그리고 근처의 pose들과 관련된 뭐시기 criterion2가 local ambiguity
- criterion1이 global ambiguity?
- 

### locally consistent matches

- pose to pose hypotheses는 서로 동일해야한다.
- loop를 만들어서 rigid transformation했을 때, consistent하면 인정
  - 즉, 4번의 transformation이 identity transformation이어야한다.
- ![image-20191221204140422](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-12-21-Front-Ends-for-Graph-Baesd-SLAM.assets\image-20191221204140422.png)
- 각 hypothesis pair로 matrix를 만들고 높을수록 identity일 확률이 높다는 것.

- indicator vector
  - hypothesis를 1 또는 0으로 각 원소를 채운 subset(한 pair만 1로 두는 듯?)
  - 해당 group이 정확한 hypothesis로 구성돼 있는지 보는 것.
  - 이것을 matrix와 곱해서 점수를 구해서 가장 높은 indicator vector
  - eigen vector eigen value problem이 됨.
  - eigen vector 개수만큼 multiple solution이 존재하는 것.
  - eigen value가 클 수록 score가 높다.
  - 첫 번째로 큰 값과 두 번째로 큰 값 사이의 비율이 2 이상은 되어야 locally unambiguous
  - 비슷하면 ambiguous하다는 뜻이고 mix-max 방식이 힘을 발휘한다.
  - 선택된 eigen vector의 값은 0~1로 normalize하고 반올림해서 0 또는 1로 만든다.
  - 이 방법으로 local ambiguous는 따질 수 있다.

### Global Consistency

- 이전까지 본 적이 없던 지역에서 관측한 것이 이전에 관측한 곳과 같은 형태임을 발견
- 이때, 이것을 fit 해도 되는지??
- A의 크기를 측정하고 ellipse의 크기를 측정한다. covariance ellipse의 크기의 반 이상이 A의 크기여야한다.



대부분 slam 시스템이 위 방법을 쓴다.

