# Transaction Isolation Level

## 1. 개념
- 트랜잭션 간의 간섭을 어느 정도까지 허용할지 결정하는 설정.
- 동시성(성능)과 일관성(정확성)의 균형을 조절함.
- 내부적으로 Lock과 MVCC(Multi-Version Concurrency Control)를 이용해 구현됨.

---

## 2. 트랜잭션 간 발생할 수 있는 문제

| 문제 | 설명 |
|------|------|
| Dirty Read | 커밋되지 않은 데이터를 읽음 |
| Non-repeatable Read | 같은 데이터를 여러 번 읽을 때 값이 달라짐 |
| Phantom Read | 같은 조건으로 조회 시 결과 행 수가 달라짐 |

### (1) Dirty Read 예시
```sql
-- 트랜잭션 1
BEGIN;
UPDATE accounts SET balance = 0 WHERE user_id = 'A';
-- 아직 COMMIT하지 않음

-- 트랜잭션 2
SELECT balance FROM accounts WHERE user_id = 'A';
-- 아직 커밋되지 않은 balance=0을 읽음

-- 트랜잭션 1
ROLLBACK;
-- 트랜잭션 2는 잘못된(rollback된) 데이터를 읽은 상태
```

### (2) Non-repeatable Read 예시
```sql
-- 트랜잭션 1
BEGIN;
SELECT name FROM users WHERE id = 1;  -- 결과: 'Kim'

-- 트랜잭션 2
UPDATE users SET name = 'Lee' WHERE id = 1;
COMMIT;

-- 트랜잭션 1
SELECT name FROM users WHERE id = 1;  -- 결과: 'Lee'
-- 같은 데이터를 두 번 읽었는데 값이 달라짐
```

### (3) Phantom Read 예시
```sql
-- 트랜잭션 1
BEGIN;
SELECT * FROM orders WHERE price > 100;

-- 트랜잭션 2
INSERT INTO orders (price) VALUES (200);
COMMIT;

-- 트랜잭션 1
SELECT * FROM orders WHERE price > 100;
-- 새로 삽입된 행이 추가로 조회됨 (행 수가 달라짐)
```

---

## 3. Isolation Level 종류
| 수준 | Dirty Read | Non-repeatable Read | Phantom Read | 특징 |
|------|-------------|---------------------|---------------|------|
| READ UNCOMMITTED | 발생 | 발생 | 발생 | 가장 낮은 수준, 커밋 전 데이터 읽음 |
| READ COMMITTED | 방지 | 발생 | 발생 | 커밋된 데이터만 읽음 (Oracle 기본) |
| REPEATABLE READ | 방지 | 방지 | 발생 | 같은 데이터를 반복 읽기 일관성 유지 (MySQL 기본) |
| SERIALIZABLE | 방지 | 방지 | 방지 | 모든 트랜잭션을 순차적으로 처리, 가장 안전 |

---

## 4. MVCC (Multi-Version Concurrency Control)

### 개념
- 다중 버전 동시성 제어(MVCC)는 Lock을 최소화하여 **읽기와 쓰기를 동시에 수행할 수 있게 해주는 기술**.
- 데이터가 수정될 때 기존 데이터를 덮어쓰지 않고, **새로운 버전의 데이터를 생성**.
- 읽는 트랜잭션은 **자신이 시작된 시점에 커밋된 버전**만 조회함.

### 💡 핵심 아이디어
> “각 트랜잭션은 자신만의 타임라인 안에서 데이터의 스냅샷을 본다.”

즉, 트랜잭션은 현실에서의 ‘현재’를 보는 것이 아니라  
“**자신이 시작한 순간의 세계를 그대로 유지한 채 작업하는 것**”이다.

---

### 🎬 비유로 이해하기

> 🧍‍♂️ **학생 A**는 시험지를 받고 문제를 푸는 중이다.  
> 🧍‍♀️ **학생 B**는 시험 감독으로, 시험지를 고치고 있다.

- A는 시험지를 받고 난 뒤부터 **그 시점의 문제**를 기준으로 답을 작성한다.  
- 그 사이 B가 시험지를 수정하더라도,  
  A는 이미 받은 **자신의 시험지(스냅샷)** 를 계속 본다.  
- A가 답안을 다 쓰고 제출(Commit)하면,  
  그제서야 새로운 시험지 버전이 배포된다.

이게 바로 MVCC다.  
→ A는 **자신만의 시점에 고정된 버전**을 읽고,  
→ B는 **새로운 버전을 만든다**.  
→ 두 사람은 동시에 일하지만 서로 방해하지 않는다.

---

