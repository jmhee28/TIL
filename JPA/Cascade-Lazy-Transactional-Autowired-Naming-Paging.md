# Cascade, Lazy Loading, Transactional, Autowired, JPA Naming, Paging

## 1. Cascade Types in JPA

Cascade는 부모-자식 관계에서 부모의 작업(Insert, Delete 등)을 자식에게 전파하는 설정입니다.

- **CascadeType.PERSIST**: 부모 저장 시 자식도 함께 저장
- **CascadeType.REMOVE**: 부모 삭제 시 자식도 함께 삭제
- **orphanRemoval = true**: 부모와의 관계가 끊긴 자식을 자동 삭제

```java
@OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
private List<Child> children;
```

**`CascadeType.REMOVE`와 `orphanRemoval = true` 차이**

- `CascadeType.REMOVE`: 부모가 삭제될 때 자식도 삭제
- `orphanRemoval = true`: 부모와의 연결이 끊긴 자식만 삭제

---

## 2. Lazy Loading in `@OneToMany`

`@OneToMany`의 기본 Fetch 전략은 `LAZY`입니다. 부모를 조회할 때 자식을 즉시 가져오지 않고, 실제 접근 시 로딩합니다.

- **장점**: 불필요한 데이터 로드를 줄여 성능 향상
- **단점**: 접근 시 추가 쿼리 발생

```java
@OneToMany(mappedBy = "parent", fetch = FetchType.LAZY)
private List<Child> children;
```

즉시 로딩으로 바꾸려면 `EAGER`를 사용합니다. 대량 로딩 위험이 있으므로 신중히 사용합니다.

```java
@OneToMany(mappedBy = "parent", fetch = FetchType.EAGER)
private List<Child> children;
```

---

## 3. `@Transactional`과 Rollback

- **`@Transactional`**: 트랜잭션 경계를 설정하며, 성공 시 `commit`, 실패 시 `rollback`됩니다.
- **테스트 메서드**: 테스트에서 `@Transactional`을 사용하면 기본적으로 종료 시 `rollback`됩니다.

```java
@Test
@Transactional
public void testSave() {
    // 테스트 실행 후 insert된 데이터는 rollback됨
}
```

---

## 4. `@Autowired`와 의존성 주입

`@Autowired`는 스프링 컨테이너의 빈을 자동 주입합니다.

### 주입 방식

1. **필드 주입**

```java
@Autowired
private QuestionRepository questionRepository;
```

- 장점: 간결함
- 단점: 테스트 어려움, 순환 참조 위험

2. **Setter 주입**

```java
@Autowired
public void setQuestionRepository(QuestionRepository questionRepository) {
    this.questionRepository = questionRepository;
}
```

3. **생성자 주입 (권장)**

```java
private final QuestionRepository questionRepository;

public QuestionService(QuestionRepository questionRepository) {
    this.questionRepository = questionRepository;
}
```

---

## 5. JPA Naming Methods

JPA는 메서드 이름으로 쿼리를 자동 생성합니다.

- 기본 패턴: `findBy`, `existsBy`, `countBy`, `deleteBy`

```java
List<User> findByName(String name);
boolean existsByEmail(String email);
void deleteById(Long id);
```

여러 조건도 조합할 수 있습니다.

```java
List<User> findByAgeGreaterThanAndCity(String city, int age);
```

복잡한 조건은 JPQL 또는 네이티브 쿼리를 권장합니다.

```java
@Query("SELECT u FROM User u WHERE u.age > :age AND u.city = :city")
List<User> findUsersByCityAndAge(@Param("city") String city, @Param("age") int age);
```

---

## 6. `Page`와 `Pageable`

`Pageable`은 페이징 요청 정보를, `Page`는 페이지 결과와 메타데이터를 담습니다.

### 6.1 `Pageable`

- 요청 페이지 번호(0부터 시작), 페이지 크기, 정렬 정보를 포함합니다.
- `PageRequest`로 생성합니다.

```java
import org.springframework.data.domain.PageRequest;
import org.springframework.data.domain.Pageable;
import org.springframework.data.domain.Sort;

Pageable pageable = PageRequest.of(0, 10, Sort.by("id").descending());
```

### 6.2 `Page`

- 현재 페이지 데이터와 전체 페이지 수, 전체 데이터 수 등을 제공합니다.

```java
Page<Post> page = postRepository.findAll(pageable);

List<Post> posts = page.getContent();
int totalPages = page.getTotalPages();
long totalElements = page.getTotalElements();
boolean isLast = page.isLast();
```

### 6.3 사용 예시

**Repository**

```java
public interface PostRepository extends JpaRepository<Post, Long> {
    Page<Post> findByTitle(String title, Pageable pageable);
}
```

**Service**

```java
Pageable pageable = PageRequest.of(0, 5, Sort.by("id").descending());
Page<Post> page = postRepository.findByTitle("Example Title", pageable);

List<Post> posts = page.getContent();
int totalPages = page.getTotalPages();
long totalElements = page.getTotalElements();
```

### 6.4 페이징 API 응답 예시

```json
{
  "content": [
    { "id": 1, "title": "Post 1" },
    { "id": 2, "title": "Post 2" }
  ],
  "pageable": {
    "sort": { "sorted": true, "unsorted": false, "empty": false },
    "pageNumber": 0,
    "pageSize": 5,
    "offset": 0,
    "paged": true,
    "unpaged": false
  },
  "totalPages": 10,
  "totalElements": 50,
  "last": false,
  "first": true,
  "size": 5,
  "number": 0,
  "sort": { "sorted": true, "unsorted": false, "empty": false },
  "numberOfElements": 5,
  "empty": false
}
```

---

## 참고 자료

- Hibernate 공식 문서: https://hibernate.org/
- Spring Data JPA: https://spring.io/projects/spring-data-jpa
- `@Transactional` 설명: https://www.baeldung.com/spring-transactional-annotation
