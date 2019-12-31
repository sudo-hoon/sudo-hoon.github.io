---
layout: post
title: "Motion Planning"
date: "2019-12-27"
# slug: "example_content"
description: "Motion Planning"
category: 
  - Motion Planning
# tags will also be used as html meta keywords.
tags:
  - Motion Planning
  - Robot
comments: true
use_math: true
# ![](/images/posts/cpp_hot/img1.png){:.aligncenter}
# hyperlink : [name](url)
---
# Motion Planning

- A*(A star), D\*(D star), RRT(Rapidly Random Tree), RRT\* (RRT star)

- Incremental Path Planning
  - temporal consistency를 위한 temporal planning이다. 

## Graph Search

### 문제 정의 

![image-20191227071056688](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191227071056688.png)

- __vertex__ : node, 로봇이 거치는 하나의 pose

  - pose : 로봇의 x, y ,z, yaw, pitch, roll

- __edge__ : 로봇 pose와 pose 사이의 cost

  - 멀 수록, 힘든 경로일 수록 높은 값을 갖는다.
  - $w(s_i, s_j)$ : $s_i$ node와 $s_j$ node 사이의 cost

- __heuristic function, $h(s_k, s_{goal})$__ : 해당 노드에서 goal 까지의 예상되는 cost

  - __cost-to-go__ 라고 불린다.

  - map 상에서 직선 거리로 결정한다

  - > 왜 heuristic을 사용하는가?  
    > heuristic은 search를 해 나갈 방향을 지정해주는 역할을 한다.  
    > 따라서 잘 정의된 heuristic은 searching efficiency에 도움이 된다.
    >
    > 단, heuristic 계산에 너무 많은 load가 부하되면 오히려 손해가 된다.

- __cost-so-far, $g(s_{start}, s_k)$__ : 현재까지의 cost의 합

- __total cost__ = $f(s_k)=g(s_k) + h(s_k)$

- __Successor와 Predecessor__

  - __Successor__ of node s : s로부터 도달할 수 있는 모든 node
    - notation : Succ(s)
  - __Predecessor__ of node s : s에 도달할 수 있는 모든 node
    - notation : Pred(s)

### graph Representation

![image-20191227072201708](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191227072201708.png)

- robot이 움직일 수 있는 방법에 따라 Graph의 edge를 표현할 수 있다.



## Dijkstra Algorithm

- graph에서 edge가 음수가 아닌 cost를 가질 때, shortest path를 찾을 수 있는 알고리즘
- __total cost__ : $f(s_k)=g(s_k)$
  - heuristic은 고려하지 않는다.

- 상세 구현 :
  - https://mattlee.tistory.com/50 

## A\*(A star) Algorithm

### Motivation

- dijkstra Algorithm은 전체 그래프를 건드리려고 하지만 A* algorithm은 효율적으로 건드린다.
- heuristic을 통해 searching해갈 방향을 지정할 수 있다.
- Dijkstra와의 비교
  -  https://www.youtube.com/watch?v=g024lzsknDo 

### Problem

- __total cost__ : $f(s_k) = g(s_k) + h(s_k)$

### 상세 구현

-  http://www.gisdeveloper.co.kr/?p=3897 





## D\*(D star) lite Algorithm

- 그래프의 순회와 동시에 그래프의 edge cost가 변할 때, 효율적이고 반복적으로 best-first search를 하는 방법
  - best-first search : breath-first search와 달리 prior가 있고 그 prior에 기반해서 최선의 search를 하는 것.

### Motivation

- 문제 상황
  - map은 계속 update 된다.
    - 로봇이 움직이는 동안 map의 변화가 발생할 수 있다.
    - 초기의 planning이 부적절하다, re-planning이 필요하다.
  - sensor는 limited range를 갖는다.
    - 로봇이 움직이며 지도를 계속 갱신하며 목적지를 향해야할 수 있다.
  - 한정된 computation time을 갖는다.
    - 효율적인 re-planning 기법이 필요하다.

### Idea

![image-20191227073116871](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191227073116871.png)

### Incremental Search

- 초기 searching

