---
title: "씹어먹는 C++정리"
date: 2026-07-01 09:00:00 +0900
categories: [Study, CPP_Study]
tags: [CPP]
---

# 씹어먹는 C++

# C++의 참조자(레퍼런스)

## 기존 C에서의 참조

```cpp
#include <iostream>
int change_val(int *p) {
*p = 3;
return 0;
}
int main() {

int number = 5;
std::cout << number << std::endl;
change_val(&number);
std::cout << number << std::endl;

}
```

- 포인터를 인자로 받은 후에 포인터를 역참조 해서 값에 접근

## C++에서 새롭게 추가된 참조 방식

```cpp
int main() {
int a = 3;
int& another_a = a;
another_a = 5;
std::cout << "a : " << a << std::endl;
std::cout << "another_a : " << another_a << std::endl;
return 0;
}
```

- 참조 연산자 &를 사용하여 참조를 표시
- 참조는 포인터와 달리 Nullable하지 않으며 정의 시에 뭘 참조하는지 명확히 지정 해야 함
- 한 번 어떤 변수의 참조자가 되면 참조를 변경할 수 없음
- 만약 a를 참조하는 참조자 another_a가 있다면 컴파일러는 another_a를 위해서 메모리 상에 공간을 할당할 필요 없이 another_a를 a로 교체하면 됨
- 하지만 항상 레퍼런스가 메모리상에 존재하지 않는 것은 아님

## 함수 인자로 레퍼런스 받기

```cpp
#include <iostream>

int change_val(int &p) {
p = 3;
return 0;
}

int main() {

int number = 5;
std::cout << number << std::endl;
change_val(number);
std::cout << number << std::endl;

}
```

- 함수의 인자 int &p가 정의되는 순간은 change_val(number)로 호출할 때 이므로 사실상 int& p = number와 동일하기 때문에 문제 없음.

## 상수에 대한 참조자

※ 컴파일 오류가 나타나는 코드

```cpp
#include <iostream>

int main() {

int &ref = 4;
std::cout << ref << std::endl;

}
```

- C++문법 상 상수 리터럴을 일반적인 레퍼런스가 참조하는 것은 불가능함

```cpp
const int &ref = 4
```

- 상수 참조자로 선언하면 리터럴도 참조 가능하긴 함

## 레퍼런스의 배열과 배열의 레퍼런스

- 레퍼런스의 레퍼런스, 레퍼런스의 배열, 레퍼런스의 포인터는 존재할 수 없음
- C++상 배열의 이름은 (arr) 첫 번째 주소값으로 변환이 될 수 있어야 arr[1]이 *(arr + 1)로 변경되서 처리가 가능
- 그런데 레퍼런스는 특별한 경우가 아닌 이상 메모리 상에서 공간을 차지 하지 않기 때문에 주소값이 존재할 수 없음.
- 따라서 레퍼런스의 배열은 불가능
- 하지만 그 반대인 배열들의 레퍼런스가 불가능 한 것은 아님

 

```cpp
#include <iostream>

int main() 
{
int arr[3] = {1, 2, 3};
int(&ref)[3] = arr;
ref[0] = 2;
ref[1] = 3;
ref[2] = 1;
std::cout << arr[0] << arr[1] << arr[2] << std::endl;
return 0;

}
```

- 이 코드는 컴파일 가능함

```cpp
int arr[3] = {1, 2, 3};
int (&ref)[3] = arr;
```

- ref가 arr을 참조하도록 했기 때문에 ref[0]~ref[2]가 각각 arr[0]~arr[2]의 레퍼런스가 됨
- 포인터와는 달리 배열의 레퍼런스는 참조하기 위해서 반드시 배열의 크기를 명시해야 함

## 레퍼런스를 리턴하는 함수

### 지역변수의 레퍼런스를 리턴

```cpp
int& function() 
{
int a = 2;
return a;
}

int main() 
{
int b = function();
b = 3;
return 0;
}
```

- 컴파일 시 경고가 뜨고 정의되지 않은 동작이 발생하는 코드
- function은 참조자를 리턴하는데 function에 정의되어 있는 a는 함수의 리턴과 함께 사라짐
- 참조하고 있던 변수가 사라져버려 오류가 발생하는 것

<aside>
⛔

**레퍼런스를 리턴하는 함수에서는 지역변수의 레퍼런스를 리턴하지 않도록 조심해야
함**

</aside>

### 외부 변수의 레퍼런스를 리턴

```cpp
int& function(int& a) 
{
a = 5;
return a;
}

int main() 
{
int b = 2;
int c = function(b);
return 0;
}
```

- function이 리턴한 참조자는 함수가 종료된 이후에도 계속 존재하는 변수인 b를 계속 참조함

### 참조자가 아닌 값을 리턴하는 함수를 참조자로 받기

```cpp
int function()
 {
int a = 5;
return a;
}

int main() 
{
int& c = function();
return 0;
}
```

- 상수가 아닌 레퍼런스가 function함수의 리턴값을 참조할 수 없어 컴파일 오류가 발생함
- 지역변수를 참조자로 리턴하는 함수랑 동일하게 함수의 리턴값은 해당 문장이 끝난 후 바로 사라지기 때문에 참조자를 만들면 댕글링 레퍼런스(참조자는 존재하나 참조하는 값이 없음)

```cpp
#include <iostream>

int function() 
{
int a = 5;
return a;
}

int main() 
{
const int& c = function();
std::cout << "c : " << c << std::endl;
return 0;
}
```

- const참조자로 function의 리턴값을 받으면 컴파일 에러가 나타나지 않음
- 예외적으로 상수 레퍼런스로 리턴값을 받게 되면 해당 리턴값의 생명이 레퍼런스가 사라질 때 까지 연장됨

![image.png](/assets/img/cpp/image.png)

## 레퍼런스가 메모리 상에 반드시 존재해야 하는 경우

### 참조자의 주소를 사용할 때

```cpp
int a = 10;
int& b = a;

int* p = &b;
```

- &b라는 주소값이 필요할 때는 단순 치환으로 처리하기 어려움
- 내부적으로는 보통 포인터처럼 구현됨

### 참조자를 멤버로 가지는 객체

```cpp
struct S {
    int& ref;
};

int a = 10;
S s{a};
```

- 이 경우 구조체 s가 메모리에 존재하므로 객체의 일부인 s.ref도 메모리에 존재해야하기 때문에 포인터 필드로 저장됨

### 참조자를 캡처하는 람다

```cpp
int a = 10;
auto f = [&a]() { return a; };
```

- 람다 객체 내부에 캡처 정보 저장
- a에 대한 참조를 유지해야 함
- 실제로는 주소 저장

# 객체지향프로그래밍의 시작

## 객체란?

![image.png](/assets/img/cpp/image-1.png)

- 객체는 자기 만의 정보를 나타내는 변수들고 이를 가지고 어떠한 작업을 하는 함수들로 둘러싸고 있다고 보면 됨.
- 이런 객체의 변수나 함수를 보통 인스턴스 변수(instance variable)와 인스턴스 메소드(instance method)라고 부름
- 메소드가 변수들을 감싸고 있는 것 처럼 그림이 그려진 이유는 실제로 변수들이 외부로부터 보호되고 있기 때문.
- 외부에서 어떠한 객체의 인스턴스 변수의 값을 바꾸지 못하고 오직 객체의 인스턴스 함수를 통해서만 가능
- 이를 **‘캡슐화’**라고 부름

## 클래스

- 클래스(Class)는 객체의 설계도라고 볼 수 있음
- 우리는 이 객체의 설계도(클래스)를 통해 실제 객체를 만들게 됨
- 이와 같이 클래스를 이용해서 만들어진 객체를 인스턴스(Instance)라고 부름

## 함수의 오버로딩(Overloading)

```cpp
void print(int x) { std::cout << "int : " << x << std::endl; }
void print(double x) { std::cout << "double : " << x << std::endl; }
```

