---
title: "C++ Rule of Three와 Rule of Five"
date: 2025-08-26 18:45:00 +0900
categories: [C++, OOP]
tags: [c++, rule-of-three, rule-of-five, constructor, memory-management]
description: "C++에서 자원 관리를 안전하게 하기 위한 규칙인 Rule of Three와 Rule of Five를 코드 예제와 함께 정리한다"
---

# Rule of Three와 Rule of Five

C++에서 직접 메모리나 리소스를 관리하는 클래스를 만들다 보면 반드시 알아야 하는 규칙이 있다.  
바로 **Rule of Three**와 **Rule of Five**이다.  
이 규칙은 복사, 대입, 이동과 관련된 생성자와 연산자를 정의할 때의 가이드라인이다.

---

## 1. Rule of Three란?

C++98 시절부터 있었던 규칙으로, 만약 다음 세 가지 중 하나를 정의해야 한다면 나머지 둘도 반드시 정의해야 한다는 원칙이다.

1. **소멸자(Destructor)**  
2. **복사 생성자(Copy Constructor)**  
3. **복사 대입 연산자(Copy Assignment Operator)**  

이유는 간단하다.  
클래스가 직접 리소스를 관리한다면(예: 동적 메모리), 복사나 대입 시 단순한 얕은 복사(shallow copy)로는 문제가 생기기 때문이다.

### 예시: Rule of Three 위반
```cpp
class Bad {
    int* data;
public:
    Bad(int n) { data = new int[n]; }
    ~Bad() { delete[] data; }
};
````

여기서 복사 생성자나 대입 연산자를 정의하지 않았다.
따라서 얕은 복사가 일어나고, 같은 포인터를 두 번 해제하면서 프로그램이 크래시 날 수 있다.

---

## 2. Rule of Three 지키기

```cpp
class Good {
    int* data;
    size_t size;
public:
    Good(size_t n) : size(n) { data = new int[n]; }

    // 복사 생성자
    Good(const Good& other) : size(other.size) {
        data = new int[size];
        std::copy(other.data, other.data + size, data);
    }

    // 복사 대입 연산자
    Good& operator=(const Good& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new int[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }

    ~Good() { delete[] data; }
};
```

이렇게 해야 리소스를 안전하게 관리할 수 있다.

---

## 3. Rule of Five란?

C++11 이후 등장한 개념이다.
\*\*이동语(move semantics)\*\*이 추가되면서 규칙이 확장되었다.
Rule of Three에 두 가지가 더 추가된 것이다.

4. 이동 생성자 (Move Constructor)
5. 이동 대입 연산자 (Move Assignment Operator)

즉, 리소스를 직접 관리하는 클래스라면 **다섯 가지를 모두 정의**해야 한다.

---

## 4. Rule of Five 예시

```cpp
class Smart {
    int* data;
    size_t size;
public:
    Smart(size_t n) : size(n) { data = new int[n]; }

    // 복사 생성자
    Smart(const Smart& other) : size(other.size) {
        data = new int[size];
        std::copy(other.data, other.data + size, data);
    }

    // 복사 대입 연산자
    Smart& operator=(const Smart& other) {
        if (this != &other) {
            delete[] data;
            size = other.size;
            data = new int[size];
            std::copy(other.data, other.data + size, data);
        }
        return *this;
    }

    // 이동 생성자
    Smart(Smart&& other) noexcept : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }

    // 이동 대입 연산자
    Smart& operator=(Smart&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }

    ~Smart() { delete[] data; }
};
```

---

## 5. Rule of Zero?

현대 C++에서는 아예 이런 함수들을 직접 작성하지 않고, 표준 라이브러리가 제공하는 컨테이너(`std::vector`, `std::string` 등)를 사용하자는 철학도 있다.
이를 **Rule of Zero**라고 부른다.
즉, 리소스를 직접 관리하지 말고 안전한 도구를 쓰라는 것이다.

---

## 6. 게임 개발에서의 활용

게임 엔진에서는 종종 대규모 버퍼, 텍스처, 네트워크 패킷 같은 자원을 직접 관리해야 한다.
이때 Rule of Five를 지키지 않으면, 성능 저하나 메모리 누수, 크래시가 발생한다.

반대로, 단순히 자료 구조만 필요하다면 `std::vector`나 `TArray` 같은 안전한 컨테이너를 쓰는 것이 훨씬 낫다.

---

## 7. 정리

* Rule of Three: 소멸자, 복사 생성자, 복사 대입 연산자
* Rule of Five: + 이동 생성자, 이동 대입 연산자
* Rule of Zero: 직접 구현하지 말고 안전한 도구 사용

> 결론: 직접 리소스를 관리해야 한다면 최소한 Rule of Five를 지켜야 한다.
> 하지만 가능하다면 Rule of Zero를 따라가는 것이 더 안전하고 효율적이다.
