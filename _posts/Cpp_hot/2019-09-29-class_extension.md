---
layout: post
title: "C++ Class의 완성"
date: "2019-09-29"
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

# C++ Class 기본 정리

----

- C++에서 클래스에 대한 기본적인 내용을 정리하였다.

## 1. Class의 기본

### 정보 은닉

- 클래스 내의 멤버 변수는 private 선언으로 외부 참조로 부터 보호되어야한다.
  - 제한된 방법으로의 접근만을 허용하여 잘못된 값이 저장되지 않고 실수가 발견되도록 해야함.
  - 제한된 방법 : 멤버 함수, 엑세스 함수라고도 부른다.

__const 함수__

```cpp
int myFunc(); // 일반 함수
int yourFunc() const // const 함수
{
    myFunc(); // 일반 함수를 const 함수에서 호출하므로 error
}
```

- MyFunc 함수 내에서는 멤버 변수의 값을 바꾸는 행동을 하지 않겠다는 선언.
- const 함수 내에서는 const 함수만을 호출할 수 있다.



### 캡슐화

- 클래스 내에서 기능 별로 멤버 함수를 생성한다.
- 하나의 행동을 위해서 기능들을 하나의 함수로 묶는 것을 캡슐화라고 한다.
  - 감기 약을 먹을 때 코 감기약, 목 감기약, 기침약을 묶어서 먹는 것 처럼.



## 2. 생성자(Constructor)와 소멸자(Destructor)

```cpp
class SimpleClass
{
    int num;
    SimpleClass(int n) // 생성자
    {
        num = n;
    }
    ~SimpleClass()	// 소멸자
    {}
    int GetNum() const
    {
        return num;
    }
}
```

- __생성자__
  - 클래스의 이름과 같고 반환형을 선언하지 않은 함수.
  - 선언을 안하면 아무런 역할을 하지 않는 디폴트 생성자가 생성된다.
  - '객체 생성시에 딱 한번 호출된다.'
    - 오버로딩이 가능하다.
    - 디폴트 매개 변수가 사용 가능하다.
      - 일반적인 멤버 함수의 역할과 같다.
- __소멸자__
  - 클래스의 이름에 ~가 붙는다.
  - 선언을 안하면 아무런 역할을 하지 않는 디폴트 소멸자가 생성된다.
  - '객체 소멸 시 딱 한번 호출된다.'
- __생성자의 호출__

```cpp
SimpleClass sc1;	// 생성자가 선언되지 않아 디폴트 생성자가 생성되었을 때
SimpleClass * ptr1 = new SimpleClass;	// 역시 마찬가지
SimpleClass * ptr1 = new SimpleClass(10);	// 선언한 생성자가 int 매개 변수를 받을 때
```

- SimpleClass sc1(); (x)

  - void input, SimpleClass 반환형을 받는 함수 선언과 같아서 에러 발생.

-  선언한 생성자의 형태와 같이 호출해야한다.

  

- __Member Initialization__

```cpp
class SimpleClass
{
    int num1;
    SimpleClass(int n)
    {
        num1 = n;
    }
}
class YourClass
{
    SimpleClass sc1;
    int num2;
    YourClass()
        :sc1(10), num2(3);	// Member initialization, sc1의 num1 = 10, num2 = 3이 됨.
    { 
     };
}
```

- Member initialization을 통해 객체 생성자를 호출 할 때, 다른 객체의 생성자를 호출할 수 있다.
- 또한 내부 멤버 변수를 초기화할 수 있다.
  - initializer를 이용하면 선언+초기화로 바이너리가 생성된다.
  - const 멤버 변수도 initializer를 이용하면 초기화할 수 있다.
  - 참조자도 멤버 변수로 선언될 수 있다.
    - const 변수, 참조자는 선언과 동시에 초기화가 이루어져아한다.
- __객체의 생성 과정__
  - 1. 메모리 공간 할당
    2. initializer를 통한 멤버 변수(객체) 초기화
    3. 생성자의 몸체 실행

## 3. 클래스 배열, this pointer

```cpp
soSimple arr[10];	// 정적 할당
soSimple * ptrArr = new soSimple[10];	// 동적 할당
soSimple * parr[10];	// 객체 포인터 배열
```

- 10개의 객체가 모두 같은 생성자가 호출된다.
  - 즉, 따로 다른 생성자를 호출시킬 수 없다.
  - 또한 매개 변수 전달이 불가능하므로 void형 생성자만이 호출 가능하다.
    - 즉, 디폴트 생성자 또는 void형 생성자가 선언되어 있어야한다.

### This pointer

```cpp
Class Number
{
	int num;
	Number(int num) : num(10)		// 멤버 변수 num = 10
	{
		cout << num << endl;		// 출력 : 5
		cout << this-> num << endl;	// 출력 : 10
	}
}
int main()
{
    Number tmp(5);
}
```

- this : class를 참조하는 포인터.

- 용도 : 같은 이름의 멤버 변수와 매개 변수를 구별한다.

  - 멤버 변수를 가르킬 때 사용하면 보기 좋다.

    