---
layout: post
title: "C++ 연산자 오버로딩"
date: "2019-10-04"
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

# 연산자 오버로딩

----



- C++에서 연산자 오버로딩에 대한 기본적인 내용을 정리하였다.
- 함수의 오버로딩과 마찬가지로 연산자도 오버로딩이 가능하다.
- __연산자 오버로딩을 통해 객체 간의 연산을 정의할 수 있다.__
  - 객체도 기본 자료형 데이터처럼 연산할 수 있다.

## 1. 연산자 오버로딩의 이해

```cpp
class Point
{
    int xpos, ypos;
    Point(int x=0, int y=0) : xpos(x), ypos(y)
    {}
    Point operator+(const Point &ref)
    { 
        Point pos(xpos+ref.xpos, ypos+ref.ypos);
        return pos;
    }
}

int main(void)
{
    Point pos1(3,4);
    Point pos2(10,20);
    Point pos3 = pos1.operator+(pos2);
    Point pos4 = pos1 + pos2;
}
```

- `<return> operator<op>(input)`의 형태로 연산자 오버로딩을 할 수 있다.
  - 위의 예제에서는 `Point operator+(const Point &ref)`의 형태로 덧셈 연산을 오버로딩 하였다.
  - pos3 또는 pos4와 같은 방식으로 + 연산을 할 수 있다.
    - __숫자간의 연산이 아닌 객체 간의 연산!__
- __`pos1 + pos2; = pos1.operator+(pos2);`__
  - __이 관계를 잘 이해하는 것이 연산자 오버로딩을 사용하는데 큰 도움이 된다.__
- __연산자 오버로딩 함수는 const 선언을 하는 것이 좋다.__
  - 연산은 피 연산자를 바꾸는 것이 아닌 새로운 결과를 생산하는 것이기 떄문.

## 멤버 함수 연산자 오버로딩 vs 전역 함수 연산자 오버로딩

```cpp
class Point
{
private:
    int xpos, ypos;
public:
    Point(int x=0, int y=0) : xpos(x), ypos(y)
    {}
    Point operator+(const Point &ref);
    friend Point operator+(const Point &pos1, const Point &pos2)
}

Point Point::operator+(const Point &ref)				// 멤버 함수 연산자 오버로딩
{
    Point pos(xpos+ref.xpos, ypos+ref.ypos);
    return pos;
}

Point operator+(const Public &pos1, const Point &pos2)	// 전역 함수 연산자 오버로딩
{
    Point pos(pos1.xpos+pos2.xpos, pos1.ypos+pos2.ypos);
    return pos;
}
```

- 전역 함수 연산자 오버로딩은 클래스 내에서 friend 선언을 해야 private 영역에 접근할 수 있다.
  - 대부분의 경우 멤버 함수로 오버로딩하는 것이 좋다.

## 오버로딩이 불가능한 연산자

| .      | .*          | ::           | ?:         | sizeof           |
| ------ | ----------- | ------------ | ---------- | ---------------- |
| typeid | static_cast | dynamic_cast | const_cast | reinterpret_cast |

## 멤버 함수 기반으로만 오버로딩이 가능한 연산자

| =    | ()   | []   | ->   |
| ---- | ---- | ---- | ---- |
|      |      |      |      |

## 연산자 오버로딩의 주의사항

1. __본래의 의도를 벗어난 형태의 연산자 오버로딩은 하지 않는다.__
   - 사칙연산에 대해 피 연산자의 값을 변화시키지 않는다.
   - 해당 연산자에 대해 직관적이고 합리적인 멤버 변수간의 연산을 사용해야한다.
2. __연산자의 우선순위와 결합성은 바뀌지 않는다.__
   - 연산자 오버로딩은 기능만 변하지 우선 순위, 결합성은 변하지 않는다.
3. __매개 변수의 디폴트 값 설정이 불가능하다.__
4. __연산자의 순수 기능으로 오버로딩해야한다.__
   - 즉, 덧셈 연산의 연산자 오버로딩의 결과를 뺼셈 연산으로 하지 않는다.
     - 컴파일러에서 에러를 낸다.

## 단항 연산자의 오버로딩

