---
title: "C++ Move Semantics와 rvalue 참조"
date: 2025-08-12 13:15:00 +0900
categories: [C++, Advanced]
tags: [c++, move, rvalue, reference, c++11]
description: "C++11에서 도입된 Move Semantics와 rvalue 참조 개념을 코드 예제와 함께 정리한다"
---

# Move Semantics와 rvalue 참조

C++11 이후 가장 큰 변화 중 하나가 바로 **rvalue 참조(rvalue reference)**와 **Move Semantics**의 도입이다.  
이 개념 덕분에 C++은 불필요한 복사를 줄이고 훨씬 효율적인 성능을 낼 수 있게 되었다.  
처음 보면 `std::move`와 `&&` 기호가 낯설지만, 차근차근 살펴보면 의외로 단순하다.

---

## 1. lvalue와 rvalue

먼저 C++에서 **lvalue**와 **rvalue**를 구분할 필요가 있다.

- **lvalue**: 이름을 가지고 재사용 가능한 값. 예: 변수, 참조 가능 값  
- **rvalue**: 일시적인 값. 예: `3 + 5`, 함수 반환 값, 임시 객체  

```cpp
int x = 10;    // x는 lvalue
int y = x + 5; // (x + 5)는 rvalue
````

즉, rvalue는 **잠깐 쓰고 사라질 값**이다.

---

## 2. rvalue 참조

C++11에서 도입된 `&&` 문법은 **rvalue 참조**를 의미한다.
즉, 일시적인 값(rvalue)을 참조할 수 있게 해준다.

```cpp
void foo(int&& n) {
    cout << "rvalue 참조: " << n << endl;
}

int main() {
    foo(10);      // OK, 10은 rvalue
    // int a = 5;
    // foo(a);    // 에러, a는 lvalue
}
```

---

## 3. 복사와 이동의 차이

### 복사(Copy)

객체의 내용을 통째로 복사한다. 비용이 크다.

```cpp
string a = "Hello";
string b = a; // 복사 발생
```

### 이동(Move)

자원의 소유권만 옮기고, 실제 데이터는 복사하지 않는다. 비용이 적다.

```cpp
string a = "Hello";
string b = std::move(a); // 이동 발생
```

`a`의 내부 버퍼가 `b`로 넘어가고, `a`는 비워진 상태가 된다.

---

## 4. Move 생성자와 이동 대입 연산자

사용자 정의 클래스에서도 이동语의 장점을 살릴 수 있다.
이를 위해서는 **Move 생성자**와 **이동 대입 연산자**를 정의해야 한다.

```cpp
class MyVector {
    int* data;
    size_t size;
public:
    MyVector(size_t n) : size(n) {
        data = new int[n];
    }

    // Move 생성자
    MyVector(MyVector&& other) noexcept
        : data(other.data), size(other.size) {
        other.data = nullptr;
        other.size = 0;
    }

    // 이동 대입 연산자
    MyVector& operator=(MyVector&& other) noexcept {
        if (this != &other) {
            delete[] data;
            data = other.data;
            size = other.size;
            other.data = nullptr;
            other.size = 0;
        }
        return *this;
    }

    ~MyVector() { delete[] data; }
};
```

이렇게 하면 `std::vector`처럼 대규모 데이터를 효율적으로 옮길 수 있다.

---

## 5. std::move의 역할

`std::move`는 사실 "이동"을 하지 않는다.
단지 **lvalue를 rvalue처럼 취급하라**는 캐스팅 도구일 뿐이다.
진짜 이동은 Move 생성자나 이동 대입 연산자가 처리한다.

```cpp
string a = "Hello";
string b = std::move(a); // a를 rvalue처럼 캐스팅
```

---

## 6. 성능 이점

복사와 이동의 차이를 체감하려면 `std::vector` 예제가 가장 직관적이다.

```cpp
vector<string> v;
v.push_back("A long string..."); // 이동 발생 → 성능 이득
```

만약 이동语가 없었다면, 매번 문자열을 복사해야 해서 성능이 크게 떨어졌을 것이다.
특히 게임 엔진처럼 객체가 자주 생성/소멸되는 환경에서는 무시 못 할 차이가 된다.

---

## 7. 게임 개발에서의 활용

언리얼 엔진의 `TArray`, `FString` 같은 자료형은 내부적으로 이동语를 적극 활용한다.
네트워크 패킷 버퍼 이동, 리소스 핸들 전달 등에서도 이동语는 큰 성능 최적화를 가져온다.

> 복사가 반복되는 루프는 성능 병목이 된다. 이동语를 지원하면 단순히 `std::move` 한 줄로 병목을 제거할 수 있다.

---

## 8. 정리

* lvalue: 이름 있는 값, rvalue: 일시적 값
* rvalue 참조(`&&`)는 rvalue를 참조할 수 있게 한다
* Move Semantics는 "복사 대신 소유권 이동" 개념이다
* `std::move`는 단순 캐스팅 도구, 진짜 이동은 Move 생성자/연산자가 담당한다
* 현대 C++에서 성능 최적화의 핵심 도구 중 하나다

> 결론: 복사 대신 이동을 활용하면 "빠르고 안전한 C++"을 쓸 수 있다.
> 특히 대규모 데이터를 다루는 게임 개발에서는 Move Semantics가 사실상 필수다.
