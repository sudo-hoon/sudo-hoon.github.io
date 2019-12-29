---
layout: post
title: "C++ 상속"
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

# 상속(Inheritance)

---------------



- C++에서 상속에 대한 기본적인 내용을 정리하였다.
- 상속을 적용해야만 해결 가능한 문제에 대해 논의하고 상속에 대한 기본 개념에 대해 설명하고 해당 문제를 상속을 통해 해결한다.

## 1. 상속이란?

- A 클래스가 B클래스를 상속하면 A 클래스는 B 클래스가 지니고 있는 모든 멤버를 물려받는다.

  - 즉, A 클래스 객체는 A 클래스와 B 클래스의 멤버 함수, 변수를 모두 갖는다.

- 용어 정리

  - | 상위 클래스              | 하위 클래스                 |
    | ------------------------ | --------------------------- |
    | 기초 클래스 (base class) | 유도 클래스(derivved class) |
    | Super class              | Sub class                   |
    | 부모 클래스              | 자식 클래스                 |

### 상속의 방법

```cpp
class Person
{
private:
	int age;
protected:
	char name[10];
public:
    Person(int myage, char * name)
    {
        ...
    }
    int getPay(){
        A...
    }
    int tmp;
}
class UnivStudent : protected Person
{
	char major[10];
	UnivStudent(char * mymajor)
	: Person(myage, myname)// 상속받는 클래스의 멤버 변수를 초기화해주어야 한다.
	{
		strcpy(major, mymajor);
	}
	int getPay(){
		b...
	}
}
```

- 위에서 UnivStudent는 Person class를 상속받는다.
  - UnivStudent로 선언되는 객체는 age와 name을 멤버 변수로 갖는다.

- __상속 받은 클래스의 생성자 정의__
  - UnivStudent는 상속하는 Person의 age와 name를 초기화 해주어야한다.
  - 초기화는 위와 같이 생성자에서 Initializer를 통해 초기화해주는 것이 좋다.
  - 만약, Initializer를 통해 상위 클래스의 생성자를 호출하지 않으면 상위 클래스의 void형 생성자가 자동으로 호출된다.
    - 즉, 하위 클래스가 선언되면 상위 클래스의 생성자는 항상 호출된다.
      - 순서 : 상위 클래스의 생성자 호출 -> 하위 클래스의 생성자 호출
    - 각각 클래스의 멤버 변수는 해당 생성자에서 초기화 해주는 것이 원칙
      - 따라서 하위 클래스에서 상위 클래스의 멤버변수를 직접 초기화 하지 않고 상위 클래스의 생성자 호출을 통해 초기화 한다.
- __상속 받은 클래스의 소멸__
  - 생성자와 마찬가지로 소멸자도 항상 호출된다.
    - 순서 : 하위 클래스의 소멸자 호출 -> 상위 클래스의 소멸자 호출
    - 생성자에서 생성된 동적 메모리는 꼭 해당 클래스의 소멸자에서 해제

### 상속의 접근 제어

- 상속을 하더라도 상속한 클래스의 private 아래의 멤버 변수는 접근할 수 없다.

  - UnivStudent에서 Person의 name과 age에 직접 접근할 수는 없다.

- __Protected__

  - private처럼 외부에서 접근할 수 없지만 상속 받는 하위 클래스에서만 접근 가능하다.
    - 하지만 상속 받는 클래스 사이에서도 정보 은닉을 지키는 것이 좋다.

- __상속의 방법__

  - 위의 코드 블락에서 보이듯이 상속을 할 때도 접근 제어 지시자를 선언할 수 있다.

    - 의미 : 해당 접근 제어 지시자 보다 접근 범위가 넓은 멤버는 해당 접근 제어 지시자로 변경시켜서 상속하겠다.

    - ex) 위에서 protected 상속을 하므로 tmp 멤버 변수는 protected로 변경된다.

      - ```cpp
        class UnivStudent : protected Person
        {
        접근 불가:
        	int age;		// 상속 받았지만 private이었기에 접근 불가
        protected:
        	char name[10];	// 상속 받았기에 접근 가능
        	int name;		// public이었지만 protected 상속이므로 접근 제어 상태 변경
        public:
        	char major[10];
        }
        ```

      - Person을 상속한 UnivStudent의 멤버 변수는 위와 같이 된다.

### 함수 오버라이딩