- 동일한 이름의 메소드를 다른 매개변수를 사용하려 각각 정의하는 것
- 함수의 오버로딩 과정
    1. 자신과 타입이 정확히 일치하는 함수 찾기
    2. 정확히 일치하는 타입이 없는 경우 형변환을 통해 일치하는 함수를 찾음
        1.  Char, unsigned char, short→int
        2.  Unsigned short → int or unsigned int
        3. Float→Double
        4. Enum → int
    3. 일치하는 것이 없다면 좀 더 포괄적인 형변환을 통해 일치하는 함수를 찾음
        1. 임의의 숫자 타입은 다른 숫자 타입으로 변환됨(예시 : float → int)
        2. Enum도 임의의 숫자 타입으로 변환됨(예시 : Enum → Double)
        3. 0은 포인터 타입니다 숫자타입으로 변환된 0은 포인터 타입이나 숫자 타입으로 변환
        4. 포인터는 void 포인터로 변환됨

## 생성자(Constructor)

- 기본적으로 객체 생성시 자동으로 호출되는 함수
- 생성자의 정의

```cpp
// 객체를초기화하는역할을하기때문에리턴값이없다!
/* 클래스이름*/ (/* 인자*/) {}
```

## 디폴트 생성자(Default Constructor)

- 생성자를 명시하지 않았을 때 컴파일러가 자동으로 추가해주는 생성자
- 인자를 가지지 않으며 아무런 일도 하지 않음
- 명시적으로 디폴트 생성자 명시

```cpp
class Test {
public:
Test() = default; // 디폴트생성자를정의해라
};
```

## 생성자 오버로딩

```cpp
Date() {
std::cout << "기본생성자호출!" << std::endl;
year_ = 2012;
month_ = 7;
day_ = 12;
}
Date(int year, int month, int day) {
std::cout << "인자3 개인생성자호출!" << std::endl;
year_ = year;
month_ = month;
da
```

- 생성자도 오버로딩 적용 가능

## 소멸자(Destructor)

- 객체가 소멸 될 때 자동으로 호출되는 함수
- ∼(클래스의이름)으로 정의

## 복사 생성자

- 하나의 객채를 생성하고 그 객체를 이용해 다른 객체들을 복사 생성 하는 것
- 어떤 클래스 T의 복사 생성자의 정의

```cpp
T(const T& a);
```

- const이기 때문에 복사 생성자 내부에서 a의 데이터를 변경할 수 없고 새롭게 초기화 되는 인스턴스 변수들에게 복사만 가능
- 사용 예시

```cpp
Photon_Cannon pc1(3, 3);
Photon_Cannon pc2(pc1);
```

- 디폴트 생성자와 디폴트 소멸자처럼 C++컴파일러는 이미 디폴트 복사 생성자를 지원해주고 있음

## 디폴트 복사 생성자의 한계

- 인스턴스 변수를 복사하는 과정에서 각 인스턴스 변수가 같은 주소값을 가짐

![image.png](/assets/img/cpp/image-2.png)

![image.png](/assets/img/cpp/image-3.png)

- 객체들이 파괴되면서 소멸자를 호출하면 다른 인스턴스 변수는 이미 해제된 변수를 가리키게 되어 댕글링 포인터 발생
- 이 현상을 해결하려면 변수를 그대로 복사하는게 아니라 다른 메모리에 동적 할당 후 그 내용을 복사해야함
    - 메모리를 새로 할당해서 내용을 복사하는 것을 깊은 복사(deep copy)라고 부르며 단순 대입만 해주는 것을 얕은 복사(Shallow Copy)라고 부름.
- 깊은 복사가 필요한 경우에는 사용자가 직접 복사 생성자를 만들어야 함

## 생성자의 초기화 리스트(Initalizer List)

- 기존 생성자

```cpp
Marine::Marine() {
hp = 50;
coord_x = coord_y = 0;
damage = 5;
is_dead = false;
}
```

- 초기화 리스트

```cpp
Marine::Marine() : hp(50), coord_x(0), coord_y(0), damage(5), is_dead(false) {}
```

- 초기화 리스트를 사용하면 생성과 초기화를 동시에 함
- 초기화 리스트를 사용하지 않는다면 생성을 먼저 하고 그 다음에 대입을 수행
- 초기화 리스트를 사용하는 것이 조금 더 효율적인 작업

- 생성과 동시에 초기화 되어야하는 레퍼런스와 상수를 가진 클래스를 생성할 때 초기화 리스트를 사용해야만함

## Static

- Static 멤버 변수는 클래스의 모든 객체들이 공유하는 변수로써 모든 객체들이 ‘하나의’static멤버 변수를 사용함
- static함수는 객체를 만들지 않고 클래스 자체에서 호출할 수 있음, (클래스)::(static 함수) 형식으로 호출

## This

- C++언어 차원에서 정의되어 있는 키워드
- 객체 자신을 가리키는 포인터의 역할을 함

## const함수

- C++에서는 변수의 값을 바꾸지 않고 읽기만 하는 함수를 ‘상수 함수’로써 선언할 수 있음

# 연산자 오버로딩(Overloading)

- 연산자 함수 제작 방법

```cpp
(리턴타입) operator(연산자) (연산자가 받는 인자)
```

- 위 방법 이외에는 함수 이름으로 연산자를 절대 넣을 수 없음
- ==를 오버로딩 하고 싶다면 ==연산자는 그 결과값이 언제나 bool이고 인자로는 ==로 비교하는 것 하나만 받게 됨


# 클래스의 상속

## 상속

- 기반 클래스(Base)를 물려받은 파생(Derived)클래스를 만드는 것을 상속이라 함
- Base와 Derived에 동일한 함수가 정의되어 있을 때 Derived에서 함수를 호출하면 Base의 함수가 아니라 Derived의 함수를 호출함
- 이를 오버라이딩(overriding)이라고 함

## Protected

- private(본인 외에 접근 불가)이나 public(자유롭게 접근 가능)의 중간 위치에 있는 접근 지시자
- 상속받는 클래스에서는 접근 가능하고 그 외의 기타 정보는 접근 불가능

## 가상 함수와 다형성

### Is-a와 has-a

- class Manager : public Employee의 의미
    - Manager클래스는 Employee의 모든 기능을 포함
    - 따라서 Manager를 Employee라고 칭해도 무방함
    - Manager is a Employee
- 모든 상속 관계는 is a라고 볼 수 있음
- Manager is a Employee는 성립하나 그 반대는 성립하지 않음

## 업캐스팅

- 파생 클래스에서 기반 클래스로 캐스팅 하는 것
- 기반 클래스를 가리키는 포인터가 파생 클래스를 가르켜도 문제 없음
- Derived is a Base이기 때문에 가능함

![image.png](/assets/img/cpp/image-4.png)

## 다운 캐스팅

- 암시적인 다운 캐스팅은 컴파일러 상에서 금지하고 있음
- 파생 클래스에서 정의한 함수가 기반 클래스에 존재하지 않을 수 도 있기 때문
- static_cast로 강제로 변환한다면 컴파일은 되지만, 실제 객체의 타입이 다를 때 기반 클래스에 존재하지 않는 함수 접근 시 정의되지 않은 동작이 발생함
- 이러한 캐스팅 오류를 방지하기 위해서 dynamic_cast를 지원함
    - 가상함수 테이블이 있어야지 다운캐스팅을 dynamic_cast로 가능
    - RTTI : **********알아보기*****************
- static_cast는 컴파일 타임에 결정, Dynamic_cast는 런타임에 결정
- static_cast는 런타임에 캐스팅 가능 여부를 확인하는 것이 불가능 하기 때문에 조심해야 함.

## Virtual 키워드

- 기반 클래스나 파생 클래스에서 함수를 호출할 때 실제 객체에 맞는 함수가 호출되도록 하는 키워드
- 컴파일 시에 어떤 함수가 실행될 지 정해지지 않고 런타임 시에 정해지는 일을 가리켜서 동적 바인딩(Dynamic Binding)이라고 부름
- 컴파일 타임에 어떤 함수가 호출될지 정해지는 것을 정적 바인딩(Static Binding)이라고 함
- Virtual 키워드가 붙은 함수를 가상 함수(Virtual Function)라고 함

