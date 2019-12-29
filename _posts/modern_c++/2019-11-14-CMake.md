---
layout: post
title: "Modern C++ std::array, std::vector, range loop"
date: "2019-09-21"
# slug: "example_content"
description: "Modern C++ 정리"
category: 
  - Modern C++
# tags will also be used as html meta keywords.
tags:
  - Modern C++
  - C++
comments: true
use_math: true
# ![](/images/posts/cpp_hot/img1.png){:.aligncenter}
# hyperlink : [name](url)
---
# std::array

- 같은 type의 item들을 저장한다.

```cpp
#include <array>
std::array<float, 3> arr = {1.0f, 2.0f, 3.0f};
```

- 선언 : `std::array<type, size> arr`

- Operation :

  1. 접근 :  `arr[i]`

  2. 개수 확인 : `arr.size()`

  3. 첫 원소 접근 : `arr.front()`

  4. 마지막 원소 접근 : `arr.back()`

     

# std::vector

- array와 달리 dynamic하게 데이터를 저장한다.

```cpp
#include <vector>
#include <iostream>
std::vector <int> numbers = {1, 2, 3};
std::vector <string > names = {"sudo", "hoon"};

```

- 선언 : `std::vector<type> vec`

- Operation :

  1. 접근 : `vec[i]`

  2. 새 아이템 삽입 : `vec.emplace_back(value)` -> c++11 이후 권장   
                                 `vec.push_back(value)` -> 예전 사용법

  3. 첫 원소 접근 : vec.front()

  4. 마지막 원소 접근 : vec.back()

  5. 미리 벡터 크기 정의 : vec.reserve(size)  

     > 데이터를 삽입하면 동적으로 크기를 늘려야하기 떄문에 애초에 어느 정도 크기를 잡으면 데이터 삽입시 속도를 최적화할 수 있다.



## Range Loop

- array 또는 vector의 indexing을 하지 않고 각 원소에 접근할 수 있다.

```cpp
for (const auto& value : arr) {
 //  arr 안의 원소 값 하나씩 변수 value에 넣어서 중 괄호 안의 명령을 수행한다.
}
```

- 사용법 : `for (<type> <variable> : <array or vector>) `