- 위의 두 클래스 모두에서 getPay() 함수가 존재한다.

  - 이때, __Person의 getPay()는 UnivStudent의 getPay()로 overriding 된다.__

  - `Person::getPay()`로 클래스 명을 명시함으로써 overriding된 함수를 호출할 수 있다.

    

## 2. 상속의 조건

- 상속을 어떠한 상황에 사용해야 하는가?

### IS-A 관계

- __하위 is a 상위 의 관계를 가질 때, 상속을 사용하는 것이 유용하다.__
  - ex) UnivStudent는 Person의 일종이다.
  - ex) 무선 전화기는 전화기의 일종이다.
- 하위 Has 상위의 관계도 가능은 하지만 권장하지는 않는다.
  - ex) 경찰은 총을 갖는다.

- 그 외에는 사용 X



## 3. 객체 포인터

```cpp
Person * ptr;		// Person class type의 포인터 변수를 선언
ptr = new Person();	// Person class 객체의 heap 영역 생성 및 주소 할당

class student : public Person{};
class parttime : public student{};
```

- 위와 같은 class type의 포인터 변수를 __객체 포인터 변수__라고 한다.

### 상속과 포인터 변수

- __AAA형 포인터 변수는 AAA 객체 또는 AAA를 직접 혹은 간접으로 상속하는 모든 객체의 주소 값을 저장할 수 있다.__
  - `Person * ptr = new student()`
  - `Person * ptr = new parttime();`
    - IS-A의 관점 : 학생은 사람의 일종이다, 파트타임 학생은 사람의 일종이다.  -> 합리적이다.

### 포인터 변수의 에러 발생

- 객체 포인터가 AAA를 직 간접적으로 상속하는 객체의 주소를 저장할 수 있더라도 직접 접근할 수는 없다.
- 따라서 위의 객체 포인터를 그대로 사용할 경우 발생하는 에러 상황들이 있다.

``` cpp
class AAA {
    int Afunc();
};
class BBB : public AAA{
    int Bfunc();
};
int main()
{
    AAA * a = new BBB;
    a -> Bfunc(); 		// compile error
    BBB * b = a;		// compile error
    BBB * b = new BBB;
    AAA * a = b;		// compile ok
}
```

- a의 포인터 타입 AAA type이기 때문에 BBB type의 멤버인 Bfunc()을 가르키는 연산을 할 수 없다.
- 두 번째 error 역시도 다른 pointer type이기 때문에 pointer 연산이 불가능하다.
- 세 번째에서는 error가 나지 않는다.
  - 즉, 상위 클래스 type의 포인터는 하위 클래스 타입의 주소를 저장할 수 있다.

## 4. 가상 함수 (Virtual Function)

### 가상 함수

```cpp
class AAA {
    void func(){
        cout << 'A';
    }
};
class BBB : public AAA{
    void func(){
        cout << 'B';
    }
};
int main()
{
    BBB * b = new BBB();
    AAA * a = b;
    
    a->Func();	// A가 나온다.
    b->Func();	// B가 나온다.
}
```

- 위 상황에서 `b->func();` 는 a의 func을 overriding 한 결과이다.
- 그런데, a 포인터 역시 b의 주소값을 가르키는데 불구하고 호출되는 함수가 다르다.
  - 객체지향 프로그래밍에서 이러한 상황이 발생하고, 이것을 부자연스럽게 보는 것 같다.
- __이러한 상황을 막기 위해서 가상 함수라는 것이 존재한다.__

```cpp
class AAA {
    virtual void func(){	// 가상 함수의 선언
        cout << 'A';
    }
};
class BBB : public AAA{
    virtual void func(){
        cout << 'B';
    }
};
int main()
{
    BBB * b = new BBB();
    AAA * a = b;
    
    a->Func();	// B가 나온다.
    b->Func();	// B가 나온다.
}
```

- 위와 같이 가상 함수를 선언하면 pointer 데이터 타입에 상관없이 저장된 주소값에 해당하는 함수가 호출된다.

> 다시 한번 왜 상속을 하는가?  
> 상속은 class 간의 공통된 규약을 표현할 수 있다. 예를 들어, permanent worker, Temporary worker, salewoker 세 종류의 Employee가 있다면 공통으로 Employee로 묶을 수 있다. 따라서 가장 상위 클래스로 Employee가 존재하고 그 Employee들을 다루는 클래스(컨트롤 클래스라고 불린다.)EmployeeHandler 클래스가 선언되어 있다면 모든 종류의 Employee들을 묶어서 통제할 수 있고 공통된 방식으로, 일관되게 데이터를 처리할 수 있다.