## Override 키워드

- 파생 클래스에서 기반 클래스의 가상 함수를 오버라이드 하는 경우 override키워드를 통해서 명시적으로로 나타낼 수 있음

## Virtual 소멸자

- 클래스의 상속을 사용할 때 소멸자를 가상함수로 만들어야 함
- 기반 클래스 포인터가 파생 클래스를 가리킬 때 포인터가 가리키는 것은 기반 클래스 객체가 아니라 파생 클래스의 객체이기 때문에 파생 클래스의 소멸자가 호출되어야 하지만 Virtual을 붙이지 않으면 소멸자가 호출되지 않음
- 소멸자가 호출되지 않음으로서 발생하는 메모리 누수를 막기위해 기반 클래스의 소멸자를 virtual로 선언하면 파생 클래스의 소멸자를 성공적으로 호출 가능함.

## 가상 함수의 구현 원리

- C++에서 virtual키워드를 통해 사용자가 직접 가상 함수를 선언하도록 하는 이유 → 가상 함수를 사용하게 되면 약간의 오버헤드가 존재하기 때문

<aside>
⛔

오버헤드 :  **어떤 작업이나 시스템을 운영할 때 직접적인 목표 달성 외에 추가로 소모되는 간접적인 시간, 메모리, 비용, 자원**

</aside>

- C++ 컴파일러는 가상 함수가 하나라도 존재하는 클래스에 대해서 가상 함수 테이블(Virtual function table; vtable)을 만듬

![image.png](/assets/img/cpp/image-5.png)

## 순수 가상 함수( Pure virtual function)와 추상 클래스(Abstract Class)

```cpp
virtual void speak() = 0
```

- 순수 가상함수는 무엇을 하는지 정의되어 있지 않는 함수로 반드시 오버라이딩 되어야만 하는 함수
- 가상함수의 =0을 붙여서 반드시 오버라이딩 되도록 만든 함수를 완전한 가상 함수라해서 순수 가상 함수라고 부름
- 순수 가상 함수는 본체가 없기 때문에 이 함수를 호출하는 것은 불가능함. 그렇기 때문에 순수 가상 함수가 선언된 클래스의 객체를 생성하는 것 조차 불가능함.
- 순수 가상 함수를 가지고 있는 클래스의 객체를 생성하기 위해서는 이 클래스를 상속받는 클래스를 만들어서 모든 순수 가상 함수를 오버라이딩 해줘야함.

## 다중 상속(Multiple Inheritance)

- 한 클래스가 다른 여러개의 클래스를 상속 받는 것
- 상속받은 클래스에 중복된 이름의 멤버 변수나 함수가 존재하면 어떤 클래스의 변수, 혹은 함수인지 구별하지 못하는 컴파일 오류 발생.

## 다이아몬드 상속(Diamond Inheritance)

![image.png](/assets/img/cpp/image-6.png)

- Human에 Name이라는 멤버 변수가 있다고 할 때 HandsomeHuman과 SmartHuman은 모두 Human을 상속 받고 있으므로 Name이라는 변수를 가지게 됨.
- 그런데 Me가 이 두 클래스를 상속받으니 Name이라는 변수가 겹치게됨.
- Human을 상속받는 한 HandsomeHuman과 SmartHuman을 안겹치게 만든다고 해도 Human의 내용이 중복됨.

```cpp
class Human {
public:
};
class HandsomeHuman : public virtual Human {
// ...
};
class SmartHuman : public virtual Human {
// ...
};
class Me : public HandsomeHuman, public SmartHuman {
// ...
};
```

- 해결 방법은 Human을 Virtual로 상속받으면 Me에서 다중 상속 시에도 컴파일러가 언제나 Human을 한 번만 포함하도록 지정 가능
- 가상 상속시에 Me의 생성자에서  HandsomeHuman과SmartHuman 의생성자를호출함은당연하고,Human의생성자 또한호출해주어야만함

## 다중 상속은 언제 사용해야 할까?

## 다중 상속을 써도 되는 경우

### 1. 서로 독립적인 기능(역할)을 조합할 때

예를 들어:

- `Flyable` (날 수 있음)
- `Swimmable` (수영 가능)

```
classFlyable:
deffly(self): ...

classSwimmable:
defswim(self): ...

classDuck(Flyable,Swimmable):
pass
```

- 오리는 "날 수도 있고 수영도 하는" 존재라 자연스러움
- 이런 경우는 **역할 기반 믹스인(mixin)** 패턴이라고 함

### 2. 공통 기능을 재사용하는 믹스인(Mixin)

- 로그 기능
- 직렬화 기능 (`to_dict`)
- 권한 체크

```
classLoggerMixin:
deflog(self,msg): ...

classSaveMixin:
defsave(self): ...

classUser(LoggerMixin,SaveMixin):
pass
```

- "기능 추가" 용도로만 쓰는 경우는 안전한 편임

## 다중 상속을 피해야 하는 경우

### 1. "is-a 관계"를 여러 개 만들 때

```
classBird: ...
classMachine: ...

classFlyingRobot(Bird,Machine):# ⚠️ 애매함
pass
```

- 설계가 꼬이기 쉽고 의미도 불분명해짐

### 2. 부모 클래스들이 서로 상태(state)를 가질 때

- 변수 이름 충돌
- 초기화 순서 문제
- 의도치 않은 override
- 위 문제들 때문에 디버깅이 매우 어려워짐

### 3. 다이아몬드 상속 구조

```
classA: ...
classB(A): ...
classC(A): ...
classD(B,C): ...
```

- A로 가는 경로가 두 갈래가 됨. MRO가 호출 순서를 정해 주지만 추적이 복잡함

## 다중 상속 판단 기준

- 기능을 "붙이는 것"이면 사용해도 됨 (Mixin)
- 정체성을 "합치는 것"이면 위험함 (재설계 고려)

## 대안

### 1. 컴포지션 (Composition)

```
classEngine:
defstart(self): ...

classCar:
def__init__(self):
self.engine=Engine()
```

- 대부분의 경우 다중 상속보다 안전하고 유지보수가 쉬움
- 다중 상속은 "기능 조합(mixin)"에는 쓸 만하지만, "개념/정체성 결합"에는 거의 쓰지 않는 게 좋음

# C++표준 입출력 라이브러리

## C++입출력 라이브러리

![image.png](/assets/img/cpp/image-7.png)

- C++의 모든 입출력 클래스는 ios_base를 기반 클래스로 함
- ios_base클래스는 스트림의 입출력 형식 관련 데이터를 처리
    - 예시) 정수형 출력을 10진수로 할지 16진수로 할지, 실수 형 출력시 정밀도 등
- ios클래스는 실제로 스트림 버퍼를 초기화 함.
    - 스트림 버퍼란 데이터를 내보내거나 받아들이기 전에 임시로 저장하는 곳.
    - 사용자가 1바이트 씩 읽는다고 했을 때 실제 프로그램은 한 뭉터기(1바이트 이상)를 스트림 버퍼에 저장한 후 사용자가 요청할 때마다 1바이트씩 꺼내는 방식
- ios클래스에선 그 외에도 현재 입출력 작업의 상태를 처리
    - 파일을 읽다가 끝에 도달했는지 확인하려면 eof함수 호출
    - 현재 입출력 작업을 잘 수행 할 수 있는지 확인하러면 good함수를 호출

## istream클래스

- istream은 실제로 입력을 수행하는 클래스
- Operator>>, cin등이 클래스의 객체 중 하나.
- Operator>>는 모든 공백문자를 입력시에 무시해 버림.

```cpp
// 주의할점
#include <iostream>
using namespace std;

int main() {
int t;
while (true) {
std::cin >> t;
std::cout << "입력:: " << t << std::endl;
if (t == 0) break;
}

}
```

