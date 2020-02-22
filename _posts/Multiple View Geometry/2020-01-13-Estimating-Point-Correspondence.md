---
layout: post
title: "Estimating Point Correspondence"
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
# ![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/img1.png){:.aligncenter}
# hyperlink : [name](url)
---
> 본 내용은 daniel cremers의 multiple view geometry 강의를 요약 및 정리를 하기 위해서 작성되었습니다.   
> 처음 공부하는 내용이라 틀린 부분이 있을 수 있습니다. 틀린 부분을 지적해주시면 감사드립니다.

# Estimating Point Correspondence

여러 view에서 얻은 2D 카메라 이미지로부터 3D 영상을 복원하기 위해서는 여러 영상에서 같은 point라고 대응되는 점을 찾아야 한다.  
대응되는 점을 찾으면 여러 2D 이미지에서 geometric한 방법으로 3D 영상을 복원할 수 있다.  
대응 되는 포인트를 찾는 것을 Point matching이라 한다.

## Small Deformation vs Wide Baseline

point matching에서는 크게 두 가지 케이스가 있다.

### small deformation

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113211449548.png){:.aligncenter}

두 프레임 사이의 변화가 매우 적은 상황을 의미한다.   
(위 그림에서 눈에 띄는 변화를 못찾겠으면 정상이다.)    
이때는 classical optical flow estimation을 통해서 point matching을 할 수 있다.    
(전체 이미지의 변화를 보는 방법을 사용한다. 효율성을 위해서 feature를 뽑아서 하는 것이 더 빠르긴 하다.)   
__method__ : __Lucas/Kanade method__ or __Horn/Schunck method__

### wide baseline stereo

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113211318329.png){:.aligncenter}

두 프레임 사이의 변화가 매우 큰 상황을 의미한다.  
small deformation과 달리 전체 point를 dense matching하는 것은 불가능하다.  
적절한 feature point들을 뽑아서 matching하는 방법을 이용한다.



# Small Deformation & Optical Flow

- 위에서 언급한 __Lucas - Kanade method__에 대해 설명한다.  
- optical flow는 두 연속된 image에서 2D motion field이다.
  - 즉, 한 화면에서의 포인트가 다음 화면에서 어떻게 움직이는지에 대한 motion field
- __Lucas -Kanade method__는 지역적으로 이웃한 point들은 constant motion을 갖는다는 가정하에 sparse flow vector를 얻는 방식이다.
- __Horn - Schunck method__는 spatially smooth motion을 가정한 dense flow vector를 얻는 방식이다.
  - Lucas-kanade 방식과 거의 유사하다.

### Representation of Small Deformation

첫 번째 이미지에서의 포인트 x1은 다음 이미지에서 small deformation을 겪는다.  
그것에 대한 수학적인 모델링은 다음과 같다.  
rigidly moving object의 모든 점들의 움직임은 다음과 같다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113211806453.png){:.aligncenter}

여기서 $\lambda$는 scale factor이며 h는 motion model이다.   
translation model은 다음과 같다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114215714566.png){:.aligncenter}

affine transform으로 modeling한다면 다음과 같다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113212030135.png){:.aligncenter}

또는 다음과 같이 parameterization한 form으로 표현할 수 있다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113212100980.png){:.aligncenter}

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113212108139.png){:.aligncenter}

위와 같은 표현의 장점은 3x3 affine transform matrix가 갖는 9개의 parameter를 모두 추정하는 대신 6개의 parameter만 추정하면 된다.  

### Lucas-Kanade Method

두 가지 가정을 전제한다.

- __가정 1__ : __Brighteness Constancy Assumption__
  - 2D 이미지에서 한 점은 시간이 지나도(연속된 장면들에서) 밝기의 변화가 없다.
  - 한 점이 밝기 변화 없이 가만히 있는 것이 아니라 살짝 움직여도 밝기의 변화가 없다는 의미이다.
    - 실제로는 한 점이 살짝 움직이면 햇빛 방향이 약간 틀어지며 밝기의 변화가 생긴다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113212938505.png){:.aligncenter}

위 결과를 시간에 따라 미분하면 다음과 같은 미분방정식을 얻을 수 있으며 이것을 __optical flow constraint__라 한다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113213014472.png){:.aligncenter}

이때, 괄호안의 $v = \frac{dx}{dt}$는 2D 이미지에서 해당 point의 속도이다.(local flow vector)

- __가정 2 : Constant motion in a neighborhood__
  - 인접한 점들은 같은 속도로 움직인다.

인접한 점들($x'$) 역시 위의 optical flow constraint(모두 같은 속도를 갖는다.) 만족해야한다.  
이때, 인접한 점들의 범위는 해당 점을 기준으로 원형의 window($W(x)$)를 씌운다고 생각하면 된다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113214026054.png){:.aligncenter}

- __method__

__Lucas - kanade method__는 local window 내에서 위 가정들을 가장 잘 만족하는 x의 속도 v를 찾는 방법이다.  
아래와 같이 least squares error를 최소화하는 속도를 찾는다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113214105731.png){:.aligncenter}

