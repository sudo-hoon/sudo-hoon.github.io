---
layout: post
title: "Reconstruction from Two Views"
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

# Reconstruction from Two Views

지난 글에서 두 이미지간의 points correspondence를 찾는 방법에 대해 이야기하였다.  
이번에는 두 이미지에서 points correspondence를 알 때, 카메라의 위치와 해당 포인트들의 위치를 추정하는 방법에 대해 다룬다. 

- Reconstruction Problem

![](/images/posts/MVG/2020-01-26-.assets\image-20200126184848597.png){:.aligncenter}

Reconstruction problem은 6 Camera parameter(R, T)와 3D points $X_j$를 동시에 추정하는 문제이다.  
해당 문제의 error function은 위와 같이 세울 수 있으며 이것을 최소화하는 것이 위 문제의 목적이다.  
위의 optimization problem은 __bundle adjustment__라고도 불리며 어려운 optimization 문제이다.    

계속 진행하기 전에 3D Reconstruction을 하기 위한 3 가지 assumption이 있다.

- 두 이미지 사이에서 corresponding points set이 given이다.
- scene이 static이다. 즉, 카메라가 움직이는 동안 3D points들은 움직이지 않는다.
- intrinsic camera parameters들을 알고있다.

위 3가지 assumption을 갖고 3D reconstruction은 다음과 같이 진행된다.

1. corresponding points set들을 이용하여 __Camera Pose__를 추정한다.
2. __triangulation__을 통해 모든 __corresponding points의 3D location__을 추정한다.
3. __bundle adjustment__를 통해 위에서 얻은 Camera pose와 points들의 3d location을 optimization한다.

Camera pose와 point들의 3d location이 독립적으로 추정된 후 두 결과를 동시에 최적화한다.

# Camera Pose estimation

글 맨 위의 수식에서 보이듯이 3D reconstruction problem에서 Camera Pose와 point location을 동시에 푸는 것은 어렵다.   
하지만 특정한 constraint를 걸어준다면 3d point를 모르더라도 Camera Pose를 구할 수 있다.

## Epipole, Epipolar line, Epipolar Plane

![](/images/posts/MVG/2020-01-26-.assets\image-20200126184802600.png){:.aligncenter}

| 3D point                               | X               |
| -------------------------------------- | --------------- |
| __image plane위에 projection된 point__ | __x1, x2__      |
| __image plane의 optical center__       | __o1, o2__      |
| __epipole__                            | __e1, e2__      |
| __epipolar plane__                     | __(o1, o2, X)__ |
| __epipolar line__                      | __l1, l2__      |

- epipole : 두 이미지에서 optical center간에 직선을 연결했을 때, image plane와 교차하는 점.
- epipolar plane : optical center들과 하나의 3d point, 즉 총 3개의 점으로 형성되는 평면
  - 한 3D point와 2개의 image plane이 있을 때, 하나의 __epipolar plane__이 생긴다.  
- epipolar line : image plane 상에 projection된 point와 epipole을 잇는 직선

- epipolar라는 것이 왜 중요한가?

3D point X에 대해서 첫 번째 image plane에 정사영된 $x_1$이라는 값을 얻었고 두 번째 image plane과의 관계인 R,T를 모두 안다고 하자. 하지만 카메라를 통해서 X와 $x_1$간의 거리(depth)인 $\lambda$값은 알 수 없다.  
만약, X를 첫 번째 image plane에 매우 가깝다고 해보자.(즉, $\lambda$ ~ 0, X ~ $x_1$)  
이때, 두 번째 image plane에 projection된 point $x_2$는 거의 $e_2$일 것이다.   
$\lambda$값을 조금씩 올린다면 $x_2$는 epipolar line $l_2$위에서 움직일 것이다.  

- __R, T를 알고 $\lambda$를 모른다면 $x_2$의 위치를 unique하게 결정할 수 없다.__ 
- __하지만 epipolar line $l_2$는 unique하게 결정할 수 있다.__