```cpp
class Point
{
private:
    int xpos;
public:
    Point& operator--()		// 전위 감소
    {
        xpos-=1;
        return *this;
    }
    Point& operator--(int)	// 후위 감소
    {
        xpos-=1;
        return *this;
    }
    friend Point& operator--(Point &ref);		// 전역 전위 감소
    friend const Point operator--(Point &ref, int);	// 전역 후위 감소
}

Point& operator--(Point &ref)
{
    ref.xpos -=1;
    return ref;
}

const Point operator--(Point &ref, int)
{
    const Point retobj(ref);
    ref.xpos -=1;
    return refobj;
}

```

- 매개 변수에 int가 들어가면 후위 연산(xpos++)를 의미

  - int에 대한 큰 의미는 없다.

- __후위 증가는 const형 반환 선언을 한다__

  - `++(++pos) = ++(pos.operator--()) = ++(const Point형 객체)`  
    ` = const Point형 객체.operator++(int)`
  
  - 결국 const Point 형 객체에서 const 형이 아닌 멤버함수를 호출하므로 에러가 발생한다.
  
    - ++(++num)은 되지만 (num--)--은 에러가 나는 원래 C++언어의 특성을 반영하였다.

## 2. 교환 법칙의 구현

- 자료형이 다른 두 객체에 대해서 교환 법칙이 성립하기 위해서는 매개 변수형을 달리하여 두 개의 오버로딩이 필요하다.

- ex) 객체와 상수

- ```cpp
  class Point
  {
      Point operator*(int times)
      {
          Point pos(xpos*times);
          return pos;
      }
  }
  
  int main(void)
  {
      Point cpy1, cpy2, pos(1);
    cpy1 = pos*3;
      cpy2 = 3*pos;
      return 0;
  }
  ```
  
- `cpy1 = pos*3`은 `cpy1 = pos.operator*(3)`과 같다.

- `cpy2 = 3*pos`는 `cpy2 = 3.operator*(pos)`이므로 에러가 난다.

  - 해결책은 `cpy2=operator*(3,pos)`형태의 연산이 되게 하면 된다.

  - __즉, 전역 연산자 오버로딩!__

  - ```cpp
    class Point
    {
        ...
        friend Point operator*(int times, Point& ref);
    }
    Point operator*(int times, Point& ref)
    {
        Point pos(ref.xpos*times, ref.ypos*times);
        return pos;
    }
    ```

  - 위와 같이 선언을 하면 `cpy2 = operator*(3, pos)`처리가 된다.

  - __위와 같은 상황에 전역 연산자 오버로딩을 사용할 수 있다.__



## 3. 디폴트 대입 연산자

- 대입 연산자 오버로딩을 하지 않으면 __디폴트 대입 연산자__가 생성된다.
  - 디폴트 대입 연산자는 멤버 대 멤버의 얕은 복사를 한다.
    - 즉, 기존의 변수가 가르키는 주소를 복사를 할 뿐이다.
    - 복사 생성자와 같이 기존의 변수가 사라지면 문제가 발생한다.
  - 따라서, __연산자 내에서 동적 할당을 하거나 깊은 복사가 필요하면 직접 정의해야 한다.__

1. 복사 생성자 호출  

   ```cpp
   int main(void)
   {
       Point pos1(5, 7);
       Point pos2 = pos1;
   }
   ```

   - pos1은 객체가 생성되며 생성자가 호출되었다.
     - pos2는 객체가 생성됨과 동시에 pos1으로 초기화된다.
     - 즉, `Point pos2 = pos1;`은 `Point pos2(pos1);` 과 같다.
       - 따라서, 복사 생성자가 호출된다.

2. 대입 연산자 호출  

   ```cpp
   int main(void)
   {
       Point pos1(5, 7);
       Point pos2(2, 3);
       pos2 = pos1;
   }
   ```

   - 이미 객체를 생성하며 각 객체에서 생성자가 호출되었다.
     - 그 후의 대입 연산은 대입 연산자가 처리한다.
     - 즉, `pos2 = pos1;`은 `pos2.operator=(pos1)`과 같다.

