---
title: "C++ const / constexpr / inline 차이"
date: 2025-07-29 16:05:00 +0900
categories: [C++, Keyword]
tags: [c++, const, constexpr, inline, compile-time]
description: "C++에서 const, constexpr, inline 키워드의 차이와 활용법을 예제와 함께 정리한다"
---

# const / constexpr / inline 차이

C++ 키워드 중에서 헷갈리기 쉬운 3총사가 있다. 바로 **const**, **constexpr**, **inline**이다.  
셋 다 "최적화"나 "상수 처리"와 관련된 것 같지만, 실제로는 서로 다른 목적을 가진다.  
이번 글에서는 세 키워드의 의미와 차이를 하나씩 살펴본다.

---

## 1. const: 변경 불가능한 값

`const`는 단순하다. 변수나 포인터, 멤버 함수 등에 붙여서 **값을 바꿀 수 없게** 만든다.

```cpp
const int a = 10;
// a = 20; // 에러
````

### const와 포인터

```cpp
const int* p1 = &a;   // 포인터가 가리키는 값 변경 불가
int* const p2 = &a;   // 포인터 자체 변경 불가
const int* const p3 = &a; // 둘 다 불가
```

### const 멤버 함수

```cpp
class Player {
    int hp;
public:
    int getHP() const { return hp; }
};
```

여기서 `getHP`는 멤버 변수를 수정할 수 없는 함수가 된다.

---

## 2. constexpr: 컴파일 타임 상수

`constexpr`은 **컴파일 시점에 값이 확정되는 상수**를 의미한다.
`const`는 런타임에만 고정되는 값일 수도 있지만, `constexpr`은 반드시 컴파일 타임에 계산 가능해야 한다.

```cpp
constexpr int square(int x) {
    return x * x;
}

int arr[square(5)]; // OK, 크기가 25로 컴파일 타임에 확정
```

### const vs constexpr

* `const` → 변경 불가, 하지만 컴파일 타임 상수는 아닐 수도 있음
* `constexpr` → 반드시 컴파일 타임 상수

```cpp
const int x = rand();     // OK, 하지만 런타임 값
constexpr int y = 10;     // 컴파일 타임 상수
```

---

## 3. inline: 함수 호출 최적화

`inline`은 함수 호출을 줄이기 위해 함수 본문을 **호출 위치에 직접 삽입**하라는 힌트이다.

```cpp
inline int add(int a, int b) {
    return a + b;
}
```

하지만 요즘 컴파일러는 스스로 인라인 여부를 최적화하므로, `inline` 키워드는 거의 "링커에서 중복 정의를 허용하는 용도"로 쓰인다.

예를 들어, 헤더 파일에 함수를 정의할 때 `inline`을 붙여야 ODR(One Definition Rule) 위반을 피할 수 있다.

```cpp
// util.h
inline int multiply(int a, int b) {
    return a * b;
}
```

---

## 4. 세 키워드 비교

| 키워드         | 의미                  | 시점     | 주요 용도              |
| ----------- | ------------------- | ------ | ------------------ |
| `const`     | 값 변경 불가             | 런타임    | 불변 변수, 상수 참조       |
| `constexpr` | 컴파일 타임 상수           | 컴파일 타임 | 상수 표현식, 배열 크기, 최적화 |
| `inline`    | 함수 호출 대체(힌트), 중복 허용 | 컴파일·링크 | 작은 함수 최적화, 헤더 정의   |

---

## 5. 게임 개발에서의 활용

언리얼 엔진에서 상수 값을 정의할 때 `constexpr`을 활용하면 성능 최적화에 도움이 된다.
예를 들어, 배열 크기나 고정된 매직 넘버 같은 값들을 런타임에 굳이 계산하지 않고 컴파일 타임에 확정시킬 수 있다.

```cpp
constexpr int MaxHealth = 100;
```

또한, 자주 호출되는 짧은 수학 함수(예: `Clamp`, `Lerp`)는 `FORCEINLINE` 매크로를 붙여 헤더에서 정의되곤 한다.
이는 사실상 `inline`과 같은 원리다.

---

## 6. 정리

* `const`는 값의 변경을 막는다.
* `constexpr`는 **컴파일 타임 상수**를 보장한다.
* `inline`은 호출 오버헤드를 줄이거나 헤더 함수 정의에서 중복을 막는다.

> 결론: `const`는 “불변”, `constexpr`는 “컴파일 타임 확정”, `inline`은 “최적화 힌트”라고 기억하면 된다.
> 이 셋을 구분해서 쓰면 코드 안정성과 성능 모두 챙길 수 있다.