실제 상황에서 알고 있는 값은 $x_1, x_2, e_1, e_2, l_1, l_2$이며 모르는 값은 $R, T, \lambda$ ($\lambda x = X$)​이다.  

- __$x_1$과 $x_2$를 알지만 $\lambda$를 모르기 때문에 두 관계를 통해 unique한 $R, T$를 알 수는 없다.__
- __하지만 $l_1$과 $l_2$의 관계를 통해 $\lambda$를 모르더라도 unique한 R, T를 결정할 수 있다.__

__즉, epipolar line간의 관계를 통해서 3D point인 X값 없이도 두 image plane간의 geometry를 결정할 수 있다!__

- 이때, $l_1$와 $l_2$간의 관계를 나타내는 matrix가 __fundumental matrix__와  __Essential matrix__이다.

## Epipolar constraint and Eight Point Algorithm

위에서 다룬 내용을 수식적으로 보자.  
Camera intrinsic parameter를 알기 때문에(K=1) 3D point와 2D image point간의 관계는 아래와 같다. 

![](/images/posts/MVG/2020-01-26-.assets\image-20200126193211277.png){:.aligncenter}

왼쪽의 수식을 오른쪽에 대입하고 skew symmetric matrix($\hat{T}$)를 양변에 곱하면 아래와 같다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126193333721.png){:.aligncenter}

![](/images/posts/MVG/2020-01-26-.assets\image-20200126193350525.png){:.aligncenter}

양 변에 $x_2^T$를 곱하면 아래와 같다. ($\hat{T}x_2 = T \times x_2$)

![](/images/posts/MVG/2020-01-26-.assets\image-20200126193651750.png){:.aligncenter}

위 수식을 __Epipolar Constraint__라 한다.  
위에서 말했듯이, epipolar relation을 통해 depth $\lambda$, 즉, 3D point X를 모르더라도 image plane 상에 정사영된 두 점 $x_1, x_2$간의 관계식을 얻을 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126194006168.png){:.aligncenter}

이때, 위 matrix를 __Essential Matrix__라 한다.

### Properties of Essential Matrix

![](/images/posts/MVG/2020-01-26-.assets\image-20200126204806539.png){:.aligncenter}

모든 essential matrix로 구성되는 space를 __Essential Space__라 한다.  

- Essential matrix의 Singual Value Decomposition의 속성
  - 2개의 Singular value는 같은 값을 갖고 하나는 0이다.
  - 즉, rank가 2이다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126204925418.png){:.aligncenter}

![](/images/posts/MVG/2020-01-26-.assets\image-20200126204934065.png){:.aligncenter}

- Essential Matrix로 부터 relative pose R, T를 구할 수 있으며 아래와 같이 2개의 가능한 solution이 존재한다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126204954887.png){:.aligncenter}

$R_z(\theta)=(cos(\theta)\ sin(\theta)\ 0;\ -sin(\theta)\ cos(\theta)\ 0;\ 0\ 0 \ 0)$

두 솔루션 중에서 한 solution만 positive depth를 주기 때문에 적절한 solution을 찾을 수 있다.

### Essential Matrix를 구하는 방법

1. Epipolar Constraint를 통해서 Essential matrix를 구하고 그것을 Essential Space에 projection한다.
   - 가장 큰 두개의 Singular value의 평균 값으로 2개의 singular value를 설정하고 가장 작은 값은 0으로 설정한다. 
2. essential space에서 epipolar constraint를 non-linear constrained optimization을 한다.

## Eight Point Algorithm

__Essential Matrix__만 구한다면 3D point location을 모르더라도 Camera Pose(R, T)를 구할 수 있다는 것을 알게되었다. Essential Matrix를 구하는 가장 유명한 알고리즘인 Eight Point Algorithm에 대해 알아보자.

### Eight Point Linear Algorithm

