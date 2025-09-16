---
title: "C++ 템플릿 함수와 클래스"
date: 2025-07-15 14:10:00 +0900
categories: [C++, Template]
tags: [c++, template, generic, stl]
description: "C++의 제네릭 프로그래밍을 가능하게 하는 템플릿 함수와 클래스 개념을 정리하고 실제 사용 예시를 다룬다"
---

# C++ 템플릿 함수와 클래스

C++의 가장 큰 매력 중 하나는 **제네릭 프로그래밍(Generic Programming)**을 지원한다는 점이다.  
하나의 코드로 여러 타입에 대해 동작할 수 있도록 해주는 기능이 바로 **템플릿(template)**이다.  
템플릿은 STL의 핵심이자, 현대 C++ 프로그래밍을 지탱하는 기둥과 같은 존재이다.  

---

## 1. 템플릿 함수의 기본

템플릿 함수는 특정 타입에 의존하지 않고, 타입을 매개변수처럼 받아서 함수의 동작을 정의한다.

```cpp
#include <iostream>
using namespace std;

template<typename T>
T add(T a, T b) {
    return a + b;
}

int main() {
    cout << add<int>(1, 2) << endl;       // 3
    cout << add<double>(1.5, 2.3) << endl; // 3.8
}
````

여기서 `T`는 **타입 매개변수**이다.
컴파일러는 실제로 `add<int>`, `add<double>` 같은 **구체화된 함수**를 생성한다.
즉, 템플릿 함수는 **컴파일 타임에 코드가 생성**된다.

---

## 2. 타입 추론(Type Deduction)

C++ 컴파일러는 함수 호출 시 매개변수 타입을 보고 템플릿의 타입을 자동으로 추론한다.

```cpp
cout << add(10, 20);     // int로 추론
cout << add(1.2, 3.4);   // double로 추론
```

이 덕분에 코드가 훨씬 간결해진다. 다만, 서로 다른 타입을 혼합하면 에러가 난다.

```cpp
add(1, 2.5); // 컴파일 에러 (int와 double 혼합)
```

이런 경우 `auto`, `decltype` 또는 `std::common_type` 같은 트릭으로 해결할 수 있다.

---

## 3. 템플릿 클래스

함수뿐만 아니라 클래스도 템플릿으로 정의할 수 있다.
대표적인 예가 `std::vector<T>`이다.

```cpp
template<typename T>
class Box {
    T value;
public:
    Box(T v) : value(v) {}
    T get() { return value; }
};
```

사용 예:

```cpp
Box<int> b1(42);
Box<string> b2("Hello");

cout << b1.get() << endl;  // 42
cout << b2.get() << endl;  // Hello
```

하나의 클래스 정의로 여러 타입의 객체를 생성할 수 있다는 점이 강력하다.

---

## 4. 비타입(non-type) 템플릿 매개변수

템플릿에는 타입뿐 아니라 값도 전달할 수 있다. 이를 **비타입 템플릿 매개변수**라고 한다.

```cpp
template<int N>
int getValue() {
    return N;
}

int main() {
    cout << getValue<10>() << endl; // 10
}
```

STL의 `std::array<T, N>`도 이런 방식으로 구현되어 있다.
`N`은 배열의 크기이며, 이 값은 컴파일 타임에 결정된다.

---

## 5. 부분 특수화(Partial Specialization)

템플릿은 때때로 특정 타입에 대해 다른 동작을 정의하고 싶을 때가 있다.
이럴 때 \*\*특수화(specialization)\*\*를 사용한다.

```cpp
template<typename T>
struct Printer {
    void print(T value) {
        cout << value << endl;
    }
};

// 특수화: char* 전용
template<>
struct Printer<char*> {
    void print(char* value) {
        cout << "문자열: " << value << endl;
    }
};
```

이 방식으로 기본 템플릿의 동작을 커스터마이즈할 수 있다.

---

## 6. 템플릿과 현대 C++

C++11 이후로는 `auto`, `decltype`, `variadic template` 같은 기능이 추가되면서 템플릿이 훨씬 강력해졌다.

```cpp
template<typename... Args>
void printAll(Args... args) {
    (cout << ... << args) << endl; // fold expression
}

int main() {
    printAll(1, "text", 3.14); // 1text3.14
}
```

이런 기능 덕분에 템플릿은 사실상 **범용 프로그래밍 언어 안에 또 다른 메타 프로그래밍 언어**처럼 활용된다.

---

## 7. 게임 개발에서의 활용

언리얼 엔진의 C++ API에서도 템플릿이 많이 쓰인다.
예를 들어 `TArray<T>`, `TMap<Key, Value>` 같은 자료구조들이 그렇다.
또한 `TSubclassOf<T>` 같은 매크로 기반의 템플릿 타입도 자주 마주친다.

게임 코드에서 타입 안정성과 재사용성을 확보하기 위해 템플릿은 필수적이다.

---

## 8. 정리

* 템플릿은 **제네릭 프로그래밍**을 가능하게 한다.
* 함수와 클래스 모두 템플릿으로 만들 수 있다.
* 타입 매개변수뿐 아니라 값(비타입 매개변수)도 전달할 수 있다.
* STL과 언리얼 엔진을 포함해, 현대 C++의 거의 모든 라이브러리가 템플릿 위에 세워져 있다.

> 결론: 템플릿은 단순한 문법 요소가 아니라 C++의 핵심 도구이다.
> 잘 활용하면 코드 중복을 없애고, 안정적이며 강력한 추상화를 이끌어낼 수 있다.
