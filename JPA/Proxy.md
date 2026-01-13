
# 프록시

## 프록시란?

프록시는 실제 객체를 대신하는 가짜 객체로, 데이터베이스 조회를 지연시키는 역할을 합니다

- **em.find()**: 데이터베이스에서 실제 엔티티 객체를 조회
- **em.getReference()**: 데이터베이스 조회를 지연시키는 프록시 객체 반환

```java
Member member = em.getReference(Member.class, "id1"); // 프록시 객체 반환
System.out.println(member.getName()); // 이 시점에서 DB 조회 발생
```

![](https://i.imgur.com/fAMF1Xq.png)

### 프록시는 왜 필요할까?

![](https://i.imgur.com/YUj61BM.png)

위의 그림과 같은 관계가 있을 때, 멤버를 조회할 때 team도 조회할 필요가 있을까?
만약의 team의 정보가 필요없다면 같이 조회하지 않는게 성능 상 더 좋을 것이다.

## 프록시의 특징

![](https://i.imgur.com/XCIPOdL.png)

- 실제 클래스를 상속 받아서 만들어짐
- 실제 클래스와 겉 모양이 같다.
- 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 됨(이론상)
- 프록시 객체는 실제 객체의 참조(target)를 보관
- 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출
    
    ![](https://i.imgur.com/3xHgRhu.png)
    

### 프록시 객체의 초기화

- 우선 **em.getReference()**로 프록시 객체를 가져와 Member member에 저장한다.
- *member.getName()**을 호출하면, 프록시 객체의 Member target에 값이 존재하지 않기 때문에 영속성 컨텍스트에 초기화를 요청한다.
- 이후 영속성 컨텍스트가 DB에 SELECT SQL을 실행하고,
- 실제 엔티티 객체를 생성한 뒤
- 프록시 객체의 target에 **참조 값**을 담아 프록시 객체와 실제 객체를 연결한다.

![](https://i.imgur.com/TOojQJF.png)

- 프록시 객체는 처음 사용할 때 한 번만 초기화
- 프록시 객체를 초기화 할 때, 프록시 객체가 실제 엔티티로 바뀌는 것은 아님, 초기화되면 프록시 객체를 통해서 실제 엔티티에 접근 가능
    - 즉, 초기화는 프록시 객체를 통해 실제 엔티티에 접근할 수 있도록 참조 값을 추가하는 과정
- 프록시 객체는 원본 엔티티를 상속받음, 따라서 타입 체크시 주의해야함 (== 비교 실패, 대신 instance of 사용)
    
    ```java
    // 출력 False
    System.out.println(findMember.getClass() == Member.class);
    
    // 출력 True
    System.out.println(findMember instanceof Member);
    
    ```
    
- 영속성 컨텍스트에 찾는 엔티티가 이미 있으면 em.getReference0를 호출해 도 실제 엔티티 반환
    - 원본 엔티티가 이미 **1차 캐시에 올라와있는 상태(동일한 영속성 컨텍스트)**에서 굳이 프록시 객체를 가져올 필요가 없다.
    - JPA는 **동일한 영속성 컨텍스트 안**에서 PK가 같은 두 객체를 == 비교를 할 경우 항상 TRUE를 출력해야 한다. 따라서 **em.getReference()**를 2번 호출해 프록시 객체를 두 번 꺼내더라도 같은 프록시 객체가 꺼내진다.

```java
Member m1 = em.find(Member.class, member1.getId()); // 실제 엔티티 조회 & 1차 캐시 저장
System.out.println("m1: " + m1.getClass());

Member ref = em.getReference(Member.class, member1.getId()); // 실제 엔티티
System.out.println("ref: " + ref.getClass());

// 출력은 무조건 TRUE
System.out.println("m1 == ref: " (m1 == ref));

```

```java
Member refMember = em.getReference(Member.class, member1.getId()); // 프록시 객체 조회
System.out.println("refMember: " + refMember.getClass());

Member findMember = em.find(Member.class, member1.getId()); // SELECT SQL 실행
System.out.println("findMember: " + findMember.getClass()); // 프록시 객체

// 출력은 무조건 TRUE
// 프록시 객체가 한 번 반환되면, em.find()를 호출해도 프록시 객체가 반환된다.
System.out.println("refMember == findMember: " (refMember == findMember));

```

- 영속성 컨텍스트의 도움을 받을 수 없는 준영속 상태일 때, 프록시를 초기화하면 문제 발생
    - 하이버네이트에서 **org.hibernate.LazyInitializationException**을 터트린다.
    - **프록시 객체 초기화**는 영속성 컨텍스트의 도움을 받아 진행되기 때문이다.
    - 실무에서 자주 만날 수 있다. 꼭 기억해 두자.

```java
Member refMember = em.getReference(Member.class, member1.getId()); // 프록시 객체 조회
System.out.println("refMember: " + refMember.getClass());

// 영속 상태에서 준영속 상태로 변경
em.close(); // em.detach(); || em.clear();
// 프록시 객체 초기화 -> 문제 발생 (영속성 컨텍스트의 도움을 받을 수 없기 때문)
System.out.println("refMember name: " + refMember.getUsername());

```

### 프록시 확인

- 프록시 인스턴스의 초기화 여부 확인
PersistenceUnitUtil. isLoaded(Object entity)
- 프록시 클래스 확인 방법
entity.getClass.getName #9.javasist.. or
HibernateProxy…)
- 프록시 강제 초기화
org.hibernate.Hibernate.initialize(entity);
- 참고: JPA 표준은 강제 초기화 없음
강제 호출: member.getName

```java
Member refMember = em.getReference(Member.class, member1.getId());

// 초기화 여부 확인 (출력 FALSE)
System.out.println("isLoaded=" + emf.getPersistenceUtilUnit().isLoaded(refMember));

// 강제 초기화 -> refMember.getUsername()으로도 가능
Hibernate.initialize(refMember);

// 초기화 여부 확인 (출력 TRUE)
System.out.println("isLoaded=" + emf.getPersistenceUtilUnit().isLoaded(refMember));

// class명 확인
System.out.println("refMember: " + refMember.getClass());

```

# 즉시 로딩과 지연 로딩

## 지연 로딩 (Lazy Loading)

위에서 말했듯이, 단순히 Member 정보만 사용하는 비즈니스 로직이 있다면 Member를 조회할 때 굳이 Team까지 조회할 필요가 없다. 이런 경우를 위해 JPA에서 **지연 로딩(Lazy Loading)**을 지원한다.

- 아래처럼 페치 타입을 **LAZY**로 지정하면, Member을 조회할 때 Team은 프록시 객체로 조회한다.
    
    ![](https://i.imgur.com/aEANC2K.png)
    

```java
@Entity

public class Member {

@Id

@GeneratedValue

private Long id;

@Column(name = "USERNAME")

private String name;

@ManyToOne(**fetch** **= FetchType.****_LAZY_**) //**

@JoinColumn(name = "TEAM_ID")

private Team team;

```

```java
Member member = em.find(Member.class, 1L); // Team은 프록시 객체로 조회

Team team = member.getTeam();
team.getName(); // 실제 Team을 사용하는 시점에 초기화(DB 조회)

```

**지연 로딩 LAZY**를 사용해서 연관된 엔티티를 프록시로 조회할 수 있다. 따라서 연관된 엔티티를 실제 사용하는 시점에 실제 엔티티를 조회한다.

## 즉시로딩 (Eager Loading)

반면에 Member와 Team을 자주 함께 사용한다면 **즉시 로딩(Eager Loading)**을 사용하는 게 더 효율적이다.

![](https://i.imgur.com/yJmqFU6.png)

```java
@Entity
public class Member {

    ...

    @ManyToOne(fetch = FetchType.EAGER)
    @JoinColumn(name = "TEAM_ID")
    private Team team;
}

```

- **즉시 로딩 EAGER를** 사용하면 JPA 구현체는 가능하면 조인을 사용해서 한 번의 SQL로 연관된 엔티티까지 모두 조회한다.
- 또는 엔티티마다 em.find()를 호출해 조회하기도 한다

```java
Member member = em.find(Member.class, 1L); // Member와 연관된 Team까지 한 번에 조회

```

### 주의사항

- **가급적** **지연** **로딩만** **사용***(**특히 실무에서**)**
- 즉시 로딩을 적용하면 예상하지 못한 SQL이 발생
- **즉시** **로딩은** [**JPQL****에서** **N+1** **문제](https://velog.io/@jinyoungchoi95/JPA-%EB%AA%A8%EB%93%A0-N1-%EB%B0%9C%EC%83%9D-%EC%BC%80%EC%9D%B4%EC%8A%A4%EA%B3%BC-%ED%95%B4%EA%B2%B0%EC%B1%85)를** **일으킨다***.**
- **@ManyToOne, @OneToOne****은** **기본이** **Eager** **로딩**
- **> LAZY****로** **설정**
- @OneToMany, @ManyToMany는 기본이 Lazy 로딩

# Cascade

- 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속상태로 만들도 싶을 때
- Ex) 예: 부모 엔티티를 저장할 때 자식 엔티티도 함께 저장.
- 부모와 자식 모두의 **생명주기**가 연동
    
    ![](https://i.imgur.com/AW9QB85.png)
    

```java
@OneToMany(mappedBy="parent",Cascade=**CascadeType.PERSIST**)
```

![](https://i.imgur.com/K2347E1.png)

- **ALL: 모두적용**
- **PERSIST: 영속**
- **REMOVE: 삭제**
- MERGE: 병합
- REFRESH: REFRESH
- DETACH: DETACH

# Orphan 고아 객체

- 고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티 를 자동으로 삭제
- orphanRemoval = true
- `Parent parent1 = em.find(Parent.class, id); parent1.getChildren0.remove(0);//자식 엔티티를 컬렉션에서 제거`
- `DELETE FROM CHILD WHERE ID=?`
- 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
- 참조하는 곳이 하나일 때 사용해야함!
- 특정 엔티티가 개인 소유할 때 사용
- @OneToOne, @OneToMany만 가능

## 영속성 전이 + 고아 객체, 생명주기

- **CascadeType.ALL + orphanRemoval=true**
- 스스로 생명주기를 관리하는 엔티티는 em.persist()로 영속화, em.remove()로 제거
- 두 옵션을 모두 활성화 하면 부모 엔티티를 통해서 자식의 생명 주기를 관리할 수 있음
- 도메인 주도 설계(DDD)의 Aggregate Root개념을 구현할 때 유용

### **Cascade와 orphanRemoval의 동작 시점**

### **Cascade**:

- **실행 시점**: 부모 엔티티에 대한 작업이 발생할 때, 즉 `persist`, `merge`, `remove`, `refresh` 등의 작업이 호출되는 **즉시** 전파됩니다.
- 트랜잭션 커밋과는 독립적으로 **엔티티 매니저**에 의해 관리됩니다.

```java
@Entity
class Parent {
    @OneToMany(cascade = CascadeType.PERSIST)
    private List<Child> children = new ArrayList<>();
}

// 코드
Parent parent = new Parent();
Child child = new Child();
parent.getChildren().add(child);
entityManager.persist(parent);

```

**동작 순서**

1. `entityManager.persist(parent)` 호출
2. **Cascade** 설정으로 인해 `child`도 `persist` 호출
3. 두 엔티티 모두 **트랜잭션이 커밋되기 전에 영속성 컨텍스트에 저장**

### **orphanRemoval**:

- **실행 시점**: 부모와 자식 간의 관계가 **끊어진 시점**에 발생하며, 영속성 컨텍스트에서 해당 자식 엔티티를 삭제로 마크합니다.
- 부모-자식 관계가 변경되었는지 감지된 후, 고아 상태의 엔티티가 삭제로 처리됩니다.
- 변경 사항은 트랜잭션 커밋 시 DB에 반영됩니다.

```java
@Entity
class Parent {
    @OneToMany(orphanRemoval = true)
    private List<Child> children = new ArrayList<>();
}

// 코드
Parent parent = entityManager.find(Parent.class, 1L); // 1. Parent를 영속 상태로 가져옴
Child childToRemove = parent.getChildren().get(0);

parent.getChildren().remove(childToRemove); // 2. 부모와 자식 간 관계 제거

// 3. Dirty Checking 발생
transaction.commit(); // 4. orphanRemoval에 의해 고아 상태 자식 삭제

```

**동작 순서**

- **`entityManager.find`**: `parent`와 연관된 `children`이 영속 상태가 됨.
- **관계 제거**: `parent.getChildren().remove(childToRemove)`를 호출해 자식을 부모 관계에서 제거.
- **Dirty Checking**: JPA는 부모-자식 관계가 변경된 것을 감지.
- **orphanRemoval**:
    - JPA는 `childToRemove`가 고아 상태임을 확인.
    - 해당 엔티티를 영속성 컨텍스트에서 **삭제로 마크**.
- **트랜잭션 커밋**: 삭제된 상태가 DB에 반영.