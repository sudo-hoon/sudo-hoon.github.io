---
layout: post
title: "Hierarchical Pose-Graphs for Online Mapping"
date: "2019-11-30"

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

# Hierarchical Approach to Least Squares SLAM

- 이 알고리즘을 실시간으로 쓸 수 있는가?
  - graph의 양을 고정시키면 된다.
  - 현재의 로봇 위치와 먼 곳과는 전체 그래프를 유지할 필요가 없다.
  - 로컬 map들을 잘 optimization하다 보면 전체 map이 잘 그려질 것이다.
- motivation
  - 너무 먼 곳의 그래프를 전체 다 갖고 있지 말자.
    - 어차피 상대적인 거리의 형태로 데이터를 얻으므로 한 먼 곳의 node들 중 한 곳만 그래프를 잘 유지해도 전체를 맞출 수 있다.

### Hierarchical pose-graph

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-30-Hierarchical-Pose-Graphs.assets\image-20191130104640754.png" alt="image-20191130104640754" style="zoom:80%;" />

- 구를 도는 로봇의 graph를 생각해보자.
- bottom layer : locally map에 대한 graph
- 멀 수록 높은 layer의 그래프 처럼 그래프를 유지한다.

### motivation

​	<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-30-Hierarchical-Pose-Graphs.assets\image-20191130105035220.png" alt="image-20191130105035220" style="zoom:50%;" />

- loop closing을 할 때, 빨간 원을 제외하고는 필요가 없는 노드들이다.

- 로봇은 순간이동하지 않으므로 local map에서만 자세한 graph로 optimization하면 모든 노드가 적당한 위치에 optimization될 것이다.



### Key Idea

1. low level optimization
   - 각 edge들을 group으로 묶고 각각 optimization을 한다.
   - 똑같은 곳을 다시 방문해서 생긴 노드들은 기존의 group boundary에 들어와있다면 포함시킨다.

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-30-Hierarchical-Pose-Graphs.assets\image-20191130105909923.png" alt="image-20191130105909923" style="zoom:50%;" />

2. low level representative
   - 각 group에서 대표들을 하나씩 뽑는다.

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-30-Hierarchical-Pose-Graphs.assets\image-20191130105927880.png" alt="image-20191130105927880" style="zoom:50%;" />

3. upper level graph(sparsified graph)
   - 각 group의 대표끼리 연결된 새로운 그래프를 만들고 optimization 한다.
   - 이때, group 대표들 간의 edge parameter 값은 locally optimization 결과로 정해진다.

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-30-Hierarchical-Pose-Graphs.assets\image-20191130110121898.png" alt="image-20191130110121898" style="zoom:50%;" />

4. upper level optimization
   - upper level에서 optimization을 해서 그래프를 고친다.

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-30-Hierarchical-Pose-Graphs.assets\image-20191130110218981.png" alt="image-20191130110218981" style="zoom:50%;" />

5. lower level propagation
   - upper level에서의 optimization은 lower level에 전파된다.
   - low level의 노드들은 대표에 rigid하게 묶여 있기 때문에 모든 node들이 영향을 받는다.

<img src="D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-30-Hierarchical-Pose-Graphs.assets\image-20191130110346683.png" alt="image-20191130110346683" style="zoom:50%;" />

### How?

1. 어떻게 new group을 만드는가?
   - 간단히 distance-based decision
   - new group의 first-node가 representative.

2. 언제 information을 downwards로 전파해야하는가?
   - inconsistence가 클때? lower level에서의 edge가 말하는 위치와 upper에서 최적화한 결과가 너무 다를때?
3. 어떻게 Sparsified graph 의 edge를 계산하는가?

![image-20191130111105433](D:\Workspace\Blog\sudo-hoon.github.io\_posts\SLAM\2019-11-30-Hierarchical-Pose-Graphs.assets\image-20191130111105433.png)

- 우선 각 local group을 optimization 한다.
  - optimization된 대표값들의 relative pose가 edge의 관측 값.
  - $x_a$를 고정 시켰을 때의  $x_b$ 와의 relative uncertainity의 reverse가 edge의 information matrix
  - $H_{[b,b]}^{-1}$ :  a번째 행,열들을 제거한 Hessian의 inverse의 b,b 번째 노드

4. 어떻게 information을 low level로 전파하는가?
   - 이해 불가.
   - 위에서 optimization 하면 대표들이 rigid body transformation할 테고 상대적인 거리로 노드들이 표현되므로 전체가 다 전파되는거 아닌가?

- initial guess가 왜 중요한가? p56 전체 다 설명듣기

### For the best possible map

- 최상위까지 optimizaiton해서 아래로 전파한 후 다시 lowest level에서 optimization하기

### consistency

- 얼마나 top level이 original input을 잘 표현하는가?
  - 각 level에서의 불확실성을 보면 됨.
    - higher level이 불확실성이 큼.
    - 그래프에서 불확실성 그림이 일치하지 않는 부분이 있다?
    - linearization때문이다.
      - 이것들도 설명 다시 듣기