### ⚙️ 동작 방식 요약
1. **쓰기(UPDATE/DELETE)** → 기존 데이터를 덮어쓰지 않고 새 버전 생성  
2. **읽기(SELECT)** → 트랜잭션 시작 시점의 커밋된 버전만 조회  
3. **COMMIT 시점** → 새 버전이 “현재”로 반영  
4. **ROLLBACK 시점** → 새 버전 폐기, 이전 버전 유지  

👉 덕분에 **READ COMMITTED**나 **REPEATABLE READ** 격리 수준에서도  
읽기-쓰기 충돌 없이 높은 동시성을 유지할 수 있다.

---

## 5. DBMS별 기본값
| DBMS | 기본 격리 수준 | 특징 |
|------|----------------|------|
| MySQL (InnoDB) | REPEATABLE READ | Gap Lock으로 Phantom Read 대부분 방지 |
| Oracle | READ COMMITTED | MVCC 기반, 실질적으로 일관성 유지 |
| PostgreSQL | READ COMMITTED | Serializable Snapshot Isolation 지원 |

---

## 6. 설정 방법

### MySQL / MariaDB
```sql
-- 현재 격리 수준 확인
SELECT @@transaction_isolation;

-- 세션 단위 변경
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 전역(Global) 변경
SET GLOBAL TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- 트랜잭션 단위 지정
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
START TRANSACTION;
-- SQL 실행
COMMIT;
```

### PostgreSQL
```sql
SHOW TRANSACTION ISOLATION LEVEL;
SET SESSION CHARACTERISTICS AS TRANSACTION ISOLATION LEVEL REPEATABLE READ;
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
COMMIT;
```

### Oracle
```sql
SELECT SYS_CONTEXT('USERENV', 'ISOLATION_LEVEL') FROM DUAL;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### SQL Server
```sql
DBCC USEROPTIONS;
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN TRANSACTION;
COMMIT TRANSACTION;
```

---

## 7. 실무 예시
| 상황 | 추천 수준 | 이유 |
|------|-------------|------|
| 일반 웹 서비스 | READ COMMITTED | 안정성과 성능 균형 |
| 보고서, 조회 중심 | READ COMMITTED / SNAPSHOT | Lock 최소화 |
| 금융, 결제 | REPEATABLE READ / SERIALIZABLE | 데이터 무결성 보장 |
| 로그, 통계 | READ UNCOMMITTED | 속도 우선, 정확도 낮음 |

---

## 8. 면접에서 자주 나오는 질문

### Q1. Isolation Level을 왜 사용하는가?
> 트랜잭션 간의 데이터 간섭을 제어하기 위해 사용한다.  
> 데이터의 **일관성과 무결성**을 유지하면서도 **동시성(성능)** 을 확보하기 위함이다.

---

### Q2. Dirty Read, Non-repeatable Read, Phantom Read 차이점은?
| 구분 | 설명 | 예시 |
|------|------|------|
| Dirty Read | 커밋되지 않은 데이터를 읽음 | A가 수정 후 롤백, B는 그 수정된 값을 읽음 |
| Non-repeatable Read | 같은 데이터를 두 번 읽을 때 값이 달라짐 | B가 A 사이에 데이터를 수정함 |
| Phantom Read | 같은 조건 조회 시 행 수가 달라짐 | 다른 트랜잭션이 새 데이터를 INSERT함 |

---

### Q3. Isolation Level을 높이면 성능이 떨어지는 이유는?
> Lock 유지 시간이 길어지고,  
> 동시에 접근할 수 있는 트랜잭션 수가 줄어들기 때문.  
> 즉, **일관성을 얻는 대가로 동시성을 희생**하게 된다.

---

### Q4. MVCC는 왜 필요한가?
> 읽기 작업이 쓰기 작업과 충돌하지 않게 하여  
> **Lock 없이도 일관된 데이터를 읽을 수 있도록 하기 위함.**  
> 덕분에 READ COMMITTED / REPEATABLE READ 수준에서도  
> 동시성을 높이고 성능을 유지할 수 있다.

---

### Q5. MySQL과 Oracle의 기본 Isolation Level은?
| DBMS | 기본 수준 |
|------|-------------|
| MySQL (InnoDB) | REPEATABLE READ |
| Oracle | READ COMMITTED |

---

### Q6. Phantom Read를 완전히 막으려면?
> **SERIALIZABLE** 수준을 사용해야 한다.  
> 트랜잭션을 완전히 직렬화하여 처리하기 때문에  
> 새로운 행이 추가되거나 삭제되는 것도 차단된다.

---

### Q7. 실무에서는 어떤 수준을 주로 사용하는가?
> 대부분의 서비스는 **READ COMMITTED**를 사용한다.  
> Dirty Read를 방지하면서도 성능 저하가 적기 때문이다.  
> 단, 금융·결제 등 정확도가 중요한 시스템은 **REPEATABLE READ 이상**을 사용한다.