### 순수 가상 함수(Pure Virtual Function)과 추상 클래스(Abstract Class)

- 위에서 언급한 Employee 클래스는 모든 worker들이 공통으로 가지는 상위 클래스의 개념일 뿐이지 객체의 생성의 목적은 아니다.
  - __객체 생성을 목적으로 정의되지 않는 클래스가 존재한다.__
  - 이럴때, 객체를 생성하는 것 자체가 문제의 소지가 있으므로 컴파일시 에러가 발생하게 만들어야 한다.

- 순수 가상 함수의 표현 : 선언시 모든 함수를 가상 함수 처리 및 = 0으로 초기화한다.

  ```cpp
  class Employee
  {
      char name[50];
      Employee(char * name){..}
      void ShowYourName() const {...}
      virtual int GeyPay() const = 0;			// 순수 가상 함수
      virtual int ShowSalaryInfo() const = 0; // 순수 가상 함수
  }
  ```

  - `virtual ~~~ =0`은 클래스의 멤버 함수가 정의 되지 않았음을 의미한다.
    - 즉, 이후에 상속하는 객체가 해당 순수 가상 함수를 정의할 것이라고 의미를 주는 것.
  - `Employee * emp = new Employee("KDK")` 로 객체를 생성하면 컴파일 에러가 발생한다.
    - 하나 이상의 순수 가상 함수가 선언되면 컴파일 에러가 발생한다.
    - 이러한 클래스를 __Abstract Class__라 한다.
    - 꼭 상속을 받아서 객체를 생성해야한다.
  - 순수 가상함수들로 구성된 class를 __interface__라 한다.
    - interface는 polymolphism 관점에서 사용된다.

### 가상 소멸자

- 만약, BBB 클래스가 AAA클래스를 상속할 때, `AAA * Aptr = new BBB , delete Aptr`을 했을 때,  
  AAA 클래스의 소멸자만 호출된다.
- 따라서, 소멸자에도 가상 함수 선언을 해주어야 한다.
  - 즉, `virtual ~AAA()`
  - 가상 소멸자가 선언되면 하위 클래스의 소멸자부터 상위 클래스까지 순차적으로 호출된다.

### 참조자

- 참조자의 선언 타입에 따라 호출되는 멤버 함수 역시 달라진다.
- `AAA & sref = new BBB` 일때, 호출되는 함수는 AAA의 멤버 함수이다.
  - 이 역시도 virtual 선언을 통해서 BBB의 멤버 함수가 호출되게 할 수 있다.

## 5. 다형성(Polymorphism)

- 위와 같은 가상 함수 호출 관계에서의 특성을 다형성이라 한다.
  - 즉, 모양은 같은데 형태는 다르다.
  - 선언되는 데이터 타입에 따라 호출되는 클래스의 함수가 다른 것 같은 상황을 의미.



## 6. 기타

- 상속은 얕은 상속이 좋다.
  - 꼭 상속을 쓰지 않아도 될 떄는 클래스 안에 다른 클래스의 객체를 멤버 변수로 including하는 방식이 좋다.

### Strategy pattern

```cpp
#include <iostream >
using std :: cout; using std :: endl;
struct Strategy { // Saving space , should be classes.
virtual void Print () const = 0;
};
struct StrategyA : public Strategy {
void Print () const override { cout << "A" << endl; }
};
struct StrategyB : public Strategy {
 void Print () const override { cout << "B" << endl; }
};
struct MyStruct {
 MyStruct(const Strategy& s): strategy_ (s) {}
 void Print () const { strategy_ .Print (); }
 const Strategy& strategy_ ;
};
int main () {
 // Create a local var of MyStruct and call its Print
 MyStruct( StrategyA ()).Print ();
 MyStruct( StrategyB ()).Print ();
}
```

- 위와 같이 interface를 통해 같은 명령어(Class_object.print())로 다른 기능을 표현할 수 있다.
  - 예를 들어서 device가 i2c와 spi를 지원할 때, open 명령어를 만들고 그것을 stategy pattern으로 구현할 수 있다.