---
title: "C++ 예외 처리와 noexcept"
date: 2025-09-05 10:55:00 +0900
categories: [C++, Exception]
tags: [c++, exception, noexcept, try-catch, error-handling]
description: "C++ 예외 처리 방식과 noexcept 키워드의 의미, 그리고 게임 개발에서의 활용을 정리한다"
---

# 예외 처리와 noexcept

C++은 성능 지향 언어이지만, 동시에 안전한 에러 처리도 지원한다.  
그 핵심 도구가 **예외 처리(exception handling)**이고, C++11 이후에는 이를 명시적으로 표현할 수 있는 **noexcept** 키워드가 추가되었다.  
이번 글에서는 예외 처리 기본부터 noexcept까지 정리한다.

---

## 1. 기본 예외 처리 구조

예외 처리는 **try-catch** 블록으로 이루어진다.

```cpp
#include <iostream>
#include <stdexcept>
using namespace std;

int divide(int a, int b) {
    if (b == 0) throw runtime_error("0으로 나눌 수 없음");
    return a / b;
}

int main() {
    try {
        cout << divide(10, 0) << endl;
    } catch (const exception& e) {
        cout << "에러: " << e.what() << endl;
    }
}
````

출력:

```
에러: 0으로 나눌 수 없음
```

예외는 함수 호출 스택을 타고 전파되며, 적절한 `catch` 블록에서 처리된다.

---

## 2. 예외 처리의 장점과 단점

### 장점

* 에러 처리를 코드 흐름과 분리할 수 있다.
* 정상 동작 코드와 예외 처리 코드가 깔끔히 구분된다.
* 다양한 타입의 예외를 던지고 받을 수 있다.

### 단점

* 예외 전파 비용이 크다.
* 성능에 민감한 코드(예: 게임 루프)에서는 부담이 될 수 있다.
* 과도하게 사용하면 코드 복잡성이 올라간다.

---

## 3. noexcept 키워드

C++11에서 도입된 **noexcept**는 함수가 예외를 던지지 않음을 약속하는 키워드다.

```cpp
void foo() noexcept {
    // 이 함수는 예외를 던지지 않는다
}
```

### noexcept의 효과

1. **최적화**: 컴파일러가 더 효율적인 코드를 생성할 수 있다.
2. **명시성**: API 사용자에게 이 함수가 안전하다는 신뢰를 준다.
3. **예외 전파 차단**: 만약 `noexcept` 함수에서 예외가 던져지면 `std::terminate`가 호출된다.

---

## 4. 예제: noexcept와 move semantics

표준 라이브러리에서 `noexcept`는 특히 **move semantics**와 관련이 깊다.
컨테이너가 원소를 재배치할 때, move 연산자가 `noexcept`이면 복사 대신 이동을 사용한다.

```cpp
struct MyData {
    MyData(const MyData&) { cout << "copy\n"; }
    MyData(MyData&&) noexcept { cout << "move\n"; }
};

int main() {
    vector<MyData> v;
    v.reserve(2);
    v.push_back(MyData());
    v.push_back(MyData()); // move 호출
}
```

`noexcept`가 없었다면 컴파일러는 안전을 위해 복사를 선택했을 것이다.

---

## 5. 게임 개발에서의 활용

게임 루프는 1프레임이라도 성능 저하가 있으면 눈에 띈다.
따라서 보통 엔진 코어는 예외 대신 **에러 코드 반환**을 선호한다.
하지만 툴링, 에디터, 테스트 코드 등에서는 예외가 유용하다.

`noexcept`는 특히 네트워크 패킷 처리, 메모리 관리 같은 저수준 함수에서 중요하다.
불필요한 예외 전파를 막고 성능을 안정화하는 데 기여한다.

---

## 6. 정리

* 예외 처리: `try`로 감싸고 `throw`로 던지며 `catch`로 받는다.
* 장점: 깔끔한 에러 처리. 단점: 성능 비용이 크다.
* `noexcept`: 함수가 예외를 던지지 않음을 보장, 최적화와 신뢰성에 기여한다.
* 게임 개발에서는 성능과 안정성을 모두 고려해 **필요한 곳에만 예외를 쓰고**, 코어 루프는 에러 코드 방식으로 처리하는 경우가 많다.

> 결론: 예외는 "필요할 때만" 쓰는 도구이고, `noexcept`는 그 약속을 코드로 새겨넣는 장치다.
