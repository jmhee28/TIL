# [스프링 컨테이너(Spring Container)](https://ittrue.tistory.com/220)

스프링 컨테이너: 스프링 프레임워크의 핵심 컴포넌트

- 자바 객체의 생명 주기를 관리하며
- 생성된 자바 객체들에게 추가적인 기능을 제공한다.
- 설정정보를 참고하여 의존관계를 주입(DI, Dependency Injection)한다.

스프링에서는 자바 객체를 빈(Bean)이라 한다. 즉, 스프링 컨테이너는 내부에 존재하는 빈의 생명주기를 관리(빈의 생성, 관리, 제거 등)하며,
생성된 빈에게 추가적인 기능을 제공하는 것이다.

스프링 컨테이너는 XML, 어노테이션 기반의 자바 설정 클래스로 만들 수 있다.
스프링 부트(Spring Boot)를 사용하기 이전에는 xml을 통해 직접적으로 설정해 주어야 했지만, 스프링 부트가 등장하면서 대부분 사용하지 않게 되었다.

스프링 컨테이너는 Beanfactory와 ApplicationContext 두 종류의 인터페이스로 구현되어 있다.
빈 팩토리는 빈의 생성과 관계설정 같은 제어를 담당하는 IoC 오브젝트이고, 빈 팩토리를 좀 더 확장한 것이 애플리케이션 컨텍스트이다.
## 스프링 컨테이너 생성과정
`new AnnotationConfigApplicationContext(AppConfig.class)`
- 스프링 컨테이너를 생성할 때는 구성정보를 지정해주어야 한다.
- 여기서는 `AppConfig.class`가 구성 정보

## 스프링 컨테이너를 사용하는 이유

먼저, 객체를 생성하기 위해서는 new 생성자를 사용해야 한다. 그로 인해 애플리케이션에서는 수많은 객체가 존재하고 서로를 참조하게 된다.
객체 간의 참조가 많으면 많을수록 의존성이 높아지게 된다.
이는 낮은 결합도와 높은 캡슐화를 지향하는 객체지향 프로그래밍의 핵심과는 먼 방식이다.
따라서, 객체 간의 의존성을 낮추어(느슨한 결합) 결합도는 낮추고, 높은 캡슐화를 위해 스프링 컨테이너가 사용된다.

또한, 기존의 방식으로는 새로운 기능이 생기게 되면 변경 사항들을 수작업으로 수정해야 한다.
프로젝트가 커질수록 의존도는 높아질 것이고, 그에 따라 코드의 변경도 많아질 것이다.
하지만, 스프링 컨테이너를 사용하면 구현 클래스에 있는 의존성을 제거하고 인터페이스에만 의존하도록 설계할 수 있다.

# 싱글톤

싱글톤 패턴은 인스턴스가 1개만 생성되는 것을 보장하는 디자인 패턴이다.
private 생성자를 사용하여 외부로부터 임의로 new 키워드를 사용하지 못하도록 막을 수 있다.
싱글톤 패턴을 적용하면 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 사용할 수 있음.
하지만, 싱글톤 패턴은 다음과 같은 수많은 문제점들을 가지고 있다.

## 싱글톤 단점
- 싱글톤 패턴을 구현하는 코드 자체가 많이 들어간다.
- 의존관계상 클라이언트가 구체 클래스에 의존하게 된다 → DIP 위반
- 클라이언트가 구체 클래스에 의존하면서 OCP 원칙을 위반할 가능성이 높다.
- 테스트하기 어렵다.
- 내부 속성을 변경하거나 초기화하기 어렵다.
- private 생성자로 자식 클래스를 만들기 어렵다.
- 유연성이 떨어진다.
- 위 문제점들로 인해 안티패턴이라 불리기도 한다.

## 싱글톤 컨테이너

| 스프링 프레임워크는 스프링 컨테이너를 통해 싱글톤의 모든 단점들을 해결하면서도 싱글톤의 역할을 할 수 있도록 해준다.
스프링 빈이 바로 싱글톤으로 관리되는 빈이다.
스프링 컨테이너를 싱글톤 컨테이너라고도 한다.

- 스프링 컨테이너는 싱글턴 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리
  - 이전에 설명한 컨테이너 생성 과정을 자세히 보면, 컨테이너는 객체를 하나만 생성해서 관리
- 스프링 컨테이너는 싱글톤 컨테이너 역할. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라고 함
- 스프링 컨테이너의 이런 기능 덕분에 싱글턴 패턴의 모든 단점을 해결하면서 객체를 싱글톤으로 유지할 수 있음
- 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 됨
- DIP, OCP, 테스트, private 생성자로부터 자유롭게 싱글톤을 사용할 수 있음

## 싱글톤 방식의 주의점
- 여러 클라이언트가 하나의 같은 인스턴스를 공유하기 때문에 상태를 싱글톤 객체는 유지(stateful)하지 않도록 설계해야 함
- stateless로 설계
  - 특정 클라이언트에 의존적인 필드가 있으면 안됨
  - 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안
  - 가급적 읽기 전용 필드만 사용
  - 필요하면 자바에서 공유되지 않는 지역변수, 파라미터, ThreadLocal 사용

### 상태를 유지하는 필드 때문에 발생하는 문제점 확인 예제
* statefulService1, statefulService2가 같은 인스턴스임
* 따라서, A사용자가 10000원 주문한 후에 B사용자가 20000원 주문하면
* statefulService1.getPrice()를 호출하면 20000원이 나옴
* 즉, 특정 클라이언트에 대한 상태가 아닌, 모든 클라이언트에 대한 공유 필드가 되어버림
* 해결책: 무상태(stateless)로 설계 변경
```java
class StatefulServiceTest {

  @Test
  void statefulServiceSingleton() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
    StatefulService statefulService1 = ac.getBean(StatefulService.class);
    StatefulService statefulService2 = ac.getBean(StatefulService.class);

    //ThreadA: A사용자 10000원 주문
    statefulService1.order("userA", 10000);

    //ThreadB: B사용자 20000원 주문
    statefulService2.order("userB", 20000);

    //ThreadA: A사용자 주문 금액 조회
    int price = statefulService1.getPrice();
    System.out.println("price = " + price);

    Assertions.assertEquals(statefulService1.getPrice(), 20000); 
  }
}
```
## References

- https://ittrue.tistory.com/220
- https://melodist.github.io/docs/Spring/SpringCore_5
