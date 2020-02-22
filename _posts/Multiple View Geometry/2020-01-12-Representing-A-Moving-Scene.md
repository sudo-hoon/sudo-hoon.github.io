---
layout: post
title: "Representing a Moving Scene"
date: "2020-01-21"
# slug: "example_content"
description: "Multiple View Geometry by daniel cremers"
category: 
  - Multiple View Geometry
# tags will also be used as html meta keywords.
tags:
  - multi view geometry
  - vision
  - multiple view geometry
comments: true
use_math: true
# ![](/images/posts/MVG/img1.png){:.aligncenter}
# hyperlink : [name](url)
---
> 본 내용은 daniel cremers의 multiple view geometry 강의를 요약 및 정리를 하기 위해서 작성되었습니다.   
> 처음 공부하는 내용이라 틀린 부분이 있을 수 있습니다. 틀린 부분을 지적해주시면 감사드립니다.

# Camera Motion

3D scene과 카메라로 관측되는 2D projection과의 관계는 두 가지 transformation으로 표현할 수 있고 3D reconstruction의 기본은 다음 두 가지 transformation을 찾는 것이다.

### Euclidean motion or rigid-body transformation

한 프레임에서 다음 프레임으로의 카메라 모션을 의미한다.

### Projective projection

카메라에 나타나는 3D scene의 2D projection에 대한 transformation을 의미한다.



이 챕터에서는 Camera motion에 해당되는 Euclidean motion을 표기하는 방식에 대해 다룬다.



# 3D space & Rigid Body Motion

### 3D Euclidean Space

3 dimensional Euclidean space $E^3$(point)은 $X=(X_1, X_2, X_3)^T \in R^3$(coordinate) 에 의해 특정되는 모든 point를 포함한다.



### Cross Product 

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112091021641.png){:.aligncenter}

vector간의 Cross Product $\times$ 는 위와 같이 정의할 수 있다.  이때, 결과 벡터는 u와 v에 수직이다.  

### Skew-symmetric Matrices

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112091330008.png){:.aligncenter}

__정의__

skey symmetric matrix : $M=-M^T \in R^{3\times3}$

__특성__

- symmetric matrix의 형태지만 부호가 반대이며, diagonal 성분이 모두 0이다.

- vector u에 수직인 space를 span한다.
  - 즉, $M\cdot v=0 $
- 즉, 평면을 span하므로 위 matrix의 rank는 2이다.

### isomorphism operation $\hat{}$

모든 3x3 skew-symmetric matrix들의 space는 $so(3)$이다.  
이때, $\hat{}$ operator는 vector $u \in R^3$에서 $so(3)$로 mapping한다.

- isomorphism이란?

두개의 group 사이의 모든 연산과 관계를 보존하는 mapping하며 전단사(bijection) 속성을 갖는다.  
즉, inverse가 존재한다.

- 이것이 왜 필요한가?

나중에 추가하기.

### Rigid-body Motion

rigid body motion은 두 벡터의 norm과 cross product를 보존하는 transformation(motion $g_t$)이다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112092516462.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112092459816.png){:.aligncenter}

- norm이 보존되면 inner product 역시 보존된다. (polarization identity)
- 즉, norm. inner product, cross product가 보존된다.
- 따라서, triple product가 보존된다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112092710533.png){:.aligncenter}

위와 같은 triple product이 보존된다는 것은 __volume-preserving__하다는 것을 의미한다.  
3D 구조물이 있을 때, rigid body motion을 하면 그 구조물의 부피는 변하지 않는다.



### Rigid-body Motion의 수학적 표현법

- __SE3__

$SO(3)=\{R \in \mathbb R^{3\times 3}\|R^TR=I, det(R)=+1\}$

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112144913014.png){:.aligncenter}

$SE(3)$ 로 rigid-body motion을 표현할 수 있다.

- __se(3)__

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112144952307.png){:.aligncenter}

$se(3)$로 rigid-body motion을 표현할 수 있다.

### lie group and lie Algebra

> 왜 lie algebra를 사용하는가?
>
> 물체의 rigid-body motion을 표현할 때, SE3를 이용해서 표현한다.   
> 물체가 어떻게 움직였나를 추론하려면 SE3의 각 값을 추론해야한다.  
> 이때, 회전에 해당되는 R(YPR, yaw pitch roll)은 3의 자유도를 갖는다.  
> 하지만 위 3의 자유도를 계산하기 위해서는 9개의 파라미터(3x3 matrix)를 추론해야한다.  
> 이때, lie algebra를 이용하면 더 낮은 파라미터 개수를 이용해서 YPR을 추론할 수 있다.
>
> 출처 : http://karnikram.info/blog/lie/ 



## Lie group and Lie Algebra

### Infinitesimal rotation의 표현

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112145324156.png){:.aligncenter}

$X_{orig}$에서 $X_{trans}$로 물체의 회전은 위와 같이 SO(3)를 이용하여 표현할 수 있다.  
이때, $R(t)R(t)^T=I$이므로

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112145425755.png){:.aligncenter}

이다.  
$\dot RR^T$는 skew-symmetric matrix이므로

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112145555471.png){:.aligncenter}

로 포현할 수 있다. 이때, rotation matrix를 t = 0에 대해서 first-order approximation을 하면

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112145721229.png){:.aligncenter}

을 얻을 수 있다.($R(dt)=R(0)+dR$ : first-order approximation of a rotation at t = 0)  
즉, 매우 극소 시간동안의 회전 $R \in SO(3)$은 위와 같이 __so(3), skew-symmetric matrix__를 이용하여 표현할 수 있다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112145950206.png){:.aligncenter}