- ![image-20191227073433987](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191227073433987.png)
  - 초기에는 complete search와 incremental search가 순회하는 노드량이 같다.
- map update후의 searching

- ![image-20191227073516665](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191227073516665.png)
  - re-planning시, incremental search의 경우가 순회하는 노드의 갯수가 훨씬 적다.
  - Incremental search의 경우에 초기 searching은 많은 방문(즉, 많은 계산)이 필요하지만 이후 map udate 이후에는 빠르게 re-planning을 할 수 있다.

## D* lite algorithm Problem definition

- 알고리즘
  - ![image-20191228214416354](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191228214416354.png)
  - 위의 과정을 이해하려면 문제에 대한 정의를 이해해야한다.

### node selection

- ![image-20191227075305036](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191227075305036.png)
- __total cost__가 작은 node가 첫 번째 우선 순위를 갖는다.
- __cost-so-far($g(s_k)$, heuristic)__ 가 작은 node가 다음 우선 순위를 갖는다.
- 위 상황에서는 $s_1, s_2$모두 total cost가 갖지만 $s_2$가 cost-so-far가 작기 때문에 $s_2$ 노드가 선택된다. 
- 이후 priority queue를 다룰 때, __key value__로 사용된다.

### reformulation

- A* algorithm에서 정의했던 path cost(g)와 heuristic(h)를 재정의한다.

- __path cost, $g(s)$__ at D* : goal node에서 부터 s 노드까지의 cost의 합
  - A* algorithm에서는 path cost를 start node에서 s node까지의 총 cost의 합으로 정의했다.
  - ![image-20191228214555383](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191228214555383.png)
    - D* 에서는 re-planning시 start node의 위치를 현재 로봇의 pose로 옮긴다. 
    - 이때, $g(s)$는 보존되지 않는다. 즉, 값이 변경된다.
    - Case1에서 Case2로 변화할 때, $g(s)$의 값은 변한다.
  - ![image-20191228215003981](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191228215003981.png)
    - 만약 위와 같이 path cost를 goal에서 부터 s까지의 weight로 정의를 하면 path cost는 보존된다.
- __heuristic, h(s)__ : start node에서 s node까지 추정되는 거리
  - 이 부분은 start node가 변경될 때마다 재 계산된다.
- 이유 : 
  - heuristic은 쉽게 re-computation이 가능하지만 path cost는 쉽게 계산하기 힘들다.

### rhs

- A*에서는 각 node에 path cost, g(s)만 저장하였다.
- D* star에서는 각 node에 rhs(s)도 저장한다.
  - rhs(s)는 g(s)의 값이 map의 변화에 영향을 받았는지를 확인하는 용도이다.
