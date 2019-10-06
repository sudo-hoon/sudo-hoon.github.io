---
layout: post
title: "C++ Class1"
date: "2019-09-21"
# slug: "example_content"
description: "C++ 기본적인 내용 정리"
category: 
  - Cpp
# tags will also be used as html meta keywords.
tags:
  - Cpp
  - study
comments: true
use_math: true
# ![](/images/posts/cpp_hot/img1.png){:.aligncenter}
# hyperlink : [name](url)
---
> 본 게시글은 윤성우님의 C++ 열혈 강의를 요약 정리한 글 입니다.   
> 틀린 내용이 있다면 지적 부탁드립니다.

# C++ Class1

----

- C++에서 클래스에 대한 기본적인 내용을 정리하였다.

## 1. C++의 구조체

```cpp
struct Car{
    char ID[10];
    int num;
    
    void CarFunc(){
        cout<<ID<<endl;
    }
    
    void CarFunctwo(int a);
};

int main(){
    Car myCar={"sonata", 100};
    Car.num = 10;
}
void carFunctwo(int a){
    //
}
```

- C와 달리 typedef를 쓰지 않아도 된다.
- 구조체 변수를 정의할 때, 구조체 내부 변수들을 초기화할 수 있다.
- 구조체 안에 함수를 정의할 수 있다.
  - 구조체 내에 함수를 정의하면 자동으로 inline 처리가 된다.
  - 내부에서 선언, 외부에서 정의하면 inline 처리가 자동으로 되지 않는다.

## 2. 클래스(Class) 와 객체(object)

```cpp
class Car{
    char ID[10];
    int num;
    ~~~
}

int main(){
    Car myCar={"sonata", 100}; (x)
    Car.num = 10; (x)
}
```

- struct 대신 class를 쓰면 구조체가 된다.
- 구조체 내부의 함수에 직접적인 접근은 불가능하다.
  - 직접적인 접근을 위해서는 별도의 선언이 필요하다.

### 접근제어 지시자

- 종류
  1. public : 어디서든 접근 허용
  2. protected : 상속 관계에서 유도 클래스(하위 클래스)에서의 접근 허용
  3. private : 클래스 내에서 정의된 함수에서만 접근 허용

```cpp
class Car{
Private:
    int num1;
    int num2;
public:
    int num3;
}
```

- 접근 제어 지시자 A가 선언되면 그 이후의 변수나 함수는 A에 해당하는 범위내에서 접근 가능.
- 새로운 접근 제어 지시자 B가 선언되면 그 아래에의 변수나 함수는 B에 해당하는 범위내에서 접근 가능.
- class 를 이용하여 정의한 클래스는 디폴트로 private 적용되어 있다.
  - struct는 디폴트로 public이 적용되어 있다.
  - class와 유일한 차이!
    - class로 할 수 있는건 구조체로도 모두 다 할 수 있다.

> __굳이 왜 class를 사용하는가?  __
>
> - 정보 은닉을 위하여.
>   - 클래스 내부 변수를 private으로 선언하여 해당 변수에 접근하는 함수를 별도로 정의하고 안전한 형태로 변수의 접근을 유도하는 것이 '정보 은닉'
>   - 프로그래머의 실수에 대한 대책
>     - 정보 은닉을 하지 않으면 컴파일러가 잡아 내지 못하는 에러가 발생하여 문제가 커질 수 있다.

### 용어 정리

- 멤버 변수 : 클래스를 구성하는 변수
- 멤버 함수 : 클래스를 구성하는 함수
- 객체 :

### 객체의 생성

- 클래스는 데이터(멤버 변수)와 기능(멤버 함수)로 구성된다.
- 멤버 변수는 선언과 동시에 초기화될 수 없다.
- 객체의 생성 방법

```cpp
ClassName ObjName						// 일반적인 선언
ClassName * ptrObj = new Class Name;	// 동적 할당 (힙 할당)
```





## C++에서의 파일 분할

- A.hpp : 클래스의 선언을 담는다.
  - inline 함수 정의는 header에 포함되어야 한다.
  - header guard를 포함해야한다.
    - `#ifndef~#define~#endif`
- A.cpp : 클래스의 정의(멤버 함수의 정의)를 담는다.
- main.cpp : main을 담는다.

## 기타

- 왜 헤더파일을 써야하고 헤더 가드가 필요한가?

  - 빌드 과정을 이해해야한다.

  1. 각각의 단일 cpp 파일을 컴파일한다.
     - 이때, 컴파일러는 단일 cpp 파일만을 보기 때문에 사용되는 함수의 선언이 모두 포함돼있어야 한다.
  2. 링커가 컴파일된 파일들을 링크한다.
     - 선언(무엇)된 함수를 정의(어떻게)와 매칭시킨다,
  3. A라는 함수 집합을 B,C에서 쓸 때 함수 집합들을 선언하는 것은 번거로우므로 헤더파일을 사용한다.
  4. 이때, B가 C 파일을 include하면?
     - A를 두번 인클루드하므로 선언이 충돌한다.
     - 따라서 header guard 가 필요하다.
  5. A.cpp에 A.hpp는 왜 굳이 include해야하나?
     - include를 해야 A.hpp에서 선언한 함수가 A.cpp에 정의된 함수를 가르킨다는 것을 알 수 있다.
  6. 그냥 cpp 파일을 include하면 안되나?
     - A.cpp가 B.cpp를 include한다고 하자. 
     - 이때, B.cpp에 int myfunc() 가 정의되었고 하자.
     - A.cpp가 B.cpp를 include하면 A.cpp에서 또 int myfunc()를 정의한다.
     - 함수들이 중복 정의 되므로 충돌한다.