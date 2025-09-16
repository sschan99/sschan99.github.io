---
title: "C++ 가상 함수와 vtable 동작 원리"
date: 2025-07-08 10:20:00 +0900
categories: [C++, OOP]
tags: [c++, virtual, vtable, polymorphism]
description: "C++의 가상 함수와 vtable 동작 방식에 대해 예제와 그림을 곁들여 정리"
---

# C++ 가상 함수와 vtable 동작 원리

객체지향 프로그래밍을 이야기할 때 빠지지 않는 키워드가 있다. 바로 **다형성(polymorphism)**이다.  
같은 인터페이스를 호출했는데, 실제로는 어떤 객체냐에 따라 전혀 다른 동작을 할 수 있는 능력 말이다.  
C++에서는 이 다형성을 구현하기 위해 **가상 함수(virtual function)**와 **vtable**이라는 메커니즘을 사용한다.  
오늘은 이 개념을 차근차근 풀어본다.

---

## 1. 가상 함수란 무엇인가

기본적으로 C++에서 함수 호출은 **정적 바인딩(static binding)**이다.  
즉, 어떤 함수를 호출할지는 **컴파일 타임**에 결정된다.  

하지만 가상 함수를 사용하면, 호출할 함수가 **런타임에 결정**된다. 이것이 곧 다형성이다.  

### 예제 코드
```cpp
#include <iostream>
using namespace std;

struct Animal {
    virtual void speak() { cout << "???" << endl; }
};

struct Dog : Animal {
    void speak() override { cout << "멍멍" << endl; }
};

struct Cat : Animal {
    void speak() override { cout << "야옹" << endl; }
};

int main() {
    Animal* a1 = new Dog();
    Animal* a2 = new Cat();

    a1->speak(); // 멍멍
    a2->speak(); // 야옹
}
````

여기서 `a1`, `a2`는 `Animal*` 타입이지만, 실제 실행 결과는 각각 `Dog`, `Cat`의 `speak`가 불린다.
이게 바로 런타임 다형성이다.

---

## 2. vtable의 존재

가상 함수는 내부적으로 \*\*가상 함수 테이블(vtable)\*\*이라는 구조를 이용한다.
컴파일러는 각 클래스마다 **vtable**을 만들고, 가상 함수의 주소를 여기에 저장한다.

* `Animal`의 vtable → `Animal::speak` 주소
* `Dog`의 vtable → `Dog::speak` 주소
* `Cat`의 vtable → `Cat::speak` 주소

그리고 각 객체는 \*\*vptr(가상 테이블 포인터)\*\*라는 숨겨진 포인터를 가진다.
이 vptr이 자신이 속한 클래스의 vtable을 가리키고 있다.

따라서, `a1->speak()`를 호출하면:

1. 객체 내부의 vptr 확인
2. vtable에서 `speak` 함수 주소 조회
3. 해당 주소로 점프

이런 과정을 거쳐서 올바른 함수가 실행되는 것이다.

---

## 3. 그림으로 이해하기

> 간단한 도식 (텍스트 표현)

```
Animal 객체 (vptr) → [ vtable: { speak → Animal::speak } ]
Dog 객체 (vptr)    → [ vtable: { speak → Dog::speak } ]
Cat 객체 (vptr)    → [ vtable: { speak → Cat::speak } ]
```

각 객체가 어떤 vtable을 가리키느냐에 따라 호출 결과가 달라진다.
이 구조 덕분에 포인터 타입이 부모 클래스여도 올바른 자식 클래스의 함수가 실행된다.

---

## 4. 가상 함수의 특징과 주의점

1. **오버라이딩만 가능하다**
   → 같은 시그니처로 다시 정의해야 의미가 있다.

2. **생성자에서는 호출이 제한된다**
   → 생성자 내부에서 가상 함수를 호출하면, 현재 클래스의 함수만 실행된다. 아직 자식 객체가 완전히 생성되지 않았기 때문이다.

3. **소멸자에는 virtual을 붙여야 한다**
   → 부모 포인터로 자식 객체를 delete할 때, 가상 소멸자가 없으면 메모리 누수가 발생할 수 있다.

```cpp
struct Base {
    virtual ~Base() { cout << "Base 소멸자\n"; }
};
struct Derived : Base {
    ~Derived() { cout << "Derived 소멸자\n"; }
};
```

---

## 5. 성능에 대한 생각

가상 함수 호출은 일반 함수 호출보다 약간의 오버헤드가 있다.
왜냐하면 **vtable을 거쳐서 함수 주소를 찾는 과정**이 필요하기 때문이다.

하지만 이 비용은 보통 매우 작다. 오히려 유지보수성과 확장성이 훨씬 큰 이득을 준다.
그래서 성능이 극도로 중요한 루프 안에서가 아니라면, 가상 함수 사용을 두려워할 필요는 없다.

---

## 6. 정리

* 가상 함수는 C++ 다형성을 구현하는 핵심 장치이다.
* 내부적으로 vtable과 vptr을 이용해 런타임에 올바른 함수를 찾는다.
* 소멸자에는 반드시 `virtual`을 붙여야 안전하다.
* 오버헤드는 존재하지만 대부분의 상황에서 감수할 만하다.

> 결론: 가상 함수와 vtable은 “동적 바인딩”을 가능하게 하는 C++의 비밀 무기다.
> 덕분에 하나의 인터페이스로 여러 객체를 유연하게 다룰 수 있고, 이는 곧 객체지향 프로그래밍의 핵심 가치와 맞닿아 있다.