- 숫자를 입력 받는 코드에서 문자열을 입력하면 무한 루프에 빠지는 오류 발생
- ios에는 스트림의 상태를 관리하는 4개의 플래그가 정의되어있음
    - goodbit  : 스트림에 입출력 작업이 가능할 때
    - badbit : 스트림에 복구 불가능한 오류 발생 시
    - eofbit : 입력 작업 중 EOF에 도달했을 때
    - failbit : 입력 작업이 실패했을 때 (타입 불일치 등, 복구 가능한 오류)
- 타입에 맞지 않는 값을 넣어서 failbit가 켜지면 입력값을 받지 않고 리턴하면서 무한루프 발생

## C++파일 입출력

### fstream

- 파일 스트림은 기본적인 istream이나 ostream클래스 보다 더 지원하는 기능이 많기 때문에 이를 상속 받아서 작성되었고 각각을 상속 받은 것이 ifstream과 ofstream임.
- 이 두 클래스를 모두 포함하는 라이브러리로 fstream을 사용하면 됨.

```cpp
// 파일에서의입출력
#include <fstream>
#include <iostream>
#include <string>

int main() 
{
// 파일읽기준비
std::ifstream in("test.txt");
std::string s;

if (in.is_open()) 
{
in >> s;
std::cout << "입력받은문자열:: " << s << std::endl;
} 

else
 {
std::cout << "파일을찾을수없습니다!" << std::endl;
}

return 0;
}
```

- is_open은 파일이 열렸는지의 유무를 리턴. 해당 경로에 파일이 존재하지않으면 false리턴
- ifstream 객체의 소멸자에서 자동적으로 close를 해주지만 몇몇 경우에는 직접 close를 해야 함.

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() {
// 파일읽기준비
std::ifstream in("test.txt");
std::string s;

if (in.is_open()) {
in >> s;
std::cout << "입력받은문자열:: " << s << std::endl;
} else {
std::cout << "파일을찾을수없습니다!" << std::endl;
}

in.close();
in.open("other.txt");

if (in.is_open()) {
in >> s;
std::cout << "입력받은문자열:: " << s << std::endl;
} else {
std::cout << "파일을찾을수없습니다!" << std::endl;
}
return 0;
}
```

- 새로운 파일에서 같은 객체가 입력을 받기 위해서는 기존 파일 스트림과의 연결을 종료하고 새로운 파일과 연결을 시켜주면 됨.

### 파일 전체를 한 번에 읽기

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main()
 {
// 파일읽기준비
std::ifstream in("test.txt");
std::string s;

if (in.is_open()) 
{
// 위치지정자를파일끝으로옮긴다.
in.seekg(0, std::ios::end);
// 그리고그위치를읽는다. (파일의크기)
int size = in.tellg();
// 그크기의문자열을할당한다.
s.resize(size);
// 위치지정자를다시파일맨앞으로옮긴다.
in.seekg(0, std::ios::beg);
// 파일전체내용을읽어서문자열에저장한다.
in.read(&s[0], size);
std::cout << s << std::endl;
} 
else 
{
std::cout << "파일을찾을수없습니다!" << std::endl;
}

return 0;
}
```

- C언어에서 fseek같은 함수로 파일 위치 지정자를 움직일 수 있었음
    - 두 번째 인자는 파일 내 위치를 의미하고 첫 번째 인자는 그 위치로부터 얼만큼 떨어져 있는지를 의미

```cpp
in.seekg(0, std::ios::end);
```

- 이 경우 위치 지정자를 파일의 끝에서 0만큼 떨어진 곳 - 즉 파일의 끝으로 이동

```cpp
// 그리고그위치를읽는다. (파일의크기)
int size = in.tellg();
```

- tellg함수는 시작 지점으로부터 위치 지정자가 얼마나 떨어져있는지 반환
- 위치 지정자를 파일 끝으로 이동시켰을 경우 tellg함수는 파일의 크기(바이트 단위)로 반환

```cpp
// 위치지정자를다시파일맨앞으로옮긴다.
in.seekg(0, std::ios::beg);
```

- 파일을 읽기 전에 끝으로 옮겼던 위치 지정자를 다시 처음으로 옮겨야 함. 그렇지 않으면 파일에서 아무것도 읽지 못함.

```cpp
// 파일전체내용을읽어서문자열에저장한다.
in.read(&s[0], size);
```

### 파일 전체를 한 줄씩 읽기

```cpp
// getline 으로읽어들이기
#include <fstream>
#include <iostream>
#include <string>

int main() 
{
// 파일읽기준비
std::ifstream in("test.txt");
char buf[100];

if (!in.is_open()) 
{
std::cout << "파일을찾을수없습니다!" << std::endl;
return 0;
}

while (in) 
{
in.getline(buf, 100);
std::cout << buf << std::endl;
}
return 0;
}
```

- ifstream 객체의 멤버 함수로 존재하는 getline 함수는 파일에서 개행문자(\n)이 나올때 까지 최대 지정한 크기 -1 만큼 읽게 됨
    - -1의 이유는 맨 마지막 문자로 널 종료 문자를 넣어줘야 하기 때문.
- 개행문자 말고도 지정한 문자가 나올 때 까지 읽는 것으로 바꿀 수 도 있음

```cpp
in.getline(buf, 100, '.')
//마침표 나 때까지 입력 받음
```

```cpp
while (in) 
```

- ifstream에는 자기 자신을 bool로 캐스팅 할 수 있는 캐스팅 연산자가 오버로딩 되어 있음
- getline함수는 개행 문자가 나오기 전에 지정한 버퍼의 크기가 다 차게 된다면 failbit를 키게 되므로 버퍼의 크기를 너무 작게 만든다면 정상적으로 데이터를 받을 수 없음.
- 이런 한계점 극복을 위해 std::string에서 getline 함수를 제공하고 있음

```cpp
std::string s;
while (in) 
{
getline(in, s);
std::cout << s << std::endl;
}
```

<aside>
⛔

while문의 조건으로 절대 eof()를 사용하면 안됨.

eof함수는 읽기 시도가 파일의 끝에 걸린 뒤에야 true를 반환하기 때문에, 마지막 원소를 읽은 직후에도 false여서 루프가 한 번 더 돌게 됨

</aside>

### 파일에 쓰기

```cpp
#include <iostream>
#include <fstream>
#include <string>

int main() 
{
	// 파일쓰기준비
	std::ofstream out("test.txt");
	std::string s;
	if (out.is_open())
	{
	out << "이걸쓰자~~";
	}
return 0;
}
```

- 만약 파일이 존재하지 않는다면 파일을 생성한 뒤에 생성이 성공하면 출력하게 됨.
- ofstream은 열고자 하는 파일이 존재하지 않으면 해당 파일을 생성하고 열게 됨
- 해당 파일이 존재하면 특별한 설정을 하지 않는 한 해당 파일 내용이 다 지워지고 새로운 내용으로 덮어 씌움

```cpp
#include <fstream>
#include <iostream>
#include <string>

int main() 
{
	// 파일쓰기준비
	std::ofstream out("test.txt", std::ios::app);
	std::string s;
	if (out.is_open()) 
	{
	out << "덧붙이기";
	}
return 0;
}
```

- out객체를 생성할 때 옵션으로 app을 주게 되면 파일에 스트림을 연결 할 때 기존 파일의 내용을 지우고 새로 쓰는 것이 아니라 그 뒤에 새로운 내용을 붙여 쓰게 됨

### 모드 플래그

- ios::binary
    - 파일을 텍스트가 아닌 그대로의 바이트 형태로 읽고 쓰겠다.
    - 데이터를 변환 없이 그대로 처리
    - 이미지, 영상, 실행파일 등에서 필수
- ios::app
    - 파일에 스트림을 연결할 때 기존 파일의 내용을 지우고 새로 쓰는 것이 아니라 위 사진 처럼 그 뒤에 새로운 내용을 붙여 쓰게 됨
- ios::ate
    - 자동으로 파일 끝에서부터 읽기와 쓰기를 실시함
