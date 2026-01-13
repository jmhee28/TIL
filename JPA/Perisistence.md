# 영속성 컨텍스트 (Persistence Context)

- JPA의 핵심 개념으로, "엔티티를 영구 저장하는 환경"을 의미합니다.
- `EntityManager.persist(entity)`로 영속성 컨텍스트에 저장합니다.
- 물리적인 저장소가 아니라 논리적 개념입니다.
- 엔티티 매니저를 통해 접근합니다.

![Persistence Context](https://i.imgur.com/gLvkd8X.png)

---

## 엔티티의 생명주기

### 1) 비영속 (Transient)

- 영속성 컨텍스트와 무관한 상태입니다.
- 단순 객체로 메모리에만 존재합니다.

```java
// 엔티티 객체를 생성한 상태 (비영속)
Member member = new Member();
member.setId(1L);
member.setUsername("회원1");

// 이 상태에서는 JPA와 전혀 관계가 없습니다.
```

![Transient](https://i.imgur.com/uNy03Mv.png)

### 2) 영속 (Managed)

- 영속성 컨텍스트에 저장된 상태입니다.
- 변경 감지와 DB 동기화가 자동으로 이루어집니다.

```java
EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

tx.begin();

// 엔티티 객체 생성
Member member = new Member();
member.setId(1L);
member.setUsername("회원1");

// 영속성 컨텍스트에 저장
em.persist(member); // 영속 상태로 전환
System.out.println("Persist 완료");

// 영속성 컨텍스트에서 조회
Member findMember = em.find(Member.class, 1L);
System.out.println(findMember.getUsername()); // "회원1"

tx.commit();
em.close();
```

![Managed](https://i.imgur.com/Fet5jOy.png)

### 3) 준영속 (Detached)

- 영속성 컨텍스트에서 분리된 상태입니다.
- 변경 감지 및 DB 동기화가 불가능합니다.

```java
EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

tx.begin();

// 영속 상태로 조회
Member member = em.find(Member.class, 1L);

// 준영속 상태로 전환
em.detach(member); // 특정 객체만 준영속 상태로 전환
System.out.println("detach 완료");

// 준영속 상태에서는 변경 사항이 반영되지 않음
member.setUsername("변경된 이름");

tx.commit();
em.close();
```

### 4) 삭제 (Removed)

- 엔티티가 삭제 대상으로 표시된 상태입니다.
- 트랜잭션 커밋 시 실제 DB에서 삭제됩니다.

```java
EntityManager em = emf.createEntityManager();
EntityTransaction tx = em.getTransaction();

tx.begin();

// 엔티티 조회 후 삭제
Member member = em.find(Member.class, 1L);
em.remove(member); // 삭제 상태로 전환
System.out.println("remove 완료");

tx.commit();
em.close();
```

![Removed](https://i.imgur.com/QJmEKCo.png)

---

## 영속성 컨텍스트의 주요 이점

### 1) 1차 캐시

- 영속성 컨텍스트 내부 캐시에 엔티티를 저장합니다.
- 동일한 데이터 재조회 시 DB 호출을 최소화합니다.

![First Level Cache](https://i.imgur.com/BoNdEhd.png)

```java
Member member1 = em.find(Member.class, "id1"); // DB에서 조회
Member member2 = em.find(Member.class, "id1"); // 1차 캐시에서 조회
System.out.println(member1 == member2); // true
```

### 2) 동일성(identity) 보장

- 같은 트랜잭션에서 같은 키로 조회한 엔티티는 항상 같은 객체입니다.

![Identity](https://i.imgur.com/rTrEfq5.png)

### 3) 트랜잭션을 지원하는 쓰기 지연

- 변경 SQL을 즉시 실행하지 않고 모아 두었다가 커밋 시 한 번에 실행합니다.

![Write-Behind](https://i.imgur.com/7anzPub.jpeg)
![Write-Behind Flow](https://i.imgur.com/MtFLRCU.jpeg)

```java
em.persist(member1); // SQL을 데이터베이스에 보내지 않음
em.persist(member2); // SQL을 데이터베이스에 보내지 않음
em.flush();          // SQL 전송
transaction.commit(); // 데이터베이스에 확정
```

### 4) 변경 감지 (Dirty Checking)

- 영속 상태의 변경을 자동 감지해 SQL을 생성합니다.
- 명시적으로 `update()`를 호출하지 않아도 반영됩니다.

![Dirty Checking](https://i.imgur.com/tPozflU.jpeg)

```java
Member member = em.find(Member.class, "id1");
member.setUsername("변경된 이름"); // 변경 감지
transaction.commit(); // 변경 사항 반영
```

---

## 플러시(Flush)와 커밋(Commit)

### 플러시 (Flush)

- 영속성 컨텍스트의 변경 사항을 DB에 반영합니다.
- 이때의 반영은 DB 확정이 아니라 영속성 컨텍스트와 DB의 동기화입니다.
- 최종 확정은 `commit`에서 이루어집니다.
- 자동 호출: JPQL 실행 전, 트랜잭션 커밋 직전 등.
- 명시 호출: `em.flush()`.

### 커밋 (Commit)

- 트랜잭션의 변경 사항을 영구적으로 확정합니다.
- `flush()`를 포함하며 트랜잭션이 종료됩니다.
- 이후에는 롤백할 수 없습니다.

| 구분 | `flush` | `commit` |
| --- | --- | --- |
| DB 반영 | 변경 사항만 전송, 트랜잭션은 열려 있음 | 변경 사항을 전송하고 트랜잭션 종료 |
| 트랜잭션 상태 | 활성 | 종료 |
| 롤백 가능 여부 | 가능 | 불가능 |
| 자동 호출 시점 | JPQL, Native Query 실행 전 등 | `@Transactional` 메서드 종료 시 등 |

---

## 준영속 상태

### 정의

- 엔티티가 영속성 컨텍스트에서 분리된 상태입니다.
- DB 동기화 및 변경 감지가 불가능합니다.

### 전환 방법

1. `em.detach(entity)`
   특정 엔티티만 준영속으로 전환합니다.
2. `em.clear()`
   영속성 컨텍스트를 초기화해 모든 엔티티를 준영속으로 전환합니다.
3. `em.close()`
   영속성 컨텍스트를 종료합니다.

---

## 결론

영속성 컨텍스트는 JPA의 핵심으로, DB와의 효율적인 상호작용을 가능하게 합니다. 특히 **1차 캐시**, **쓰기 지연**, **변경 감지**는 성능 최적화와 객체 지향적인 개발에 큰 도움이 됩니다. 이미지를 함께 보면 개념 이해가 훨씬 쉬워집니다.
