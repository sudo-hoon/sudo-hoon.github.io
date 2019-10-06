---
layout: post
title: "C++ 예외처리"
date: "2019-10-06"
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

# 예외처리

----

- C++에서 예외처리에 대한 기본적인 내용을 정리하였다.

- 예외처리란?
  - 예외 :프로그램 실행 도중에 발생하는 예외적인 상황
    - 컴파일 시의 문법적인 에러X
  - 예외가 발생했을 때, 적절히 대처하는 것
- C 언어에서의 예외처리
  - if 문을 이용하여 예외에 대해 조건부로 처리하였다.

## 1. C++의 예외처리 메커니즘

- C++에서는 try, catch, throw를 이용하여 예외를 처리한다.
  - __try__ : 예외를 발견한다.
  - __catch__ : 예외를 잡는다.
  - __throw__ : 예외를 던진다.

```cpp
try
{
    // 예외 발생 지역
    if (예외가 발생)
        throw expn;
    // 나머지 코드
}
catch(처리할 예외의 종류)
{
    // 예외 처리 코드
}
```

- try 내에서 예외가 발생하면 throw를 통해 expn이라는 에러 메세지를 catch에 전달하고 catch에서 해당 예외를 처리한다.

- try block과 catch block 사이에는 아무런 문장도 들어가면 안된다.

- ex)

  - ```cpp
    int main(void)
    {
        int num1, num2;
        cin >>num1>>num2;
        
        try
        {
            if(num2==0)
                throw num2;
            cout << num1/num2 <<endl;
        }
        catch(int expn)
        {
            cout << "error" << endl;
        }
        return 0;
    }
    ```

  - 예외가 발생하여 expn이 throw된다면 아래의 나머지 코드들은 실행되지 않고 바로 catch로 넘어간다.

  - throw 되는 데이터의 자료형과 catch의 매개 변수의 자료형의 일치에 대해서

    - int num2 와 int expn
    - 만약 다르더라도 에러는 나지 않지만 catch block으로 예외 데이터가 전달되지 않는다.
      - 즉, 예외 처리가 되지 않는다.
      - 아래의 스택 풀기에서 설명하듯이, 예외 데이터가 terminator까지 전달되어 프로그램이 종료된다.
    - __마치 overloading 하듯이 여러 catch 구문을 사용하여 한번의 try 구문에 대해 여러 예외를 처리할 수 있다.__

    

### stack unwinding(스택 풀기)

- ```cpp
  void Divide(int num1, int num2)
  {
      if(num2 = 0)
          throw num2;
      ...
  }
  ...
  try
  {
      Divide(num1, num2);
  }
  catch(int expn)
  {
      cout << "error" << endl;
  }
  ```

  - 만약 함수 Divide 내에서 throw가 호출되면?
    - Divide 함수 내에 try-catch 구문이 없기 때문에 Divide 함수를 호출한 곳으로 전달된다.
    - 전달된 곳이 try block에 감싸 있으므로 전달된 expn은 catch로 전달되며 예외처리 된다.
      - 예외가 발생한 함수는 진행을 더 이상 실행되지 않는다!
    - 예외가 바로 처리되지 않고 호출된 영역으로 예외 데이터가 전달되는 것을 __스택 풀기__ 라고 한다.
      - 만약, main에도 try~catch 구문이 없다면?
        - terminator까지 전달되어 프로그램이 종료된다.

### 전달되는 예외의 명시

- 함수를 정의할 때, 발생 가능한 예외도 명시할 수 있다.

  - `int ThrowFunc(int num) throw (int, char) {...}`

  - 위 정의를 보고 다른 프로그래머가 해당 함수를 사용할 때, 예외 처리를 쉽게 넣을 수 있다.

  - ```cpp
    try
    {
        ThrowFunc(20);
    }
    catch(int expn) { ... }
    catch(char expn) { .... }
    ```

  - 만약, 두 자료형에 해당하는 에러가 없으면 프로그램은 자동 종료된다.

    

## 2. 예외 클래스

- 예외 데이터로 객체가 사용될 수 있다.

  - 이것이 더 일반적이며, 예외 상황을 더 잘 설명할 수 있다.

  - ```cpp
    class DepositException
    {
        int reqDep;
        DepositException(int money) : repDep(money)
        { }
        void ShowExceptionReason()				// 예외를 설명할 수 있는 멤버 함수
        {
            cout << "Exception reason : ~~~" << endl;
        }
    }
    
    class Account
    {
        ...
        void Deposit(int money) throw (DepositException)	// 발생 가능한 예외 명시
        {
            if(money<0)
            {
                DepositException expn(money);	// 에러 발생시 예외 객체 생성!
                throw expn;
            }
        }
        ...
    }
    int main(void)
    {
        try
        {
            ~~
        }
        catch(DepositException &expn)
        {
            expn.ShowExceptionReason();
        }
    }
    ```

  - 객체 내에서 예외를 설명할 수 있는 멤버 변수, 함수를 포함함으로써 예외 상황을 더 잘 표현할 수 있다.



## 3. 상속과 예외 클래스

- 예외 클래스도 상속이 가능하다.

- ```cpp
  class AccountException
  {
      virtual void ShowExceptionReason() = 0; 	// 순수 가상 함수
  }
  class DepositException : public AccountException
  {
  	....
  	void ShowExceptionReason(void) throw (AccountException) {....}
  }
  class WithdrawException : public AccountException
  {
  	....
  	void ShowExceptionReason(void) throw (AccountException) {....}
  }
  
  int main(void)
  {
      try
      {
          ....
      }
      catch(AccountException &expn)
      {
          expn.ShowExceptionReason();		// 여러 예외 상황을 한번에 표현할 수 있다.
      }
      catch(DepositException &expn)
      {
          expn.ShowExceptionReason();		----(1)
      }
  }
  ```

  - 예외 클래스의 상속은 여러 예외처리를 한번에 할 수 있도록 한다.

  - 이러한 특성으로 특정한 예외 클래스를 처리하고 싶을 때, 처리를 못할 수 있다.

    - (1) 처럼 DepositException에 대해 처리하려고 하더라도 위에서 catch 구문이 걸리므로 아래는 실행되지 않는다.

    - ```cpp
      catch(DepositException &expn)
      {
          expn.ShowExceptionReason();		// 순서를 바꾼다.
      }
      catch(AccountException &expn)
      {
          expn.ShowExceptionReason();	
      }
          
      ```

    - 위와 같이 하위 클래스를 상단에 두어서 특정 하위 클래스 예외만 실행되도록 할 수 있다.

## 4. 예외처리 기타

### new

- new가 실패하면 bad_alloc 예외가 발생한다.

- ```cpp
  catch(bad_alloc &bad)
  {
      cout<<bad.what()<<endl;
  }
  ```

### 모든 예외 처리

- ```cpp
  try{}
  catch(...)
  {
      ....
  }
  ```

- `catch(...)`은 모든 예외를 받는다.

### 예외 다시 던지기

- catch 블록에 던져진 예외는 다시 던질 수 있다.