- ios::trunc
    - 파일 스트림을 열면 기존에 있던 내용들이 모두 지워짐. 기본적으로 ofstream객체를 생성할 때 이와 같은 설정으로 만들어짐
- ios::in, std::ios::out
    - 파일에 입력을 할 지 출력을 할 지 지정함.
- ate와 app은 비슷해 보이지만 app은 원본 파일의 내용을 무조건 적으로 보장하지만 ate는 위치 지정자를 그 이전으로 옮길 수 있음
    - 하지만 ate는 기존 파일의 내용을 보존하지 않아서 기존 파일 위치로 지정자를 옮기고 입력하면 기존 데이터 지워짐

### std::ofstream 연산자 오버로딩 하기

```cpp
#include <fstream>
#include <iostream>
#include <string>

class Human 
{
	std::string name;
	int age;
	public:
	Human(const std::string& name, int age) : name(name), age(age) {}
	std::string get_info() 
	{
	return "Name :: " + name + " / Age :: " + std::to_string(age);
	}
	friend std::ofstream& operator<<(std::ofstream& o, Human& h);
};

std::ofstream& operator<<(std::ofstream& o, Human& h) {
	o << h.get_info();
	return o;
}
int main() {
// 파일쓰기준비

std::ofstream out("test.txt");

Human h("이재범", 60);

out << h << std::endl;

return 0;
}
```

### 문자열 스트림

```cpp
#include <iostream>
#include <sstream>
int main() 
{
	std::istringstream ss("123");
	int x;
	ss >> x;
	
	std::cout << "입력받은데이터:: " << x << std::endl;
	
	return 0;
}
```

- sstream에는 std::istringstream이 정의되어 있음
- 이는 문자열을 하나의 스트림이라 생각하게 해주는 가상화 장치
- 이를 사용하면 간편하게 문자열을 숫자로 변환 할 수 있음

```cpp
#include <iostream>
#include <sstream>
#include <string>

double to_number(std::string s) 
{
	std::istringstream ss(s);
	double x;
	ss >> x;
	return x;
}
int main() 
{
	std::cout << "변환:: 1 + 2 = " << to_number("1") + to_number("2") << std::endl;
	return 0;
}
```

- 거꾸로 데이터를 출력할 수 있는 std::ostringstream이 있음
- 거꾸로 숫자에서 문자열로 바꾸는 함수 제작 가능.

```cpp
#include <iostream>
#include <sstream>
#include <string>

std::string to_str(int x)
 {
	std::ostringstream ss;
	ss << x;
	return ss.str();
}
int main() 
{
	std::cout << "문자열로변환:: 1 + 2 = " << to_str(1 + 2) << std::endl;
	return 0;
}
```

# C++템플릿

- 타입에 묶이지 않고 유연한 타입을 지원하는 기능
- 사용자(프로그래머)가 원하는 타입을 넣어주면 알아서 코드를 찍어내는 틀

```cpp
template <typename T>
T add(T a, T b) 
{
    return a + b;
}
```

- 클래스 템플릿에 인자를 전달해서 실제 코드를 생성하는 것을 클래스 템플릿 인스턴스화(Class template Instantiation)이라고 함.
- 템플릿이 인스턴스화 되지 않고 덩그러니 있으면 컴파일 시에 아무런 코드로 변환되지 않기 때문에 템플릿은 반드시 인스턴스 화 되어야 함.

## 템플릿 특수화(Template)

- 일부 경우에 대해서 따로 처리하는 것을 템플릿 특수화라고 함

```cpp
template <typename A, typename B, typename C>
class test {};
```

- 위와 같은 템플릿 클래스에서 템플릿 특수화를 진행하는 예시

```cpp
template <typename B>
class test<int, B, double> {};
```

```cpp
template <>
class test<int, int, double> {};
```

```cpp
template <>
class Vector<bool> 
{
... // 원하는코드
}
```

## 함수 템플릿(Function template)

- 함수 또한 인스턴스 화 되기 전까지는 컴파일 시에 아무런 코드로 변환되지 않음
- 컴파일 시에 모든 템플릿들이 인스턴스화 된다는 사실을 기반으로 여러가지 코드를 짜는 것을 템플릿 메타프로그래밍(Template metaprogramming)이라고 함

```cpp
template <typename T, int num>
	T add_num(T t)
{
	return t + num;
}
```

- 인자로 템플릿만 받을 수 있는 것은 아님

```cpp
template <typename T, int num = 5>
T add_num(T t) {
return t + num;
}
```

- 디폴트 인자도 지정 가능

## 가변 길이 템플릿

- C++템플릿을 이용하면 임의의 개수의 인자를 받는 함수를 구현가능

```cpp
template <typename T, typename... Types>
```

- typename뒤 …으로 오는 것을 템플릿 파라미터 팩(Parameter pack)이라고 부름
- 파라미터 팩의 경우 0개 이상의 템플릿 인자들을 나타냄
- 마찬가지로 함수에 인자로 …로 오는 것을 함수 파라미터 팩 이라고 부르며 0개 이상의 함수 인자를 나타냄

```cpp
template <typename T, typename... Types>
void print(T arg, Types... args) {
std::cout << arg << ", ";
print(args...);
}
```

- 차이점은 템플릿은 타입 앞에 …이 오고 함수의 경우는 타입 뒤에 …이 온다는 점
- sizeof연산자는 인자의 크기를 리턴하지만 파라미터 팩에 sizeof를 사용할 경우 전체 인자의 개수를 리턴하게 됨

## 템플릿 메타 프로그래밍(Template Meta Programming - TMP)

- 템플릿을 사용하면 객체를 생성하지 않더라도 타팁에 어떠한 값을 부여할 수 있고 또 그 타입들을 가지고 연산을 할 수 있음
- 타입은 반드시 컴파일 타임에 확정되어야 하므로 컴파일 타임에 모든 연산이 끝나게 됨.
- 이렇게 컴파일 타임에 생성되는 코드로 프로그래밍을 하는 것을 메타 프로그래밍이라고 함.
- C++에서는 템플릿을 활용해 메타 프로그래밍을 하므로 템플릿 메타 프로그래밍이라 부름.

```cpp
template <int N>
struct Factorial 
{
	static const int result = N * Factorial<N- 1>::result;
};
template <>
struct Factorial<1> 
{
	static const int result = 1;
};
int main() 
{
	std::cout << "6! = 1*2*3*4*5*6 = " << Factorial<6>::result << std::endl;
}
```

- 팩토리얼의 재귀를 끝내기 위해서 템플릿 특수화를 이용해 1일 때 처리를 따로 만듬

### TMP를 왜 쓰는가?

- 어떠한 C++코드도 템플릿 메타 프로그래밍 코드로 변환할 수 있음
- TMP로 작성된 코드는 모두 컴파일 타임에 모든 연산이 끝나기 때문에 프로그램 실행 속도를 향상 시킬 수 있다는 장점이 있음 → 컴파일 타임이 늘어남.
- TMP로 작성된 코드는 컴파일 타임에 연산하는 것이기 때문에 디버깅이 불가능하고 템플릿 오류시에 오류 길이가 김.

# C++표준 라이브러리(컨테이너와 알고리즘)

## C++표준 템플릿 라이브러리(Standard Template Library - STL)

- 보통 C++템플릿 라이브러리(STL)은 세 개의 라이브러리들을 의미함.
- 컨테이너(Container) : 임의 타입의 객체를 보관
- 반복자(Iterator) : 컨테이너에 보관된 원소에 접근
- 알고리즘(Algorithm)

## 시퀀스 컨테이너

### C++ STL 컨테이너 - 벡터(std::vector)