- __rhs(s)__ :
  - $0\ \ \ \ \ if\ s\ =\ s_{start}$
  - $min_{s'\in Pred(s)}(g(s')+c(s',s))\ \ \ otherwise$
    - start node는 0을 갖는다.
    - s로 향하는 모든 노드들로 만들수 있는, s node로 향하는 최소의 path cost
      - $g(s')+c(s',s)$  : s'의 path cost + s'와 s 사이의 cost
      - $g(s)$는 s node의 path cost로 goal node에서부터 현재 node까지 path의 cost를 계산한다.
      - 이미 계산되어있는 $g(s')$를 이용하는 rhs와는 다른 것 같기도 같은 것 같기도 하다?
        - 결과는 같은데 구하는 방식이 훨씬 효율적인건가?

### Update and Expanding

- __update__ : 해당 노드의 rhs 값을 계산하고 해당 노드를 priority queue에 집어넣는 것

  - priority queue의 key : <$min[g(s), rhs(s)] + h(s), min[g(s), rhs(s)]$>
    - 정확히는 <$min[g(s), rhs(s)] + h(s) +k_m, min[g(s), rhs(s)]$>
  - ![image-20191228224259157](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191228224259157.png)

- __expanding__ : priority queue에서 노드를 하나 꺼내고 해당 노드의 g 값을 재 계산하는 것

  - 이때, locally consistency에 따른 action을 취한다.

  - ![image-20191228224356677](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191228224356677.png)

  - > __왜 굳이 g node와 rhs node를 분리해서 계산하는가?__
    >
    > 1. rhs 값은 expansion 하기전에 여러번 바뀔 수 있다.
    >    - 아래에서 under consistent 상황 참고.
    >
    > 2. make sure that we have updated all successors that could lower the total cost f(s) first

  - g 값의 계산은 아래 locally inconsistency에서 다룬다.

### locally inconsistency

- edge cost에 변화가 생겼을 때, D* lite algorithm은 변화가 일어난 부분에서 부터 지역적으로 변화를 반영, 반영된 결과를 이웃 노드로 전파한다.

- __locally inconsistent__ : g(s) != rhs(s)
  - s node가 map의 변화에 영향을 받았음을 의미한다.
  - 이 경우 rhs 값을 재계산하고 해당 노드를 priority queue에 넣는다.
- __locally overconsistent__ : g(s) > rhs(s)
  - g(s)에 rhs(s)을 대입한다. 즉, g(s) = rhs(s)
  - 그리고 모든 s의 predecessors를 update한다.
- __locally underconsistent__ : g(s) < rhs(s)
  - g(s)에 infinity를 대입한다.
  - 그리고 모든 s의 predecessors와 s 자기 자신을 update한다.
    - 이때, locally consistent라면 queue에 넣지 않아도 된다.
    - 왜냐하면 이때의 g=rhs=inifinity이고 priority queue에서 항상 마지막 순위를 갖는다.
  - 이때, s node는 locally consistent 또는 locally overconsistent가 된다.
    - s node는 queue에 들어갔기 때문에 rhs 값을 다시 계산하게 되며 best rhs를 찾을 때까지 반복된다.
    - 즉, locally consistent를 찾을 때 까지.
  - 이때, locally consistent가 된다는 말은 g와 rhs가 모두 무한대가 된다는 뜻이다.
    - 즉, 해당 노드를 통해서는 goal로 갈 수 없다, obstacle이다.
- __locally consistent__ : g(s) == rhs(s)
  - 이때는 s node가 map의 변화에 영향을 안 받았거나 변화가 반영되었다는 의미이다.
  - 해당 노드는 더 이상의 연산이 필요하지 않다.
    - 즉, update하지 않는다.

### $k_m$

- robot의 위치가 움직이면, 즉, $s_{start}$가 변하면 heuristic도 변한다.
  - 최대 $h(s_{last}, s_{start})$(지난 start node에서 변경된 start node 사이의 heuristic)만큼 감소한다.
- 이때, queue에 들어가있는 node의 key 값은 변화가 반영되지 않는다.
- 변화된 heuristic을 반영하기 위해서 heuristic의 변경이 있을 때 마다 key 값을 증가시킨다.
- 즉,  __modified key__ = <$min[g(s), rhs(s)] + h(s) +k_m, min[g(s), rhs(s)]$>
  - $k_{m+1}=k_m+h(s_{last}, s_{start})$

### rhs에 대한 생각

> 왜 쓸까에 대한 고찰
>
> - s node에 s1, s2 node가 이웃한다고 생각해보자.
>   - s1에서 전파된 rhs는 s의 g보다 크다.
>   - s2에서 전파된 rhs는 s의 g보다 작다.
>   - rhs from s2가 먼저 반영되고 rhs from s1이 반영되면?
>     - 이때, rhs값을 안 구하고 바로 g 값에 대입하면?
>       - 최종 반영은 rhs from s1이 된다.
>       - 즉, 더 적은 cost를 갖는 path를 놓치게 된다.
>       - 따라서 rhs라는 것을 도입해서 g에 바로 대입하지 않아야 한다.
>         - 그렇기 때문에 g < rhs 인 상황에 좀 더 특별한 연산이 필요하다.
>
> under consistent(g < rhs)에 대한 고찰
>
> - 주변의 노드에서 under consistency가 발생하면서 s node가 update(첫 번째 enqueue)된다.
>
> - 이때, 우선순위가 왔을 때, s 노드도 under consistency가 발생하여 g에 무한대를 대입하고 update한다. (두 번째 enqueue)
> - 이때, 주변의 노드도 update를 한다.
>   - 이 과정에서 rhs값은 계속 변하게 되고 최소의 rhs값을 가질 때, 해당 노드는 priority를 갖는다.
>   - 즉, 최소의 rhs값을 가질 때, expand(dequeue)를 하게되며 g값에 최소의 rhs값을 대입하게 된다.
> - 위와 같은 과정에서 under consistency는 해당 node가 최소의 g값을 갖게 해준다.
> - 또한 위의 과정으로 인해 D* lite algorithm은 최대 2번의 en-dequeue가 사용된다.

### 알고리즘

![image-20191228214416354](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191228214416354.png)

```
1. 모든 노드의 g(s)를 unexpanded로 설정한다.
2. s_start 노드가 consistent해질 때까지 Best-first search를 한다.
3. s_start 노드를 다음 best 노드로 옮긴다.
	(즉, 로봇을 key 값이 가장 적은 이웃으로 이동한다.)
4. edge cost가 변화가 일어나면
	a. heuristic의 변화를 계산한다.
	b. 변화된 edge에 연결된 모든 node들을 update한다.
5. 2를 반복한다.
```

__1. 모든 노드의 g(s)를 unexpanded로 설정한다.__

- 모든 g 값들을 inifinity, rhs($s_{goal}$)을 0으로, rhs(나머지) = inifity로, $k_m$ 을 0으로 설정한다.
- goal node의 rhs를 계산하고 queue에 넣는다.(update)

__2. s_start 노드가 consistent해질 때까지 Best-first search를 한다.__

- locally consistent를 따져서 update, expand를 반복한다.
- 맨 처음에는 A* 와 같은 과정을 겪는다.

__3. s_start 노드를 다음 best 노드로 옮긴다.__

- $s_{start}$를 옮긴다.

__4. edge cost가 변화가 일어나면__

__4-a.  heuristic의 변화를 계산한다.__

- 변화된 start point에 대해 모든 node의 heuristic을 재계산한다.
- $k_m$도 재계산한다.

__4-b. 변화된 edge에 연결된 모든 node들을 update한다.__

- cost가 변한 edge에 연결된 모든 node들의 rhs 값을 재계산한다.
- 그 후에 priority queue에 해당 node들을 집어넣는다.

__D* lite mathematical Algorithm__

![image-20191228233634664](D:\Workspace\Blog\sudo-hoon.github.io\_posts\Motion Planning\2019-12-27-Motion-Planning.assets\image-20191228233634664.png)

- __key value of priority queue__ : <$min[g(s), rhs(s)] + h(s) +k_m, min[g(s), rhs(s)]$>

## When to Use Incremental Path Planning

- D* lite algorithm에서 어느 노드든 queue에 최대 2번 넣게 되어있다.
  - underconsistent 상황에 update를 2번하게 돼있다.
  - 그래도 변화가 생겼을 때 A*로 full search 하는 것보다는 노드를 적게 건드린다.
  - A*는 노드를 최대 1번 queue에 넣는다.
- obstacle의 변화가 start에 가까울 때는 그냥 A*로 full search하는 것이 낫다.
  - 변화가 생긴 노드로부터 goal node의 방향이 변화에 민감하기 때문에 search를 다시 할 확률이 큰데, start node에 가까운 변화가 생기면 거의 모든 노드가 변화에 민감하기 때문에 어차피 full search를 할 가능성이 크고 D*는 queue에 많으면 2번 넣기 때문에 더 느릴 것이다.
- graph에서 너무 큰 변화가 발생하면 그냥 A*로 full search 하는 것이 낫다.

- sensor를 사용하면서 맵을 넓히면서 목적지를 향해 갈 때, D* lite algorithm을 사용한다.



### Example

- https://ocw.mit.edu/courses/aeronautics-and-astronautics/16-412j-cognitive-robotics-spring-2016/lecture-notes/MIT16_412JS16_L14.pdf 
  - 108p ~ 143p





## Rapidly Random Tree

