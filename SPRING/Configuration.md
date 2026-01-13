# Configuration(38.Configuration과 싱글톤)

## AppConfig
- memberService 빈 생성 코드를 보면 memberRepository() 메서드를 호출하는 것을 볼 수 있다.
  - 이 메서드를 호출 하면 `new MemoryMemberRepository()`를 호출한다.
- orderService 빈 생성 코드도 memberRepository() 메서드를 호출한다.
  - 이 메서드를 호출 하면 `new MemoryMemberRepository()`를 호출한다.
따라서, 얼핏 보면 memberService와 orderService가 각각 다른 MemoryMemberRepository 객체를 참조하는 것처럼 보인다.


```java
public class AppConfig {
    // 역할이 분명히 보임
    // Bean memberService -> new MemoryMemberRepository()
    // Bean orderService -> new MemoryMemberRepository()
    @Bean
    public MemberService memberService() { // 역할을 세우고 구현이 그안에 들어가도록
        // 생성자 주입
        return new MemberServiceImpl(memberRepository());
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
```

### 스프링 컨테이너와 싱글톤
- 스프링 컨테이너는 기본적으로 싱글톤으로 빈을 관리
- `@Configuration` 을 붙이면 바이트코드를 조작하는 CGLIB라는 라이브러리를 사용해서 싱글톤을 보장


XXCGLIB 라는 바이트코드 조작 라이브러리가 들어감
스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 AppConfig 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것
```java
@Test
  void configurationDeep(){
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean.getClass());
    
  }
```

@Bean이 붙은 메서드가 호출될 때마다, 스프링 컨테이너는 이미 생성된 빈이 있는지 먼저 확인하고, 있으면 그 빈을 반환한다.
따라서, memberRepository() 메서드가 여러 번 호출되더라도, 실제로는 한 번만 호출되어 하나의 MemoryMemberRepository 객체만 생성되고, 그 객체가
재사용된다.

### 정리
- 스프링 컨테이너는 기본적으로 싱글톤으로 빈을 관리
- `@Configuration` 을 붙이면 바이트코드를 조작하는 CGLIB라는 라이브러리를 사용해서 싱글톤을 보장
- `@Configuration` 을 붙이지 않으면 싱글톤이 보장되지 않음
- 따라서, `@Configuration` 을 꼭 붙이자!
- 만약 `@Configuration` 을 붙이지 않으면, `@Bean` 메서드가 호출될 때마다 새로운 객체가 생성되어 싱글톤이 깨질 수 있다.