이것을 속도 v에 대해 미분하면 다음과 같다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113214723336.png){:.aligncenter}

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113214729785.png){:.aligncenter}

만약 M이 역행렬이 존재한다면,

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113214812527.png){:.aligncenter}

이다.  
이때, 위에서 구했던 small deformation에 대한 motion model을 적용하면 다음과 같다.    
![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114215812189.png){:.aligncenter}

나중에 다루지만 frame 간의 시차가 꽤 크다면 local window상의 point들은 단순히 translation하는 것이 아니라 affine transformation할 것이다.   
affine model을 적용하면 아래와 같다.    
_어떻게 아래처럼 된건지 잘 모르겠다..._

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113214850432.png){:.aligncenter}

### Aperture Problem

위에서 언급된 $M(x)$는 __structure tensor__라고 한다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113225318699.png){:.aligncenter}

이때,

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200113214812527.png){:.aligncenter}

를 만족하기 위해서는 structure tensor가 invertible해야한다.  
즉, structure tensor가 full rank여야한다.(2D라면 rank = 2)



- 만약 하얀 화면에 검은색 칠판이 오른쪽으로 지나간다고 해보자. 
  - 이때, 검은 색 칠판의 중심에 있는 점에서 윈도우 $W(x)$를 씌우면?
    - 이때는 모든 속도 v가 가능하다.  
  - 위의 상황에서 검은 색 칠판의 오른쪽에서 윈도우 $W(x)$를 씌우면?
    - 이때는 수평 방향의 속도는 측정할 수 있지만 수직 방향의 속도는 측정할 수 없다.

위에서 제시한 상황은 모두 structure tensor가 full rank가 아닌 상황이다.  
따라서, 역행렬이 존재하지 않으므로 위 방법으로 문제를 풀 수 없다.

### Feature Tracking Algorithm

sturcture tensor가 full rank인 상황이라면 2D image에서 correspondence를 찾을 수 있다.  
연속되는 이미지에서 correspondence를 찾는, 즉, tracking algorithm에 대해 정리한다.



1. time t에서 2D image의 모든 point $x_t$에 대해 structure tensor를 계산한다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114214208272.png){:.aligncenter}

2. $detM(x)$ 가 threshold $\theta$를 넘는 point들을 체크한다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114214336591.png){:.aligncenter}

3. 위 조건을 만족하는 포인트들에서 local velocity를 계산한다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114214405202.png){:.aligncenter}

4. 이때, $b$ 는 $q$(affine)을 통해서 time t+1일 때의 point  $x_{t+1}$을 $x_t$와 matching할 수 있다.
5. 반복한다.



### Forstner Harris Stephens feature extractor

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114214803634.png){:.aligncenter}

이전 방식과 달리 structure tensor를  계산할 때, structure tensor의 윈도우에 width $\sigma$를 갖는 gaussian을 곱해서 weighted sum을 계산한다.  
window의 중심인 $x$가 가장 큰 값을 갖고 window의 경계에 가까워질 수록 낮은 값을 갖는다.    
Forstner/Harris detector는 다음과 같이 구할 수 있다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114215127090.png){:.aligncenter}

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114215150173.png){:.aligncenter}

$C(x) > \theta$ 을 만족하는 점들을 이미지의 feature로 선택한다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114215252708.png){:.aligncenter}

위 이미지와 같이 모서리 부분이 feature로 추출된다.  
논문들을 보면 Harris corner detector라고 많이 표현하던데, 나중에 내용을 보강하겠다.



# Wide Baseline Matching

frame마다 feature를 tracking하는 방식은 시간이 지날수록 오차(drift error)가 누적된다.  
이러한 오차를 제거하기 위해서는 많은 수의 frame이 지나고 첫 frame과 마지막 frame간에 matching을 함으로 써 해소할 수 있다.  
이때는 small deformation case와는 달리 이미지가 굉장히 많이 변해있다.  

- local window 내에서 translation model이 아닌 affine model을 사용해야한다.  
- illumination도 굉장히 달라지기 때문에 normalize가 필요하다.
  - ilumination에 독립적이기 위해서 local window 내에서 밝기의 변화량을 제거해주어야한다.

위 두 가지를 만족하기 위해서 제안된 방법으로 Normalized Cross Correlation 방법이 있다.

### Normalized Cross Correlation

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114220006932.png){:.aligncenter}

$\bar I_1, \bar I_2$ 은 첫번째, 두번째 이미지에서 local window내에서의 평균 밝기이다.  

- 평균 밝기를 뻄으로써 additive 밝기 변화를 제거한다.
- 밝기의 변화량으로 나눔으로 써 muliplicative 밝기 변화를 제거한다.
- h는 motion model로 affine transformation을 사용한다.
- 결과적으로, 가장 높은 NCC 값을 갖게하는 affine transformation parameter를 찾는 것이 목적이다.

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114220755825.png){:.aligncenter}

![](/images/posts/MVG/2020-01-13-Estimating-Point-Correspondence.assets/image-20200114220804244.png){:.aligncenter}