- STL의 컨테이너는 배열 처럼 객체들을 순차적으로 보관하는 시퀀스 컨테이너(Sequence Container)와 키를 바탕으로 대응되는 값을 찾아주는 연관 컨테이너(Associative Container)가 있음
- 시퀀스 컨테이너의 경우 Vector, List, Deque 3개가 정의되어 있음
    - Vector
        - 가변길이 배열로 원소들이 메모리 상에서 순차적으로 저장되어있음
        - 임의의 위치에 있는 원소에 매우 빠르게 접근 가능 O(1)
        - Vector의 크기보다 큰 메모리 공간을 미리 확보해 놓기때문에 맨 뒤에 원소 추가, 제거 또한 O(1)로 수행
        - 하지만 할당된 공간을 다 채웠을 경우에는 새로운 공간을 할당하고 기존의 원소들을 복사하기 때문에 O(n)으로 수행

### 반복자(Iterator)

![image.png](/assets/img/cpp/image-8.png)

- 컨테이너 원소에 접근할 수 있는 포인터와 같은 객체
- 알고리즘 라이브러리에서 대부분이 반복자를 인자로 받아서 알고리즘을 수행함
- 반복자는 컨테이너의 iterator멤버 타입으로 정의되어 있음.

![image.png](/assets/img/cpp/image-9.png)

- 역 반복자도 존재함

### 리스트(list)

![image.png](/assets/img/cpp/image-10.png)

- 양방향 연결 구조를 가진 자료형
- 시작 원소와 마지막 원소만을 기억하기 때문에 Vector와는 달리 임의의 위치에 있는 원소에 바로 접근 불가
- 임의의 위치에 있는 원소에 접근하기 위해서는 하나씩 링크를 따라가야 함.
- Vector는 임의의 위치에 원소를 추가하거나 제거하는 작업이 O(n)이었지만 리스트는 원하는 위치 앞과 뒤에 있는 링크 값만 바꿔주면 되기 때문에 O(1)이 걸림.
- 메모리에 원소들이 연속적으로 존재하지 않을 수 있음
- 리스트에서 정의되는 반복자의 타입을 보면 BidirectionalIterator으로 양방향 이동이 가능하지만 한 칸씩 밖에 이동 불가.
- Vector는 RandomAccessIterator

### 덱 (Deque - double ended queue)

![image.png](/assets/img/cpp/image-11.png)

- 벡터와 비슷하게 O(1)으로 임의의 위치의 원소에 접근할 수 있으며 맨 앞, 뒤에 원소를 추가/제거 하는 자업도 O(1)로 수행 가능.
- 임의의 위치에 있는 원소를 추가/제거에 O(n), 속도도 벡터보다 빠름
- 덱은 원소들이 실제 메모리 상에서 연속적으로 존재하지는 않음. 이 때문에 원소들이 어디에 저장되어 있는지에 대한 정보를 보관하기위해 추가적인 메모리 필요
    - 64비트 libc++라이브러리의 경우 1개의 원소를 보관하는 덱은 그 원소 크기에 비해 8배나 더 많은 메모리를 필요로 함.
    - 덱은 실행 속도를 위해 메모리를 많이 희생하는 자료구조
- 원소들이 일정 크기로 잘려서 각각의 블록 속에 존재.
- 이 블록들이 메모리 상에 어느 곳에 위치하여 있는지 저장하기 위해 각각의 블록들위 주소를 저장하는 벡터를 필요로 함
    - 이 벡터는 기존 벡터와 다르게 할당 시에 앞쪽, 뒤쪽 모두에 공간을 남겨놓음
    - 이를 통해 맨 앞과 맨 뒤에서 삽입 삭제를 빠르게 수행 가능.

### 상황에 따른 컨테이너 선택

- 일반적인 상황에서는 벡터 사용
- 맨 끝이 아닌 중간에 원소들을 추가하거나 제거하는 일을 많이 하고 원소들을 순차적으로 만 접근하면 리스트 사용
- 맨 처음과 끝 모두에 원소들을 추가하는 작업을 많이 하면 덱을 사용.

## 연관 컨테이너

![image.png](/assets/img/cpp/image-12.png)

- 시퀀스 컨테이너와는 다르게 키(Key) - 값(Value) 구조를 가짐
- 템플릿 라이브러리이기 때문에 키와 값 모두 임의의 타입의 객체가 될 수 있음
- 연관 컨테이너에서는 보통 두 가지 종류의 질문가능
    - 특정 키가 연관 컨테이너에 존재하는지 유무
    - 특정 키에 대응되는 값이 무엇인지
    - 전자의 경우 셋(Set)과 멀티셋(Multiset), 후자의 경우 맵(Map)과 멀티맵(Multimap)사용.

### 셋(Set)

- 원소가 있냐/없냐 만이 중요한 정보
- 원소를 추가할 때 Insert를 사용하되 시퀀스 컨테이너 처럼 어디에 추가할지에 대한 정보X
- 원소를 추가하거나 지우는 작업을 O(logN)에 수행
- 셋 역시 반복자를 제공하며 BidirectionalIterator이기 때문에 순차적으로만 접근 가능
- 셋의 경우 내부에 원소를 추가할 때 정렬된 상태를 유지
- 원소가 존재하는지 판별하는 find함수 제공.
- 중복되는 원소가 없음(원소 중복을 허용하고 싶으면 MultiSet 사용)

![image.png](/assets/img/cpp/image-13.png)

- 셋은 내부적으로 트리 구조를 가지고 있음
- 트리의 규칙
    - 왼쪽에 오는 모든 노드들은 나보다 작다
    - 오른쪽에 있는 모든 노드들은 나보다 크다.
- 원소를 검색하는 횟수는 트리의 높이에 비례함.

### 맵(Map)

- 셋과 거의 똑같지만 셋은 키만 보관했고 맵은 값(Value)까지도 같이 보관
- Map의 insert는 Pair를 인자로 받고 insert를 안쓰더라도 pitcher_list["니퍼트"] = 3.56; 형식으로 정의 가능.
- 이미 키가 존재한다면 값이 대체됨.
- 셋과 마찬가지로 반복자를 통해 탐색 가능.
- itr→first를 하면 해당 원소의 키를, second를 하면 해당 원소의 값을 알 수 있음.
- map[key]로 존재하지 않는 키의 값을 호출하려고 하면 []연산자가 값의 디폴트 생성자를 호출해서 원소를 추가해버림.
    - 따라서 find로 원소가 키에 존재하는지 확인 후에 값을 참조하는게 좋음.
- 맵 또한 중복된 원소를 허락하지 않아서 나중에 오는 insert는 무시.

### 멀티셋(Multiset)과 멀티맵(Multimap)

- 중복된 원소를 허용하는 연관 컨테이너.
- 멀티맵은 한 키에 값이 여러 개일 수 있어 []연산자를 제공하지 않음
- equal_range 함수를 사용하면 키의 대응되는 원소들의 반복자 중에서 시작과 끝 바로 다음을 가리키는 반복자를 pair로 만들어서 리턴.

```cpp
for (auto itr = range.first; itr != range.second; ++itr) {
std::cout << itr->first << " : " << itr->second << " " << std::endl;
}
```

- 이런식으로 대응되는 원소들을 볼 수 있게 됨.

### 정렬되지 않은 셋과 맵(Unordered_set, Unordered_map)

- set과 map의 경우와 다르게 원소들이 순서대로 정렬되어서 들어가지 않음.
- insert, erase, find 모두 평균 O(1)으로 수행 (최악의 경우 O(N))

![image.png](/assets/img/cpp/image-14.png)

- 해시 함수(Hash Function)
    - 1부터 D까지의 값을 반환하고 그 해시값을 원소를 저장할 상자의 번호로 삼음
    - 가장 중요한 성질은 같은 원소를 해시 함수에 전달하면 같은 해시값을 리턴
    - 다른 원소임에도 같은 해시값을 갖는 경우 → 해시 충돌(Hash Collision)
        - 이 경우 해시값 계산 후에 해당하는 주소에 있는 모든 원소를 탐색해야함
    - Unordered는 O(1) ~ O(N)이지만 그냥 set과 map은 최선 최악 모두 O(logN)

# 우측값과 이동연산

## 복사 생략(Copy Elision)