![](/images/posts/MVG/2020-01-26-.assets\image-20200126210200178.png){:.aligncenter}

- $E^s$ : __stacked Essential Matrix__, Essential Matrix를 vector화 한 것.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126210305487.png){:.aligncenter}

- $a=x_1\otimes x_2, \otimes $: kronecker product​

위 두 식을 epipolar constraint에 적용하고 n개의 point에 대해 적용하면 아래와 같은 linear system을 얻을 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126210449818.png){:.aligncenter}

![](/images/posts/MVG/2020-01-26-.assets\image-20200126210531756.png){:.aligncenter}

위 수식에서 9개의 unknown을 구하기 위해서는 9개의 수식, 즉, $a^i, n=9$가 필요할 것이다.  
하지만 $E^s$의 scale은 정확한 값을 알 필요 없다. ($x_2^TEx_1=0$ 이므로) 따라서 scale을 무시한다면 n = 8로도 모든 unknown을 구할 수 있다. 
즉, 최소 8개의 point로  Essential matrix를 구할 수 있으며 이 방법을 __Eight Point Linear Algorithm__이라 한다. 
이때, scale을 알 수 없으므로 sign 역시 알 수 없다. 따라서 2개의 가능한 solution을 얻게된다.  
게다가, Essential matrix로 부터 2개의 가능한 (R, T) set을 구할 수 있으므로 총 4개의 solution을 얻게된다.  
하지만 실제로 값을 구해보면 의미있는 값은 하나밖에 없으므로 쉽게 하나의 solution을 고를 수 있다.  
      

위 과정을 통해 얻은 Essential matrix는 완벽히 Essential space에 존재하지 않기 때문에 Essential space로 projection해주어야한다. 다음과 같은 방법으로 최종적인 Essential matrix를 구할 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126214458503.png){:.aligncenter}

어차피 scale indepdent하므로 $\sigma = 1$하면 된다.

- 알고리즘
  1. Linear System $\chi E^s=0$을 풀어서 Essential matrix를 구한다.
     - $SVD(\chi)=U\Sigma V^T$에서 V의 9번째 vector가 $||\chi E^s||$를 최소화하는 solution이다.
  2. Essential Space에 projection한다.
     - $E=Udiag\{\sigma_1, \sigma_2, \sigma_3\}V^T$에서 $\sigma_1 = 1, \sigma_2 = 1, \sigma_3 = 0$으로 설정한다.
  3. R, T를 계산한다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126215514103.png){:.aligncenter}

![](/images/posts/MVG/2020-01-26-.assets\image-20200126215530716.png){:.aligncenter}

### Lower Point Algorithm

위의 결과에서 8개의 point를 갖고 Essential Matrix를 구하였다.  
그런데, Essential Matrix는 3의 자유도를 갖는 R과 2의 자유도를 갖는 T(스케일을 무시하므로)로 구성되므로 실제로는 5의 Rank를 갖는다.   

> 5 point algorithm도 있지만 8 point algorithm이 더 대중적으로 사용되는 듯 하다.    
> 그 이유는 나중에 보강

### Fundamental Matrix

지금까지 Camera intrinsic parameter K = 1이라는 가정하에 Epipolar Constraint를 구하였다.  
이것을 Uncalibrated case로 확장할 수 있다.  
image plane 상에서 uncalibrated image point는 $x$, calibrated point는 $x'$라 하자.  
그렇다면 $x=K^{-1}x'$와 같으므로 아래와 같은 Epipolar constraint를 구할 수 있다.  

![](/images/posts/MVG/2020-01-26-.assets\image-20200126232338414.png){:.aligncenter}

![](/images/posts/MVG/2020-01-26-.assets\image-20200126232357607.png){:.aligncenter}

![](/images/posts/MVG/2020-01-26-.assets\image-20200126232409114.png){:.aligncenter}

