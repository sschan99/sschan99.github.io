---
title: "C++ STL 컨테이너 비교"
date: 2025-08-05 09:30:00 +0900
categories: [C++, STL]
tags: [c++, stl, vector, map, unordered_map, container]
description: "C++ STL에서 자주 쓰이는 컨테이너(vector, map, unordered_map)의 내부 동작과 성능을 비교한다"
---

# STL 컨테이너 비교

C++ 표준 라이브러리(STL)에는 수많은 컨테이너가 있다.  
`vector`, `list`, `map`, `unordered_map` 등… 처음 배우는 사람은 어떤 걸 써야 할지 헷갈리기 쉽다.  
이번 글에서는 그중 자주 쓰이는 주요 컨테이너 몇 가지를 비교해보고, 상황에 따라 어떤 걸 선택해야 할지 정리한다.

---

## 1. vector: 동적 배열

`vector`는 가장 많이 쓰이는 컨테이너다. 내부적으로는 **동적 배열(dynamic array)**로 구현되어 있다.

```cpp
#include <vector>
#include <iostream>
using namespace std;

int main() {
    vector<int> v = {1, 2, 3};
    v.push_back(4);
    cout << v[2] << endl; // 3
}
````

### 특징

* 임의 접근(Random Access) 가능 → O(1)
* 끝에서 삽입/삭제(push\_back, pop\_back)는 O(1) 평균
* 중간 삽입/삭제는 O(n) (비효율적)
* 메모리를 연속적으로 할당 → 캐시 친화적

👉 게임 개발에서 대규모 객체 리스트를 관리할 때 자주 쓰인다.

---

## 2. list: 이중 연결 리스트

`list`는 **이중 연결 리스트(doubly linked list)** 구조를 가진다.

```cpp
#include <list>
#include <iostream>
using namespace std;

int main() {
    list<int> l = {1, 2, 3};
    l.push_front(0);
    l.remove(2);
    for (int x : l) cout << x << " ";
}
```

### 특징

* 임의 접근 불가 → O(n)
* 중간 삽입/삭제는 O(1)
* 메모리 불연속 → 캐시 친화적이지 않음

👉 중간 삽입/삭제가 자주 일어나는 상황에서만 유리하다.

---

## 3. map: 균형 이진 탐색 트리

`map`은 내부적으로 **Red-Black Tree**로 구현된 정렬 맵이다.

```cpp
#include <map>
#include <iostream>
using namespace std;

int main() {
    map<int, string> m;
    m[2] = "banana";
    m[1] = "apple";
    for (auto& [k, v] : m) cout << k << ":" << v << endl;
}
```

출력:

```
1:apple
2:banana
```

### 특징

* 탐색, 삽입, 삭제 모두 O(log n)
* 키가 자동으로 정렬됨
* 메모리 사용량은 다소 높음

👉 키 순서가 중요한 경우 유리하다.

---

## 4. unordered\_map: 해시 테이블

`unordered_map`은 **해시 테이블(hash table)** 기반이다.

```cpp
#include <unordered_map>
#include <iostream>
using namespace std;

int main() {
    unordered_map<string, int> um;
    um["one"] = 1;
    um["two"] = 2;
    cout << um["two"] << endl; // 2
}
```

### 특징

* 평균 탐색, 삽입, 삭제 → O(1)
* 순서 보장 없음
* 해시 충돌 발생 시 성능 저하 가능

👉 빠른 키-값 탐색이 필요할 때 적합하다.

---

## 5. 주요 컨테이너 성능 비교

| 컨테이너            | 내부 구조          | 접근       | 삽입/삭제    | 정렬       | 특징            |
| --------------- | -------------- | -------- | -------- | -------- | ------------- |
| `vector`        | 동적 배열          | O(1)     | O(n) 중간  | X        | 캐시 친화적, 가장 범용 |
| `list`          | 이중 연결 리스트      | O(n)     | O(1) 중간  | X        | 중간 삽입/삭제에 강함  |
| `map`           | Red-Black Tree | O(log n) | O(log n) | O(log n) | 정렬된 키 보장      |
| `unordered_map` | 해시 테이블         | O(1) 평균  | O(1) 평균  | X        | 빠른 탐색, 순서 없음  |

---

## 6. 게임 개발에서의 선택

* **vector** → 대부분의 경우 최선의 선택. 메모리 접근 패턴이 좋아 성능이 높다.
* **list** → 노드 삽입/삭제가 빈번한 경우에만 고려. (실제로는 거의 안 쓴다)
* **map** → 키 정렬이 필요할 때. 예: 순서 있는 리더보드.
* **unordered\_map** → 빠른 해시 탐색이 필요할 때. 예: 네트워크 패킷 ID → 핸들러 매핑.

---

## 7. 정리

* `vector`는 기본 선택지.
* `list`는 특수한 상황에서만.
* `map`은 정렬된 키가 필요할 때.
* `unordered_map`은 순서 없는 빠른 탐색이 필요할 때.

> 결론: 컨테이너 선택은 성능 최적화에서 큰 차이를 만든다.
> 무조건 한 가지를 고집하지 말고, **접근 패턴**과 **사용 목적**을 기준으로 선택하는 습관이 필요하다.
