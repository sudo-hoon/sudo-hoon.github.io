---
layout: post
title: "Perspcetive Projection"
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

### 1. Euclidean motion or rigid-body transformation

한 프레임에서 다음 프레임으로의 카메라 모션을 의미한다.

### 2. Projective projection

카메라에 나타나는 3D scene의 2D projection에 대한 transformation을 의미한다.



- 이 챕터에서는 Projective Projection에 대해서 다룬다.

# Perspcetive Projection

### Perspective Projection?

카메라를 통해 3D scene을 보았을 때, Camera에 projection된 2D scene을 의미한다.

## Mathematics of Perspective Projection
![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112161317375.png){:.aligncenter}
위의 그림에서 대문자는 lens를 통과하기 전의 3D scene에서의 값들을 의미하고 소문자는 lens를 통과한 후의 2D image와 관련된 값들이다.          
아래 그림은 위 그림과 image plane의 위치만 다를 뿐 원리는 같다.
![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\pinhole1.png){:.aligncenter}

__principal point__ : 카메라의 중심, 즉, 렌즈 중심(B)에서 이미지 센서(image plane)에 내린 수선의 발의 image plane에서의 좌표  렌즈와 이미지 센서가 수평이 아니거나 translation이 있으면 오차가 발생할 수 있다.  
__optical axis__ : z축, 렌즈에 수직인 축    
__focal point__ : lens의 초점 거리, 이미지 센서와 렌즈 사이의 거리  
(단위 : 픽셀 사이즈, 만약 f = 500이라면, 픽셀 500개의 거리라는 뜻)
![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112161508180.png){:.aligncenter}

이때, 닮은 꼴을 이용하면 위와 같이 Projection $\pi$에 대해서 정의할 수 있다.  
$\pi$는 3D 에서 2D로 projection해주는 operation을 하며 소문자와 대문자를 주의하자.  
Z를 양 변에 곱하고 Homogeneous coordinate로 표현하면 아래와 같이 정리할 수 있다.
![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112212622101.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112213212139.png){:.aligncenter}

이때, $\Pi_0$ 는 __standard projection matrix__라 한다.  
만약, point가 렌즈로 부터 멀다면 $Z=\lambda$로 constant라 가정할 수 있다.
![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112213441266.png){:.aligncenter}

이때, $\bold X$는 camera coordinate에서 바라보는 3D scene에서의 한 점이다.  
이것을 world coordinate $X_0$에서 바라보는 점으로 변환하면 다음과 같다.
![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112214016380.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112214040503.png){:.aligncenter}

지금까지 얻은 $\bold x$는 모두 image plane상의 한 점이다.   
컴퓨터에서 image를 처리할 때는 pixel 단위로 데이터를 처리하므로 pixel coordinate로 전환이 필요하다.  
이때, transformation matrix를 구할 때, 고려해야할 점들이 몇 가지 있다. 

1. image coordinate의 중심과 pixel coordinate의 중심은 다르다.  
   아래 그림과 같이 다른 중심점을 갖기 때문에 __offset $o_x, o_y$__가 필요하다.  
2. image coordinate의 단위와 pixel coordinate의 단위를 맞춰준다.  
   __scaling $s_x, s_y$__가 필요하다.
3. 픽셀로 변환한 이미지가 사각형이 아닌 마름모와 같이 찌그러진 상태일 수 있다.  
   이때, __skew factor $s_{\theta}$__ 를 추가하여 픽셀을 사각형 형태로 유지시킨다.

위의 파라미터을 얻는 것을 camera calibration이라하며 3D scene의 point 를 image에 projection할 때, 모두 적용된다.

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112224232165.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112214655485.png){:.aligncenter}

이때, $K=K_sK_f$를 __intrinsic parameter matrix__라 한다.  
(두 값 모두 카메라 내부에서 얻는 파라미터이다.)

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112214849925.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112214938648.png){:.aligncenter}

최종 결과는 다음과 같다.

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112215008117.png){:.aligncenter}