이때, rotation group $SO(3)$는 __Lie group__이라 하며, space $so(3)$는 __Lie algebra__라 한다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112150137106.png){:.aligncenter}

### Exponential Map

so(3)를 여차저차 구했다고하자. 이것이 의미가 있으려면 SO(3)로 변환해서 Rotation matrix를 구해야한다.   
위에서 $\dot R(t) = \hat w R(t), R(0) = I$ 라는  두 가지 관계식을 얻었고 이러한 미분 방정식의 solution은 다음과 같다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112150347597.png){:.aligncenter}

- 이것이 무엇을 의미하는가?
  1. $so(3)$로 부터 $SO(3)$를 얻을 수 있다. 즉, lie algebra로 부터 lie group을 얻을 수 있다.
  2. $w \in \mathbb R^3$에서 axis $w$에 대한 angle t 만큼의 회전을 의미한다. (||w|| = 1일 때)
     - 즉, 전체 회전량은 $\hat v = \hat w t$
  3. logarithm을 통해서 $SO(3)$로 부터 $so(3)$를 얻을 수 있다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112151114729.png){:.aligncenter}

### Rigid-body motion SE(3)의 표현

위에서 언급한 것과 같이 rigid-body motion은 SE3로 표현할 수 있다.   
그렇다면 lie algebra로 se(3)는 어떻게 표현하는가?

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112151333485.png){:.aligncenter}

continuous family of rigid body transformation을 g(t)라 하자.  
이때, 위의 수식을 미분하면 다음을 얻을 수 있다. 

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112151420578.png){:.aligncenter}

위 수식에 skew-symmetric matrix와 $v(t)=\dot T(t)- \hat w(t)T(t)$를 적용하면

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112151517856.png){:.aligncenter}

로 표현할 수 있다. 위 수식의 오른쪽에 g(t)를 곱하면 아래를 얻을 수 있다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112151544660.png){:.aligncenter}

즉, 미소의 rigid body motion 변화는 4x4 matrix $\hat \xi$를 통해 얻을 수 있다.     
이때, $\hat \xi$를 __twist__라 부른다.    
($\hat \xi$는 curve g(t)의 tangent vector라고도 볼 수 있다.)  
정리하면

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112152017320.png){:.aligncenter}

__twist__

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112153510225.png){:.aligncenter}

__twist coordinate__

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112153523085.png){:.aligncenter}

### se(3)의 Exponential map

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112152203662.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112152211804.png){:.aligncenter}

# Camera motion의 표현

t1에서의 frame과 t2에서의 frame 사이의 한 coordinate point의 motion은 아래와 같이 표현할 수 있다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112152517197.png){:.aligncenter}

이때, 아래와 같이 t1에서의 frame과 t3에서의 frame 사이의 motion을 표현할 수 있다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112152557015.png){:.aligncenter}

반대로도 표현할 수 있어야하므로 아래의 관계들이 만족된다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112152659688.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112152623825.png){:.aligncenter}

# Velocity Transformation

위 관계에서 t1을 고정하고 한 coordinate point의 motion을 보자.  
이때, 그 point의 속도는 미분을 통해 아래와 같이 표현된다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112152924091.png){:.aligncenter}

이때,  se(3)를 이용하여 속도는 다음과 같이 얻을 수 있다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112153035732.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112153101930.png){:.aligncenter}

또는

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112153124157.png){:.aligncenter}

이때, $\hat V(t)$는 camera frame에서 보는 world frame의 상대 속도를 나타낸다.

# Adjoint Map

위는 관찰자가 고정되어 있을 때, 즉, world frame에서 보는 frame의 속도이다.  
이것을 확장해서 frame들 사이에서 생기는 상대적인 frame 속도를 얻어보자.  
frame Y와 X가 있을 때, 둘 사이의 transformation을 $Y(t)=g_{xy}X(t)$라 하자.   
이때, 속도는 다음과 같이 표현된다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112154046953.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112154151295.png){:.aligncenter}

즉, $\hat V \in se(3)$를 또 다른 $se(3)$ group인 $\hat V_y$에 mapping하는 것을 __adjoint map__이라 한다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112154325949.png){:.aligncenter}

# Euler Angles

익히 알고 있는 오일러 앵글은 lie algebra로 표현하면 다음과 같다.

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112155000921.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112155013614.png){:.aligncenter}

3개의 orthonormal한 축, $w_1,w_2,w_3$에 대해서 각 축에 대한 회전을 나누어서 se(3)로 표현할 수 있다.  
이때, 각 회전축에 대한 회전량인 $\beta_1,\beta_2,\beta_3$를 __Euler angles__라 한다.



# 요약

![](/images/posts/MVG/2020-01-12-Representing-A-Moving-Scene.assets\image-20200112152432549.png){:.aligncenter}

- 3D motion은 SE(3)로 표현할 수 있다.
- Homogeneous coordinate에서 linear transformation으로 표현할 수 있다.
  - 하지만 roll pitch yaw의 관점에서는 linear하지 않다.
- lie algebra se(3)로 lie group SE(3)를 얻을 수 있다.
  - se(3)가 파라미터 수가 훨씬 적고 linear한 matrix를 갖는다.
- se(3)를 통해 frame의 속도를 구할 수 있다.
- adjoint map을 통해 se(3)로 표현된 frame을 다른 frame으로 변환할 수 있다.