- 컴파일러 자체에서 복사를 생략해 버리는 작업
- 함수 내부에서 생성된 객체를 그대로 리턴할 때 수행할 수 있음.
- 복사 생략을 할 수 도있는 것이므로 경우에 따라서는 복사 생략이 가능한 경우에 복사생략을 하지 않을 수 도 있음

## 좌측값(lVale)와 우측값(Rvalue)

```cpp
int a = 3;
```

### 좌측값(L-Value)

- a는 메모리 상에서 존재하는 변수
- a의 주소값을 &연산자를 통해 알아 낼 수 있음
- 이렇게 주소값을 취할 수있는 값을 좌측값(L_Value)라고 부름.
- 좌측값은 어떤 표현식의 왼쪽 오른쪽 모두에 올 수 있음

### 우측값(R-Value)

- 주소값을 취할 수 없는 값
- 우측값은 좌측값과 다르게 항상 식의 오른쪽에만 와야함

```cpp
int a; // a 는좌측값

int& l_a = a; // l_a 는좌측값레퍼런스
int& r_b = 3; // 3 은우측값. 따라서오류
```

- 레퍼런스는 ‘좌측값’에만 레퍼런스를 가질 수 있음
- &하나를 이용해서 정의하는 레퍼런스를 좌측값 레퍼런스(L-Value Reference)라고 부르고 좌측값 레퍼런스도 좌측값이 됨.

# 스마트 포인터

## 자원(Resource) 관리의 중요성

- C++이후에 나온 많은 언어(Java .etc)들은 대부분 가비지 컬렉터라 불리는 자원 청소기가 기본적으로 내장
- 가비지 컬렉터가 더 이상 쓰이지 않는 자원들을 자동으로 해제해주기 때문에 프로그래머들이 코드를 작성할 때 자원을 해제하는 일에 크게 신경 쓸 필요가 없음
- 하지만 C++는 자원을 직접 해제해주지 않으면 프로그램이 종료되기 전 까지 남아있음

## Resource Acquisition Is Initialization - RAII

- C++에서 자원을 관리하는 디자인 패턴
- 자원의 획득은 초기화 - Resource Acquisition Is Initialization
- 이는 자원 관리를 스택에 할당한 객체를 통해 수행하는 것.
- 예외가 발생해서 함수를 빠져나가더라도 함수의 스택에 정의되어있는 모든 객체들은 빠짐없이 소멸자가 호출됨 → Stack Unwinding

## Unique_ptr : 객체의 유일한 소유권

- C++에서 메모리를 잘못된 방식으로 관리하였을 때 크게 두 가지 종류의 문제점이 발생 가능.
    - 메모리를 사용한 후에 해제하지 않는 경우(메모리 누수 : Memory Leak)
    - 이미 해제된 메모리를 다시 참조하는 경우
- 이미 해제된 메모리를 다시 해제하면 생기는(Double free)를 방지하기 위해 어떤 포인터에 객체의 유일한 소유권을 부여한다면 객체를 두 번 소멸시키는 일은 발생하지 않음
- 이렇게 특정 객체에 유일한 소유권을 부여하는 포인터 객체를 unique_ptr이라고 함.
- Unique_ptr은 어떤 객체를 유일하게 소유해야 하기 때문에 복사 생성자가 명시적으로 삭제 됨.
- 복사는 안되지만 소유권 이전은 가능

### Unique_ptr 소유권 이전

```cpp
std::unique_ptr<A> pb = std::move(pa);
std::cout << "pb : ";
pb->some()
```

- pa를 pb에 강제로 이동시켜 버림.
- 소유권이 이전된 유니크 포인터는 nullptr 상태가 되며, 이를 역참조하면 정의되지 않은 동작이 발생.
- 소유권 이전은 이전된 포인터를 절대 다시 참조하지 않겠다는 확신 하에 이동해야 함.

### Unique_ptr정리

- 유니크 포인터는 어떤 객체의 유일한 소유권을 나타내는 포인터이며 포인터가 소멸 될 때, 가리키던 객체 역시 소멸된다.
- 다른 함수에서 유니크 포인터가 소유한 객체에 접근하고 싶으면 get을 통해 해당 객체의 포인터를 전달하면 됨.
- move를 통해 소유권 이동 가능
- make_unique를 통해 유니크 포인터를 생성할 수 있음

### 유니크 포인터를 원소로 가지는 컨테이너

- 벡터의 push_back함수는 전달된 인자를 복사해서 집어넣기 때문에 유니크 포인터를 넣으려 하면 오류 발생

```cpp
int main() 
{
	std::vector<std::unique_ptr<A>> vec
	std::unique_ptr<A> pa(new A(1));
	vec.push_back(std::move(pa)); // 잘 실행됨
}
```

- 유니크 포인터를 Vector로 이동시키면 컴파일 가능
- 혹은 emplace_back 함수를 이용하면 유니크 포인터를 생성하면서 집어넣을 수 있어서 불필요한 이동 과정 생략 가능.

```cpp

	std::vector<std::unique_ptr<A>> vec;
	
	// vec.push_back(std::unique_ptr<A>(new A(1))); 과 동일
	vec.emplace_back(new A(1));
```

## Shared_ptr

![image.png](/assets/img/cpp/image-15.png)

- 특정 자원을 몇 개의 객체에서 가리키는지를 추적한 다음에 그 수가 0이 되면 해제를 시켜줌
- 참조 카운트에 직접적으로 접근은 불가함.

![image.png](/assets/img/cpp/image-16.png)

- 포인터가 제어 블록을 동적으로 할당하고 공유 포인터들이 제어블록에 필요한 정보를 공유하는 방식.
- make_shared로 생성

```cpp
std::shared_ptr<A> p1 = std::make_shared<A>();
```

### shared_ptr 생성 시 주의점

- shared_ptr은 인자로 주소값이 전달되면 자신이 해당 객체를 첫번째로 소유하는 포인터인 것 처럼 행동함

```cpp
A* a = new A();
std::shared_ptr<A> pa1(a)
std::shared_ptr<A> pa2(a)
```

- 이 경우 두 개의 제어 블록이 따로 생성됨.

### 서로 참조하는 shared_ptr

![image.png](/assets/img/cpp/image-17.png)

- 각 객체가 포인터를 하나씩 가지고 있는데 그 포인터가 서로를 가리키는 상황에서 레퍼런스 카운트가 0이 되지 않음
- 이걸 순환 참조라고 함.

## Weak_ptr

- shared_ptr의 순환 참조 문제를 해결하기 위한 포인터.

![image.png](/assets/img/cpp/image-18.png)

- 트리를 만든다고 했을 때 부모의 타입을 shared_ptr로 하면 부모와 자식이 서로 순환 참조를 하는 문제가 발생함.
- weak_ptr은 일반 포인터와 shared_ptr사이에 위치한 스마트 포인터로 객체를 안전하게 참조할 수 있게 해주지만 공유 포인터와 달리 참조 개수를 늘리지는 않음.
- shared_ptr의 레퍼런스 카운트가 0이지만 weak_ptr이 남아있을 때 객체는 해제 되어있지만 제어 블록 마저 해제 해버리면 제어 블록에서 참조 카운트가 0이라는 사실을 알 수 없게 됨.
    - 남아있는 Weak_ptr이 객체가 해제된 사실을 알기위해 필요
- 따라서 weak count를 제어블록에서 따로 기록.

# 함수 객체

## Callable

- 호출 할 수 있는 모든 것을 의미
- ()를 붙여서 호출할 수 있는 모든  것을 Callable로 정의

## std::function

- Callable들을 객체의 형태로 보관할 수 있는 클래스

# C++ 쓰레드

## 컨텍스트 스위칭

![image.png](/assets/img/cpp/image-19.png)

- 겉으로 보기에는 여러 프로그램이 동시에 실행되는 것 처럼 보이지만 프로그램들을 빠르게 바꿔가면서 실행시키는 것.
- CPU는 운영체제가 처리하라고 하는 명령어를 실행.
- 어떤 프로그램을 얼마 실행시키고 스위치 할 프로그램을 결정하는 것은 운영체제의 스케쥴러(Scheduler)임