$\Pi = K\Pi_0g$ 는 __general projection matrix__라 하며 World coordinate에서의 좌표를 camera image에서의 좌표로 변환한다.  
$\lambda$를 나눠서 정리하면 아래의 최종 이미지 coordinate에서의 좌표를 얻을 수 있다.

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112215130704.png){:.aligncenter}

| 좌표                 | operator       | 변환된 좌표          |
| -------------------- | -------------- | -------------------- |
| 4D World coordinate  | $g \in SE(3)$, | 4D Camera coordinate |
| 4D Camera coordinate | $K_f\Pi_0$     | 3D image coordinate  |
| 3D image coordinate  | $K_s$          | 3D pixel coordinate  |



> 관련 자료 :
>
> 1.  https://github.com/windowsub0406/KITTI_Tutorial/blob/master/velo2cam_projection_detail.ipynb 
>    - 매우 매우 좋은 그림이 있다.
> 2.  https://darkpgmr.tistory.com/32 
>    - 다크프로그래머

# Radial Distortion

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112220447490.png){:.aligncenter}

위 그림과 같이 focal length에 따라 optical axis에서 멀어질수록 이미지는 왜곡된다.  
정확한 영상 인식을 위해서는 위와 같은 __radial distortion__을 제거해야한다.  
이 역시도 parameterization을 통해 보상을 해주어야 한다.  
이와 관련된 두 가지 모델이 있다.

1. ![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112220759341.png){:.aligncenter}  
   이때, $\bold x_d=(x_d,y_d)$는 distorted point이고 $r^2=x_d^2+y_d^2$이다.  

2. ![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112220922784.png){:.aligncenter}  
   $r=|\bold x_d -c|$이며 c는 distortion points의 center이다.  
   이때, f(r)에서의 parameter들을 쉽게 구하는 방법은 두 그림에서 직선이어야 할 부분들(위 그림에서 선반은 꼭 직선이어야한다.) 을 비교해서 파라미터를 구할 수 있다.

# Preimage and Coimage

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112230125312.png){:.aligncenter}

위 그림에서 line L은 image plane에 projection된다.  
그런데,원점과 line L을 잇는 직선(예를 들어 $x_0 -> p_0$)상의 모든 점들은 image plane의 같은 곳에 projection 된다.  
즉, image plane상의 한 점은 수 많은 3D scene의 점에 대응된다.  
(__equivalence class of points__ : 위와 같이 scalar factor만 다른 점들의 집합  
__equivalence :  $\bold X \~ \bold Y$ __)

### image

3D coordinate에서 points가 image plane으로 projection된 points   
(정확히는 위 그림에서 원점에서 image plane의 투영된 image를 향하는 vector set)

### preimage

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112230411631.png){:.aligncenter}

image plane에 투영된 선 또는 점을 만들 수 있는 모든 3d point들을 의미한다.  
위의 예에서는 P를 둘러싸고 있는 영역을 의미한다.  
point에 대한 preimage는 line이며 line에 대한 preimage는 plane이다.  
image(vector set)이 preimage를 span한다.

### coimage

 ![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112230705589.png){:.aligncenter}

preimage의 orthogonal complement를 의미한다.  
만약, point의 coimage라면 plane이다. (point의 preimage(line)에 대해 수직인 평면)  
line의 coimage는 line이다.(line의 preimage(plane)에 대해 수직인 직선)

### image, preimage, coimage의 관계

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112231039629.png){:.aligncenter}



## Spherical Perspective Projection

지금까지 소개한 Perspective projection model은 plane surface에 image가 투영된다는 가정을 한다.  
이와 달리 Spherical perspective projection은 unit sphere에 image가 투영된다는 가정을 한다.

__spherical projection $\pi_s$__ 는 다음과 같이 정의된다.

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112220144377.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112220326155.png){:.aligncenter}

![](/images/posts/MVG/2020-01-12-Perspcetive-Projection.assets\image-20200112220337622.png){:.aligncenter}