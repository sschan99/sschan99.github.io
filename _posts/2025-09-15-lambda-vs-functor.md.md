---
title: "C++ Lambda와 함수 객체 비교"
date: 2025-09-15 21:20:00 +0900
categories: [C++, Functional]
tags: [c++, lambda, functor, functional-programming]
description: "C++ 함수 객체(Functor)와 Lambda의 차이, 공통점, 그리고 실제 활용 사례를 정리한다"
---

# Lambda와 함수 객체 비교

C++은 함수도 값처럼 다룰 수 있는 언어이다.  
이 기능을 제공하는 두 가지 대표적인 방법이 바로 **함수 객체(Functor)**와 **람다(Lambda)**이다.  
둘은 비슷해 보이지만 태생과 쓰임새가 다르다. 이번 글에서는 둘을 비교해보겠다.

---

## 1. 함수 객체(Functor)

함수 객체는 **함수 호출 연산자 `operator()`를 오버로딩한 클래스**이다.  
즉, "객체인데 함수처럼 동작하는 것"이다.

```cpp
#include <iostream>
using namespace std;

struct Adder {
    int operator()(int a, int b) const {
        return a + b;
    }
};

int main() {
    Adder add;
    cout << add(3, 4) << endl; // 7
}
````

함수 객체의 장점은 상태(state)를 가질 수 있다는 점이다.
즉, 내부 멤버 변수를 활용해 더 복잡한 동작을 만들 수 있다.

```cpp
struct Counter {
    int count = 0;
    int operator()(int x) {
        return count += x;
    }
};
```

---

## 2. 람다 함수(Lambda)

람다는 C++11에서 도입된 문법으로, \*\*익명 함수(Anonymous Function)\*\*를 손쉽게 정의할 수 있게 해준다.

```cpp
auto add = [](int a, int b) { return a + b; };
cout << add(3, 4) << endl; // 7
```

람다의 특징은 **캡처(capture)** 문법을 통해 외부 변수를 함수 안으로 끌어올 수 있다는 것이다.

```cpp
int base = 10;
auto addBase = [base](int x) { return base + x; };
cout << addBase(5) << endl; // 15
```

---

## 3. Functor vs Lambda

| 구분    | 함수 객체(Functor)                  | 람다(Lambda)           |
| ----- | ------------------------------- | -------------------- |
| 정의 방식 | `struct`/`class` + `operator()` | `[](...) { ... }` 문법 |
| 상태 유지 | 멤버 변수 가능                        | 캡처 리스트로 가능           |
| 가독성   | 길고 장황함                          | 짧고 간결함               |
| 재사용성  | 타입으로 명확히 정의됨                    | 1회성/익명성이 강함          |

👉 간단한 일회성 동작에는 람다가 적합하고, 여러 번 재사용하거나 복잡한 상태를 유지해야 한다면 Functor가 더 낫다.

---

## 4. 실제 활용 예시

### STL 알고리즘

STL의 `std::sort`는 비교 함수를 필요로 한다.
Functor나 람다 모두 사용 가능하다.

```cpp
#include <algorithm>
#include <vector>

struct Compare {
    bool operator()(int a, int b) const {
        return a > b;
    }
};

int main() {
    vector<int> v = {3, 1, 4, 1, 5};

    // Functor 사용
    sort(v.begin(), v.end(), Compare());

    // Lambda 사용
    sort(v.begin(), v.end(), [](int a, int b) { return a > b; });
}
```

람다가 등장한 이후로는 대부분 람다를 쓰는 추세다.

---

## 5. 게임 개발에서의 활용

게임 코드에서는 이벤트 핸들러나 콜백을 등록할 때 람다가 자주 쓰인다.

예를 들어, 언리얼 엔진의 `TFunction`이나 `FDelegate`에 람다를 연결하면 매우 간단하게 코드를 작성할 수 있다.

```cpp
Button->OnClicked.AddLambda([]() {
    UE_LOG(LogTemp, Warning, TEXT("Button clicked!"));
});
```

Functor는 상대적으로 잘 안 쓰이지만, **커스텀 정렬**이나 **특수한 정책 객체**를 정의할 때 여전히 강력하다.

---

## 6. 정리

* Functor: 객체 지향적인 함수. 상태를 가질 수 있음.
* Lambda: 간결하고 가독성이 좋은 익명 함수. 캡처 기능 제공.
* 둘 다 STL 알고리즘과 잘 어울린다.

> 결론: 현대 C++에서는 람다가 Functor의 자리를 대부분 대체했다.
> 하지만 Functor의 장점인 "상태 유지" 능력은 여전히 필요할 때가 있다.
> 결국 상황에 맞게 선택하는 것이 중요하다.
