---
title: "C++ RAII와 스마트 포인터"
date: 2025-07-01 09:00:00 +0900
categories: [C++, Memory]
tags: [c++, raii, smart-pointer, memory-management]
description: "C++에서 자원 관리의 핵심인 RAII 개념과 스마트 포인터 활용법을 예제 코드와 함께 정리"
---

# C++ RAII와 스마트 포인터

C++을 공부하다 보면 자주 듣게 되는 말이 있다. 바로 **RAII(Resource Acquisition Is Initialization)**이다.  
처음 들으면 다소 생소하지만, 사실상 C++ 프로그래밍 전체를 관통하는 중요한 철학이다.  
이 글에서는 RAII라는 개념이 무엇인지, 그리고 그것을 실현하는 도구인 **스마트 포인터(smart pointer)**에 대해 정리한다.

---

## 1. RAII란 무엇인가

RAII는 간단히 말하면 **객체의 수명과 자원의 수명을 묶는 방식**이다.  

- 객체가 생성될 때 → 자원을 획득한다.  
- 객체가 소멸될 때 → 자원을 해제한다.  

즉, 자원의 관리 책임을 **생성자와 소멸자**에게 맡겨버리는 것이다.  

### 전통적인 자원 관리 문제
```cpp
FILE* f = fopen("data.txt", "r");
if (!f) return -1;
// 파일 사용...
fclose(f);
````

위 코드에서 `fclose` 호출을 깜빡하면 그대로 리소스 누수가 발생한다.
예외가 발생하거나 중간에 return을 하면 파일을 닫지 못하는 상황이 생기기도 한다.

### RAII 방식

```cpp
class FileWrapper {
    FILE* file;
public:
    FileWrapper(const char* name) {
        file = fopen(name, "r");
    }
    ~FileWrapper() {
        if (file) fclose(file);
    }
};
```

* 객체가 생성될 때 파일을 연다.
* 객체가 파괴될 때 자동으로 닫는다.

예외가 발생하더라도 C++의 **스택 언와인딩(stack unwinding)** 덕분에 소멸자가 호출되어 자원이 정리된다.

---

## 2. RAII의 장점

1. **리소스 누수 방지** – 소멸자가 자동 호출되므로 실수로 해제를 빠뜨릴 일이 없다.
2. **코드 단순화** – “닫기” 같은 정리 코드를 따로 신경 쓰지 않아도 된다.
3. **예외 안전성** – 예외가 터져도 자동으로 정리된다.
4. **객체 지향 철학과 부합** – 객체가 자원을 직접 소유한다.

---

## 3. 스마트 포인터란?

스마트 포인터는 쉽게 말해 **포인터에 RAII를 적용한 클래스**이다.
일반 포인터는 소멸자가 없기 때문에 delete를 직접 호출해야 하지만, 스마트 포인터는 **객체의 수명과 함께 자원을 관리**한다.

C++11 이후 표준에는 세 가지 주요 스마트 포인터가 있다.

| 스마트 포인터      | 특징                     |
| ------------ | ---------------------- |
| `unique_ptr` | 단독 소유. 복사 불가, 이동만 가능   |
| `shared_ptr` | 참조 카운팅 기반 공유           |
| `weak_ptr`   | `shared_ptr`의 순환 참조 방지 |

---

## 4. unique\_ptr 사용 예시

`unique_ptr`은 말 그대로 소유권이 유일하다.

```cpp
#include <memory>
#include <iostream>

int main() {
    std::unique_ptr<int> p1 = std::make_unique<int>(10);
    std::cout << *p1 << "\n"; // 10 출력

    // 소유권 이동
    std::unique_ptr<int> p2 = std::move(p1);

    if (!p1) std::cout << "p1은 비어 있음\n";
}
```

출력:

```
10
p1은 비어 있음
```

* `p1`이 가지고 있던 자원을 `p2`로 이동한다.
* 이동 이후 `p1`은 null 상태가 된다.

---

## 5. shared\_ptr과 참조 카운팅

`shared_ptr`은 여러 개의 포인터가 같은 객체를 소유할 수 있도록 한다.
내부적으로 \*\*참조 카운트(reference count)\*\*를 유지하면서, 카운트가 0이 되는 순간 자동으로 자원을 해제한다.

```cpp
#include <memory>
#include <iostream>

struct MyClass {
    MyClass() { std::cout << "생성\n"; }
    ~MyClass() { std::cout << "소멸\n"; }
};

int main() {
    std::shared_ptr<MyClass> sp1 = std::make_shared<MyClass>();
    {
        std::shared_ptr<MyClass> sp2 = sp1;
        std::cout << "use_count: " << sp1.use_count() << "\n";
    }
    std::cout << "use_count: " << sp1.use_count() << "\n";
}
```

출력:

```
생성
use_count: 2
use_count: 1
소멸
```

---

## 6. weak\_ptr로 순환 참조 해결

`shared_ptr`끼리 서로를 가리키면 참조 카운트가 0이 되지 않아 메모리가 해제되지 않는다.
이 문제를 막으려면 `weak_ptr`을 사용해야 한다.

```cpp
#include <memory>

struct B;
struct A {
    std::shared_ptr<B> bptr;
};
struct B {
    std::weak_ptr<A> aptr; // weak_ptr로 순환 참조 해결
};
```

`weak_ptr`은 참조 카운트에 영향을 주지 않으므로 순환 참조를 깨뜨린다.

---

## 7. 게임 개발에서의 활용

게임 엔진 코드에서는 수많은 객체가 서로 연결되어 있다.
언리얼 엔진의 `TSharedPtr`, `TWeakPtr` 역시 같은 철학에서 나온 것이다.
특히 네트워크 환경이나 멀티스레드 환경에서 객체 수명을 안전하게 다루는 데 유용하다.

게임 플레이 도중 특정 Actor가 파괴되었는데, 다른 곳에서 여전히 접근하고 있다면 크래시가 나기 쉽다.
이럴 때 **스마트 포인터와 RAII**를 잘 활용하면 안정성을 크게 높일 수 있다.

---

## 8. 정리

* RAII는 “객체의 수명과 자원의 수명을 일치시킨다”라는 철학이다.
* 스마트 포인터는 포인터에 RAII 개념을 적용한 대표적인 도구이다.
* `unique_ptr`은 단독 소유, `shared_ptr`은 공유 소유, `weak_ptr`은 순환 참조 방지 역할을 한다.
* 현대 C++에서 raw pointer보다 스마트 포인터가 기본 선택지가 되어야 한다.

> 결론: RAII는 단순한 기법이 아니라 C++의 정신 같은 것이다.
> 예외가 터져도, 흐름이 꼬여도, 결국 객체 소멸자에서 자원이 정리된다는 확신을 주기 때문이다.
> 그래서 C++에서 메모리와 자원 관리를 이야기할 때 항상 RAII와 스마트 포인터를 가장 먼저 떠올려야 한다.
