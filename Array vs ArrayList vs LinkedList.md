# Array vs ArrayList vs LinkedList

---

## 1. 한 문장 정의

- **Array**: 고정 크기의 연속된 메모리 공간에 데이터를 저장하는 자료구조
- **ArrayList**: 크기가 자동으로 늘어나는 동적 배열 기반의 List 구현체
- **LinkedList**: 노드(데이터 + 포인터)들이 연결되어 있는 List 구현체

---

## 2. 구조 이미지

- **Array**  
  <img src="https://t1.daumcdn.net/cfile/tistory/995E66395B1CFD7D10" width="400">

- **LinkedList**  
  <img src="https://t1.daumcdn.net/cfile/tistory/99250A345B1CFD690C" width="400">

---

## 3. 특징과 코드 예시

### ✅ Array

- 선언 시 크기와 데이터 타입을 지정해야 함
- 인덱스를 통한 빠른 접근 (O(1))
- 크기 변경 불가, 삽입/삭제 비효율적 (O(n))

```java
int[] arr = new int[5];
arr[0] = 10;
arr[1] = 20;
System.out.println(arr[1]); // 20
```

---

### ✅ ArrayList

- 내부적으로 \*\*동적 배열(Dynamic Array)\*\*로 구현
- 크기가 자동 조절됨
- 인덱스 접근은 O(1), 하지만 삽입/삭제는 O(n)

```java
import java.util.ArrayList;

ArrayList<Integer> list = new ArrayList<>();
list.add(10);
list.add(20);
list.add(1, 15); // 중간 삽입
System.out.println(list); // [10, 15, 20]
```

---

### ✅ LinkedList

- 노드(Node)들이 포인터로 연결된 구조
- 삽입/삭제가 빠름 (O(1), 단 위치를 알 때)
- 인덱스 접근은 순차 탐색이 필요 (O(n))

```java
import java.util.LinkedList;

LinkedList<Integer> linkedList = new LinkedList<>();
linkedList.add(10);
linkedList.add(20);
linkedList.addFirst(5); // 맨 앞 삽입
System.out.println(linkedList); // [5, 10, 20]
```

---

## 4. 성능 비교 (시간 복잡도)

| 연산             | Array | ArrayList      | LinkedList |
| ---------------- | ----- | -------------- | ---------- |
| 인덱스 접근      | O(1)  | O(1)           | O(n)       |
| 탐색 (Search)    | O(n)  | O(n)           | O(n)       |
| 삽입/삭제 (중간) | O(n)  | O(n)           | O(1)\*     |
| 삽입/삭제 (끝)   | 불가  | Amortized O(1) | O(1)       |

\* 단, LinkedList는 삽입/삭제할 노드를 탐색하는 과정에서 O(n) 발생 가능

---

## 5. 장단점 비교 요약

| 자료구조       | 장점                                                        | 단점                                                                 |
| -------------- | ----------------------------------------------------------- | -------------------------------------------------------------------- |
| **Array**      | - 빠른 인덱스 접근 (O(1))<br>- 메모리 연속 → 캐시 효율 좋음 | - 크기 변경 불가<br>- 삽입/삭제 비효율적                             |
| **ArrayList**  | - 크기 자동 조절<br>- 인덱스 접근 빠름                      | - 중간 삽입/삭제 느림<br>- 크기 초과 시 배열 복사 발생               |
| **LinkedList** | - 삽입/삭제 효율적 (포인터만 변경)<br>- 크기 제한 없음      | - 인덱스 접근 느림<br>- 추가 메모리 필요(포인터)<br>- 캐시 효율 낮음 |

---

## 6. 언제 무엇을 쓰면 좋을까?

- **Array**
  → 크기가 고정되어 있고 빠른 인덱스 접근이 필요할 때
  (예: Lookup 테이블, 요일 상수 배열 등)

- **ArrayList**
  → 크기가 유동적이고 검색/순차 접근이 주 목적일 때
  (예: DB 조회 결과, 일반적인 데이터 리스트)

- **LinkedList**
  → 삽입/삭제가 잦고 순차 탐색 위주일 때
  (예: Queue, Undo/Redo, LRU Cache)

---

## 7. 현업에서의 활용 💡

- **Array**:

  - 정적 데이터 저장, 성능 최적화가 필요한 연산(캐시 효율 ↑)

- **ArrayList**:

  - 가장 많이 사용되는 컬렉션 (API 응답, 데이터 가공 등 거의 기본값)

- **LinkedList**:

  - 큐/덱 구현, 브라우저 방문 기록, 캐시 교체 알고리즘 등 특수 목적

---

## 8. 면접 & 스터디 예상 질문 ✨

1. **ArrayList와 LinkedList의 차이를 설명하라.**
   → 내부 구조, 접근/삽입/삭제 성능 차이
2. **ArrayList에서 중간 삽입이 O(n)인 이유는?**
   → 이후 요소들을 한 칸씩 이동해야 하기 때문
3. **LinkedList에서 k번째 요소 탐색이 O(n)인 이유는?**
   → 포인터를 따라 순차 탐색해야 하기 때문
4. **실무에서는 ArrayList와 LinkedList 중 어느 쪽을 더 많이 쓰나?**
   → 대부분 ArrayList (성능, 메모리 효율, 활용성이 좋아서)

---

📌 **정리 :**
자료구조는 절대 만능이 아니며, **상황에 맞는 선택**이 가장 중요하다!