이때, $F$를 __Fundamental Matrix__라 하며 invertible matrix는 항상 full rank이므로 rank에 영향을 주지 않는다.  
따라서, Essential Matrix를 구하던 방식과 같은 식으로 Fundamental Matrix를 구할 수 있다.  
하지만 ambiguity가 더 크게 발생한다.

### Degeneration

1. Translation이 없는 경우

   - T = 0이 되어서 E = 0이 되기 때문에 알고리즘을 적용할 수 없다.

   - 하지만 노이즈 때문에 이 경우는 잘 없다고 한다.

   - > 그래도 제자리에서 회전만하면 성능이 떨어지는 것 같다.

2. 모든 Point들이 한 plane에 존재하는 경우

   - $\chi$ matrix의 vector들이 not independent해서 ambiguity가 생긴다고 한다.

   - > 이 부분도 자세한 내용이 보강이 필요하다.

   - 이때, 추가적인 constrained가 생기므로 Essential matrix는 4의 Rank를 갖게된다.

     - 즉, 4개의 Point로도 Essential Matrix를 구할 수 있다.

## Four Point Algorithm

모든 점들이 한 평면위에 존재할 때, 4개의 Point만으로도 Essential Matrix를 구할 수 있다.  
$X_1 \in R^3$가 첫 번째 frame에서 보는 한 평면 위에 존재하는 3D point의 location이고 $N \in S^2$가 그 평면의 normal vector라 하자.  
이때, 그 normal vector와 $X_1$간의 내적은 거리 d(카메라 원점과 points plane간의 거리)로 나타낼 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126221459754.png){:.aligncenter}

두 번째 프레임에도 똑같이 적용한다면 아래와 같이 수식을 전개할 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126225744640.png){:.aligncenter}

이때, $H=R+\frac{1}{d}TN^T$를 __Homography Matrix__라 한다.  
위 결과를 2D image plane에 projection하면 아래와 같다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126225836540.png){:.aligncenter}

물결 ~는 scale을 했을 때, 동일하다는 것을 의미하며 위 표현을 __planar homography__라 한다.  
Homography matrix는 camera parameter와 plane parameter에 의존한다.  
위 수식에 skey symmetric matrix $\hat{x_2}$를 곱하면 아래와 같은 결과를 얻을 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126230018183.png){:.aligncenter}

위 수식을 __Planar Epipolar Constraint or Planar Homography Constraint__라 한다.  

이후의 과정은 8 point algorithm과 거의 유사하다.    
Stacked Homography matrix를 구하고

![](/images/posts/MVG/2020-01-26-.assets\image-20200126230240791.png){:.aligncenter}

kronecker product를 이용해서 $a$ matrix를 구해서 위 수식에 적용하면 

![](/images/posts/MVG/2020-01-26-.assets\image-20200126230310379.png){:.aligncenter}

아래와 같은 결과를 얻을 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126230225851.png){:.aligncenter}

이때, 주의해서 볼 점은 Essential matrix를 구할 때의 $a$ 는 9D vector였지만 여기서는 $a$가 9 x 3 Matrix이다.  
이것을 여러 point에 대해 적용하면 아래와 같은 수식을 얻을 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126230536532.png){:.aligncenter}

![](/images/posts/MVG/2020-01-26-.assets\image-20200126230544066.png){:.aligncenter}

Essential Matrix를 구할때와 같이 Homography matrix 역시도 scale을 제외한다면 8의 rank를 갖는다.    
그런데, 2개의 point pair {$x_1^j, x_2^j$}로 구한 a matrix는 2의 rank를 갖는다.  
따라서, 2개의 point pair당 2의 rank를 얻으므로 총 4개의 point(n = 4)로 구성된 $\chi$는 8의 rank를 갖게 되므로 solution을 구할 수 있게된다.  
즉, 4개의 point로도 Homography matrix를 충분히 구할 수 있다.