### 디폴트 대입 연산의 문제점

- pos1의 A 멤버 변수가 0x1000을 참조하고 pos2의 A 멤버 변수가 0x2000을 참조하고 있다면  

  | 주소   | 참조하는 변수 |
  | ------ | ------------- |
  | 0x1000 | pos1.A        |
  | 0x2000 | pos2.A        |

  와 같다.

- 이때, `pos2 = pos1;`이 실행되면  

  | 주소   | 참조하는 변수  |
  | ------ | -------------- |
  | 0x1000 | pos1.A, pos2.A |
  | 0x2000 |                |

  와 같이 같은 곳을 참조하게 된다.

  - 이때, 0x2000 주소를 참조하는 변수가 없으므로 __메모리 누수__가 발생한다!

- 게다가, `delete pos1`을 하게되면  

| 주소           | 참조하는 변수 |
| -------------- | ------------- |
| 0x1000(사라짐) | pos2.A        |

​	와 pos.A는 사라진 주소를 참조한다!

- 즉, __메모리 누수__와 __잘못된 참조__ 문제가 발생한다.

### 깊은 복사 대입 연산자

```cpp
Point& operator=(const Point& ref)
{
    delete A;		// 메모리 누수를 막기 위한 해제
    A = new int;	// 같은 값을 갖지만 다른 주소를 참조하게 만들기
    A = ref.A;
    return *this;
}
```



### 상속 구조에서의 대입 연산자

- 하위 클래스 객체가 생성될 때, 하위 클래스 생성자 뿐만 아니라 상위 클래스 생성자 역시 호출된다.
- __객체간의 대입 연산이 이루어질 때, 하위 클래스의 대입 연산자는 호출되지만 상위 클래스의 대입 연산자는 호출되지 않는다!__
  - 즉, 상위 클래스의 멤버 대 멤버 복사는 따로 명시해야한다.

```cpp
class First
{
    ...
}

class Second : public First
{
	..
    Second& operator=(const Second& ref)
    {
    	First::operator=(ref);			// 이렇게 상위 클래스 대입 연산자를 호출해주어야 한다!
    	...
    	return *this;
    }
}
```



## 4. 배열의 인덱스 연산

- C/C++ 배열은 __경계 검사__를 하지 않는다.
  - 즉, `int arr[3]`을 선언하더라도 `arr[-1] = 0` 또는 `arr[4] = 0`과 같이 호출할 수 있다.
  - 배열 클래스는 이러한 단점을 해결할 수 있다.

### 배열 클래스

- `arrObj[2]`는 `arrObj.operator[](2);`를 의미한다.
  - `int operator[] (int idx) {...}`
- 해결책

```cpp
class BoundChkArr
{
private:
    int * arr;
    int arrlen;
    BoundChkArr& operator=(const BoundChkArr& arr);		---- (1)
    BoundChkArr(const BoundChkArr& arr) {};				---- (2)
public:
    BoundChkArr(int len):arrlen(len)
    {
        arr = new int[len];
    }
    int& operator[] (int idx)
    {
        if(idx<0 || idx>arrlen)
        {
            const<<"Array idx exception" <<endl;
            exit(1);
        }
        return arr[idx];
    }
    int& operator[] (int idx) const						---- (3)
    {
        if(idx<0 || idx>arrlen)
        {
            const<<"Array idx exception" <<endl;
            exit(1);
        }
        return arr[idx];
    }
    ~BoundchkArr()
    {
        delete[]arr;
    }
}

int main(void)
{
    BoundChkArr arr(5);
    arr[3] = 1;
    cout << arr[3] << endl;
    BoundchkArr cpy1(5)
    cpy = arr;											---- (1)
    BoundchkArr cpy2 = arr;								---- (2)
    return 0;
}
```

- BoundchkArr class 객체 arr을 생성
  - len = 5의 array 생성
  - arr[3]에 1을 대입, 출력
    - arr[3]을 호출 시,  arr.operator[]가 호출되며 실제 배열에 접근하듯이 사용!
    - 게다가, 경계 검사까지 해준다.
