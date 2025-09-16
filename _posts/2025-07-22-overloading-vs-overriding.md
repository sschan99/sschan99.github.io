---
title: "C++ 함수 오버로딩과 오버라이딩 차이"
date: 2025-07-22 11:40:00 +0900
categories: [C++, OOP]
tags: [c++, overloading, overriding, polymorphism]
description: "C++에서 함수 오버로딩과 오버라이딩을 비교하고 실제 코드 예제로 차이를 설명한다"
---

# 함수 오버로딩과 오버라이딩 차이

C++을 공부하다 보면 헷갈리기 쉬운 개념이 있다. 바로 **함수 오버로딩(overloading)**과 **함수 오버라이딩(overriding)**이다.  
이 두 용어는 발음도 비슷하고, 둘 다 "함수를 새롭게 정의한다"라는 느낌을 주기 때문에 자주 혼동된다.  
하지만 실제로는 완전히 다른 개념이다. 이번 글에서는 이 차이를 명확히 정리해보겠다.

---

## 1. 함수 오버로딩 (Function Overloading)

오버로딩은 **동일한 함수 이름을 여러 번 정의**하는 것이다.  
단, 매개변수의 **타입**이나 **개수**가 달라야 한다. 반환 타입만 달라서는 안 된다.  

```cpp
#include <iostream>
using namespace std;

int add(int a, int b) {
    return a + b;
}

double add(double a, double b) {
    return a + b;
}

int main() {
    cout << add(1, 2) << endl;       // int 버전
    cout << add(1.2, 3.4) << endl;   // double 버전
}
````

컴파일러는 호출 시점의 매개변수 타입을 보고 어떤 함수를 호출할지 결정한다.
즉, 오버로딩은 **컴파일 타임에 결정되는 정적 바인딩**이다.

---

## 2. 함수 오버라이딩 (Function Overriding)

오버라이딩은 **상속 관계에서 부모 클래스의 함수를 자식 클래스에서 재정의**하는 것이다.
이때 함수의 \*\*시그니처(이름, 매개변수, 반환 타입)\*\*가 동일해야 한다.

```cpp
#include <iostream>
using namespace std;

struct Animal {
    virtual void speak() { cout << "???" << endl; }
};

struct Dog : Animal {
    void speak() override { cout << "멍멍" << endl; }
};

int main() {
    Animal* a = new Dog();
    a->speak(); // 멍멍
}
```

여기서 `Dog::speak`는 `Animal::speak`를 오버라이딩한다.
포인터 타입은 `Animal*`이지만, 실제 객체가 `Dog`이므로 동적 바인딩을 통해 `Dog::speak`가 실행된다.

---

## 3. 정적 바인딩 vs 동적 바인딩

핵심 차이는 함수 호출이 언제 결정되느냐이다.

| 구분    | 오버로딩(Overloading) | 오버라이딩(Overriding)     |
| ----- | ----------------- | --------------------- |
| 관계    | 같은 클래스 내부         | 상속 관계                 |
| 조건    | 매개변수 시그니처 달라야 함   | 함수 시그니처 동일해야 함        |
| 결정 시점 | 컴파일 타임            | 런타임                   |
| 키워드   | 없음                | `virtual`, `override` |

---

## 4. 오버라이딩에서 주의할 점

1. **virtual 키워드**
   부모 클래스 함수가 `virtual`이 아니면 오버라이딩이 아니라 단순한 함수 숨김(hiding)이 된다.

2. **소멸자**
   부모 클래스 소멸자는 반드시 `virtual`로 선언하는 것이 안전하다.
   그렇지 않으면 `delete`할 때 자식 소멸자가 호출되지 않아 리소스 누수가 발생할 수 있다.

```cpp
struct Base {
    virtual ~Base() { cout << "Base 소멸자" << endl; }
};
struct Derived : Base {
    ~Derived() { cout << "Derived 소멸자" << endl; }
};
```

3. **override 키워드**
   C++11 이후 추가된 `override`는 오타나 시그니처 불일치 같은 실수를 잡아준다.

---

## 5. 게임 개발에서의 사례

언리얼 엔진을 다루다 보면 오버라이딩을 굉장히 자주 쓰게 된다.
예를 들어 `AActor` 클래스를 상속받아 커스텀 캐릭터를 만들 때, `Tick`이나 `BeginPlay`를 오버라이딩하는 식이다.

```cpp
class AMyCharacter : public ACharacter {
protected:
    virtual void BeginPlay() override;
    virtual void Tick(float DeltaTime) override;
};
```

이처럼 오버라이딩은 **게임 객체의 동작을 커스터마이즈하는 기본 수단**이다.
반면, 오버로딩은 편의 기능을 추가하거나 같은 연산을 다양한 타입에서 제공할 때 자주 쓰인다.

---

## 6. 정리

* 오버로딩: 같은 이름, 다른 매개변수. **컴파일 타임 결정**
* 오버라이딩: 부모 함수 재정의. **런타임 결정**
* 둘 다 코드 재사용성을 높여주지만, 쓰임새와 메커니즘은 완전히 다르다.

> 결론: 오버로딩은 "컴파일러가 똑똑해지는 기능"이고, 오버라이딩은 "객체가 똑똑해지는 기능"이다.
> 둘을 명확히 구분해서 이해하면 객체지향 프로그래밍에서 훨씬 자유롭게 코드를 다룰 수 있다.