- 알고리즘
  1. point pair들에 대해 $\chi$를 계산한다.
  2. Singular Valude Decomposition으로 $H^s$를 계산한다.
  3. $H=R+\frac{1}{d}TN^T$를 통해 homography matrix를 계산한다.

위 결과를 보면 $R$, $N$, $\frac{1}{d}T$를 구할 수 있다. Rotation Matrix, 3D point plane normal vector는 정확히 구할 수 있지만(크기에 대한 constraint가 있으므로) Translation vector의 경우에는 d 값을 알 수 없으므로 Translation vector의 scale을 구할 수 없다.

- Essential Matrix도 마찬가지.

Essential Matrix와는 아래의 관계를 갖는다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126232021884.png){:.aligncenter}

# Structure Reconstruction

지금까지의 과정을 통해서 Camera Pose를 구하였다.  
이제는 두번째 step인 3D point를 찾는 과정에 대해 다룬다.  
j번째 point에 대해서 아래와 같이 표현할 수 있다. 

![](/images/posts/MVG/2020-01-26-.assets\image-20200126232957926.png){:.aligncenter}

이때, R, T는 구했기 때문에 알지만 scale인 $\lambda$값 들은 알지 못한다.  
이때, skew symmetric matrix인 $\hat{x_2^j}$를 양변에 곱하면 아래와 같다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126233059032.png){:.aligncenter}

이 결과를 통해 아래와 같은 n linear system을 얻을 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126233143467.png){:.aligncenter}

이것을 모든 point들에 대해 확장하면,

![](/images/posts/MVG/2020-01-26-.assets\image-20200126233204573.png){:.aligncenter}

![](/images/posts/MVG/2020-01-26-.assets\image-20200126233227350.png){:.aligncenter}

![](/images/posts/MVG/2020-01-26-.assets\image-20200126233234679.png){:.aligncenter}

와 같다.  
위 linear system은 $M^TM$의 가장 작은 eigen value를 갖는 eigen vector가 solution이 된다.  
하지만 위의 solution 또한 global scale은 구할 수 없다.(n+1개의 unknown에 대해 n개의 식이 있으므로)  
따라서 ambiguity가 존재하게 된다.

- 이때, 카메라간의 baseline이 known인 것을 이용하여 global scale을 구할 수 있다.

# Bundle Adjustment

지금까지 closed form으로 Camera Pose (R, T)와 3D point location을 구하였다.  
하지만 현실은 노이즈가 있고 위에서 노이즈를  고려하지 않았기 때문에 부정확한 결과를 얻을 수 밖에 없다.  
노이즈를 고려해서 최대한 정확한 결과를 얻기 위해서는 최종적으로 optimization과정이 필수이다.  
노이즈를 고려했을 때, reprojection error는 아래와 같이 표현할 수 있다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126233947420.png){:.aligncenter}

이때, $\tilde{x_i}^j$는 image plane에서 관측된 point이고 $\pi (X_j)$는 3D point를 image에 projection한 결과이다.  
즉, 실제 측정된 image plane에서의 결과와 closed form으로 구한 3D point를 다시 image plane에 reprojection한 결과간의 오차를 측정한 것이다.  
이것을 2개 이상의 View에 대해 확장하면 아래와 같고 non-convex function을 갖는다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126234403596.png){:.aligncenter}

이때, $\theta_{ij}$는 i 번째 image에서 j 번째 point가 visible하면 1, 아니면 0이다.

다시 2 View (stereo case)로 돌아오면 아래와 같다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126234459974.png){:.aligncenter}

이때, 아래의 constrained가 존재한다.

![](/images/posts/MVG/2020-01-26-.assets\image-20200126234546269.png){:.aligncenter}

- constrained optimization은 lagrange multiplier 를 통해 푼다. 
- non-linear optimization을 통해서 unconstrained optimization을 할 수 있다.
  - 이 방법이 더 쉽기 때문에 일반적으로 많이 사용된다.