- 배열 안의 데이터는 유일하게 저장하는 것이 올바르다. 
  - 다른 배열 클래스 객체로의 대입 또는 복사를 원천적으로 막는 것이 좋다.
  - (1)
    - `cpy = arr`을 할 시,  대입 연산자가 실행된다.
      - 대입 연산자가 private 영역에 정의되므로 실행시 오류 발생
      - 대입을 원천적으로 봉쇄한다.
  - (2)
    - `BoundChkArr cpy2 = arr;`을 할 시, 복사 생성자가 실행된다.
      - 생성자가 private 영역에 정의되므로 실행시 오류 발생, 복사 원천 봉쇄
- `void ShowAllData(const BoundChkArr& ref)`와 같은 const 매개변수를 사용하는 함수
  - ref array의 값을 변경할 수 없는 좋은 함수!
  - 하지만 `cout << ref[idx] << endl;` 시 에러 발생
    - `ref[idx]`는 `ref.operator[](idx)`와 같고, operator는 const 멤버 함수여야 한다!
    - 따라서, 배열 클래스 정의시 (3)과 같이 const 함수를 오버로딩해주는 것이 좋다.

> 참조자 반환이 자주 쓰인다.  
> 참조자 반환을 쓰는 이유를 다시 정리하자.



## 5. New와 Delete

- new 와 delete는 다른 연산자들과 오버로딩 방식이 다르다.
- 정해진 약속이 있다.

### New

- new의 역할

  1. 메모리 공간의 할당
  2. 생성자의 호출
  3. 할당하고자 하는 자료형에 맞게 반환된 주소 값의 형 변환

  - __new의 overloading은 1번, 메모리 공간의 할당 부분만 overloading 할 수 있다.__

- __new overloading의 약속__

  - 형태 : `void * operator new (size_t size) {...}`

    - `Point * ptr = new Point(3,4)`가 실행되면?

      1. __컴파일러__가 필요한 메모리 공간을 계산하여 size에 전달한다(size는 byte 단위).

      2. __사용자__가 정의한 operator new(size)를 호출한다. (유일하게 오버로딩할 수 있는 영역)

         ```cpp
         void * operator new(size_t size)
         {
             void * adr = new char[size];	// byte 단위 할당
             ....							// 추가적인 사용자 operation
             return adr;
         }
         ```

      3. __컴파일러__가 반환된 void형 포인터 adr을 할당하고자 하는 자료형에 맞게 형 변환

         - > size_t : typedef unsigned int size_t;
         
         - new가 어떻게 operator new안에 포함될 수 있는가?
         
           -  new는 static으로 이미 선언되어있음.

### Delete

- delete의 역할

  1. 소멸자의 호출
  2. 메모리 공간 해제

  - __delete의 overloading은 2번, 메모리 공간 해제 부분만 overloading할 수 있다.__

- __delete overloading의 약속__

  - 형태 : 

    ```cpp
    void operator delete (void * adr)
    {
        delete []adr;
    }
    ```

### new 와 new[]

- `void * operator new[] (size_t size){...}`
  - 객체 배열 선언시 사용된다.
  - 3개의 객체 배열이 선언되면? `Point * arr = new Point[3];`
    - 3개의 객체에 필요한 메모리 공간을 바이트 단위로 계산해서 size에 전달한다.
- delete와 []delete 도 같다!



## 6. Pointer 연산자 오버로딩

- 두 가지 포인터 연산자 모두 단항 연산자의 형태로 오버로딩이 가능하다.
  - -> : 포인터가 참조하는 객체의 멤버에 접근
  - . : 포인터가 참조하는 객체에 접근

```cpp
class Number
{
private:
    int num;
public
    Number(int n) : num(n) {}
    void ShowData() {}
    Number * operator->()
    {
        return this;
    }
    Number& operator*()
    {
        return *this;
    }
}

int main(void)
{
    Number num(20);
    num.ShowData();
    
    (*num)=30;
    num->ShowData();
    (*num).ShowData();
    return 0;
}
```

- `(*num) = 30;`은 `(num.operator*())=30;`과 같다.

  - 즉, `num=30`과 같다.

- `(*num).ShowData();`는 `(num.operator*()).ShowData();`와 같다.

  - 즉, `num.ShowData()l`와 같다.