## 쓰레드

- CPU 코어에서 돌아가는 프로그램 단위를 쓰레드(Thread)라고 부름
- 하나의 CPU의 코어에서는 한 번에 한 개의 쓰레드 명령을 실행
- 한 개의 프로세스는 최소 한 개의 쓰레드로 이루어져 있으며, 여러 개의 쓰레드로 구성 될 수 도 있음
- 여러개의 쓰레드로 구성된 프로그램을 **멀티 쓰레드 프로그램** 이라 함.
- 프로세스들은 서로 메모리를 공유하지 않지만 같은 프로세스 안의 쓰레드들은 서로 메모리를 공유.

![image.png](/assets/img/cpp/image-20.png)

## 멀티 쓰레드를 사용하는 이유

### 병렬가능한(Parallelizable)작업들

![image.png](/assets/img/cpp/image-21.png)

- 1부터 1000까지 더하는 작업같은 경우 단일 쓰레드에서 처리하는 것 보다 멀티 쓰레드를 사용하는게 훨씬 속도가 빠름.
- 여러개의 다른 쓰레드로 작업을 좀 더 빠르게 수행하는 것을 병렬화(Parallelize)라고 함.
- 어떠한 연산 A를 수행하기 위해 다른 연산 B의 결과가 필요한 의존 관계의 경우 병렬화가 어려움.

### 대기시간이 긴 작업들.

- 인터넷에서 무언가를 다운로드 받는 프로그램을 만든다고 했을 때 CPU의 처리 속도에 비해 인터넷은 매우 느림.
- 흔히 Ping이라고 부르는 내가 보낸 요청이 상대에게 도착하고 다시 나에게 돌아오는 시간이 환경에 따라 달라짐.
- 이를 하나의 쓰레드에서 처리하면 대기 시간이 길어 CPU가 비 효율적임

![image.png](/assets/img/cpp/image-22.png)

- 하지만 멀티 쓰레드를 사용하면 기다리는 시간 낭비를 줄일 수 있음

![image.png](/assets/img/cpp/image-23.png)

## C++에서 쓰레드 생성하기

- C++ 11에서부터 표준에 쓰레드가 추가됨.

```cpp
// 내생에첫쓰레드
#include <iostream>
#include <thread>

using std::thread;

void func1() 
{
	for (int i = 0; i < 10; i++) 
	{
		std::cout << "쓰레드1 작동중! \n";
	}
}
void func2() 
{
	for (int i = 0; i < 10; i++) 
	{
		std::cout << "쓰레드2 작동중! \n";
	}
}

void func3() 
{
	for (int i = 0; i < 10; i++) 
	{
		std::cout << "쓰레드3 작동중! \n";
	}
}

int main() 
{
	thread t1(func1);
	thread t2(func2);
	thread t3(func3);
	t1.join();
	t2.join();
	t3.join();
}
```

- 쓰레드들이 코어에 할당되고 언제 컨텍스트 스위치를 할 지는 운영체제가 결정.
- join은 쓰레드들이 실행을 종료하면 리턴하는 함수.
- 쓰레드를 detach하면 main함수에서 더이상 쓰레드들이 종료될 때 까지 기다리지 않음.

```cpp
thread(worker, data.begin() + i * 2500, data.begin() + (i + 1) * 2500,
&partial_sums[i])
```

- 쓰레드의 인자 전달 → 쓰레드 생성자의 첫 번째 인자로 함수를 넣고 그 뒤에 인자를 쓰면 됨.

# 뮤텍스와 조건변수

## 경쟁 상태(Race Condition)

- 서로 다른 쓰레드들이 동일한 자원을 사용할 때 발생하는 문제
- 특정 작업을 여러 쓰레드에서 동시에 실행시키면 값들이 달라 질 수 있음.

## 뮤텍스

- 특정 작업을 한 번에 한 쓰레드 위에서만 실행하도록 하는 객체

```cpp
std::mutex m; // 우리의mutex 객체
void worker(int& result, std::mutex& m)
{
	m.lock();
	result += 1;
	m.unlock();
}

```

- m.lock()은 뮤텍스 m을 사용하도록 요청하는 것.
- 한 번에 한 쓰레드에서만 m의 사용 권한을 갖기 때문에 다른 쓰레드에서 m.unlock()을 통해 m을 반환할 때 까지 대기.
- lock과 unlock사이에 한 쓰레드만이 유일하게 실행 할 수 있는 코드 부분을 **임계 영역(Critical Section)**이라 함.

## 데드락(Deaklock)

- lock()으로 뮤텍스를 취득하고 반환하지 않아 lock을 호출한 다른 쓰레드들이 기다리는 현상.
- 모든 쓰레드들이 연산을 하지 못하게 되므로 취득한 뮤텍스는 반드시 반환해야함.

```cpp
std::lock_guard<std::mutex> lock1(m1);
std::lock_guard<std::mutex> lock2(m2);

std::lock_guard<std::mutex> lock2(m2);
std::lock_guard<std::mutex> lock1(m1);
```

- 서로 뮤텍스를 점거하고 다른 쓰레드가 소유한 뮤텍스를 요청하면 두 쓰레드 모두 멈춤.

## 기아 상태

- 한 쓰레드가 다른 쓰레드에 비해 우위를 갖게 되었을 때 한 쓰레드만 열심히 일하고 다른 쓰레드는 일할 수 없는 상태.
- C++에서 제공하는 try_lock() 함수는 lock을 할 수 있다면 true를 반환, 불가능 할 때는 기다리지 않고 false를 반환.

## 데드락을 피하기 위한 가이드라인

### 중첩된 Lock을 사용하는 것을 피하기

- 모든 쓰레드들이 최대 1개의 Lock만을 소유하면 데드락 상황이 발생하는 것을 피할 수 있음

### Lock을 소유하고 있을 때 유저 코드를 호출하는 것을 피해라

- 유저 코드에서 lock을 소유할 수 도 있기 때문에 중첩된 Lock을 얻는 것을 피하려면 Lock 소유시 유저 코드를

### Lock들을 언제나 정해진 순서로 획득해라

- lock을 획득하는 순서가 다르면 데드락이 발생할 수 있음

## 생산자(Producer)와 소비자(Comsumer)패턴

- 생산자는 무언가 처리할 일을 받아오는 쓰레드를 의미
- 소비자는 받은 일을 처리하는 쓰레드를 의미

![image.png](/assets/img/cpp/image-24.png)

- Producer에서 데이터가 간격을 두고 오는 것을 안다면 Producer가 데이터를 줄 때 Consumer를깨우는 것이 효율적

## Condition_variable

```cpp
std::condition_variable cv;

std::unique_lock<std::mutex> lk(*m);
cv->wait(lk,
[&] { return !downloaded_pages->empty() || *num_processed == 25; });
```

- condition_variable의 wait함수에 어떤 조건이 참이 될 때 까지 기다릴지 해당 조건을 인자로 전달.
- 조건 변수는 해당 조건이 거짓이면 unlock후 sleep함.

## Atomic객체와 명령어 재배치

### 메모리는 엄청 느리다

- CPU와 컴퓨터 메모리인 RAM은 물리적으로 떨어져 있음
- 따라서 CPU가 메모리에서 데이터를 읽어 오기 위해서는 꽤 많은 시간이 걸림.

### 캐시

![image.png](/assets/img/cpp/image-25.png)

- 이를 보완하기 위해 캐시(Cache)도입.
- 캐시는 CPU에서 연산을 수행하는 부분과 거의 붙어있어 읽기/쓰기 속도가 매우 빠름
- 캐시의 크기는 그리 크지 않고 L1이 제일 크기가 작고 L3가 제일 크기가 큼
- 속도는 L1이 제일 빠르고 L3가 제일 느림.
- 따라서 가장 많이 접근하는 메모리는 L1부터 L3까지 순서대로 적재.