- `num->ShowData();`는 `num.operator->()ShowData();`와 같다?

  - 컴파일러에서는 이것을  `num.operator->()->ShowData();`로 해석해서 처리한다.

  - > 실제로는 이렇게 구현하면 main에서 Number class의 private 멤버 변수인 num의 주소 값을 획득하므로 private 멤버 변수의 값을 마음대로 바꿀 수 있게 된다. 이것은 좋지 않은 구현이다.
    >
    > 이 문제를 해결하려면 const 선언으로 오버로딩을 하면 되며 다음에 설명할 스마트 포인터는 const 선언을 통해 연산자를 오버로딩한다.

### 스마트 포인터(Smart Pointer)

- 스마트 포인터는 어떠한 동작을 할 수 있는 포인터다.

- 포인터 오버로딩을 통해 클래스로 구현한 포인터이다!

- > 중요하므로 추후에 추가적으로 정리한다.

```cpp
class Point
{
    int xpos;
    Point(int x=0) : xpos(x) {}
    ~Point();
    void SetPos(int x)
    {
        xpos = x;
    }
};

class SmartPtr
{
    Point * posptr;
    SmartPtr(Point * ptr) : posptr(ptr) {}		
    Point& operator*() const {return *posptr;}	----(1)
    Point* operator->() const {return posptr;}	----(2)
    ~SmartPtr() {delete posptr;}				----(3)
};

int main(void)
{
    SmartPtr sptr1(new Point(1));
    sptr1->SetPos(10);
}
```

- (1), (2) 에서 SmartPtr 객체가 마치 pointer처럼 동작할 수 있도록 연산자를 오버로딩하였다.
  - const 선언을 통해 private 멤버 변수를 참조는 할 수 있지만 값을 변경하지는 못하게 한다.
- __(3) 에서 smartpointer 객체가 해제될 떄, 참조하는 객체가 함께 해제가 된다!__
  - 스마트 포인터의 핵심



## 7. Functor

- 함수를 호출할 떄, 사용하는 () 역시도 연산자이므로 오버로딩이 가능하다.
- 다음과 같이 같은 상황에 다른 함수가 호출되도록 하는 용도로 사용 가능하다.

```cpp
class Adder
{
    int operator()(const int& n1, const int& n2)
    {
        return n1+n2;
    }
    double operator()(const double& e1, const double& e2)
    {
        return e1+e2;
    }
}

int main(Void)
{
    Adder adder;
    adder(1, 3);
    adder(2.4,1.3);
}
```

- `adder(1,3)`은 int 형 덧셈 연산자를 호출한다.

- adder(2.4, 1.3)은 double형 덧셈 연산자를 호출한다.

- >  Functor의 더욱 자세한 예는 추후에 정리.



## 8. 형 변환 연산자 오버로딩

- ```cpp
  class Number
  {
      int num;
      ...
  }
  
  int main(void)
  {
      Number num;
      num=30;
      ,,,
  }
  ```

- `num=30`은 좌변이 Number형, 우변이 int 형이다. 어떻게 처리가 되는가?

  1. `num=30`
  2. `num=Number(30);`
  3. `num.operator=(number(30));`
     - 2번에서 임시 객체가 생성되며 형 변환이 되어 처리된다.

### 형 변환 연산자 오버로딩

- 형 변환이 이루어져야 할 때, 형 변환 방법을 명시할 수 있다.

```cpp
class Number
{
    int num;
    Number(int n=0) : num(n)
    operator int ()
    {
        return num;
    }
    ...
}

int main(void)
{
    Number num1(30);
    Number num2 = num1 + 20;		----(1)
    ...
}
```

- `operator int ()`은 (1) 에서 처럼 int형 형 변환이 이루어져야할 때 호출된다.

  - 과정

  1. `Number num2 = num1 + 20;`
  2. `Number num2 = num1.operator int() + 20;`
  3. `Number num2 = 10 + 20;`
  4. `Number num2(30);`

> 만약 Number 클래스에서 +에 대해 연산자 오버로딩이 되어있으면 무엇이 먼저 실행될까?

