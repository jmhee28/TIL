# 스프링 빈(Bean)

빈(Bean)은 스프링 컨테이너에 의해 관리되는 재사용 가능한 소프트웨어 컴포넌트이다.
즉, 스프링 컨테이너가 관리하는 자바 객체를 뜻하며, 하나 이상의 빈(Bean)을 관리한다.

빈은 인스턴스화된 객체를 의미하며, 스프링 컨테이너에 등록된 객체를 스프링 빈이라고 한다.
@Bean 어노테이션을 통해 메서드로부터 반환된 객체를 스프링 컨테이너에 등록한다.
빈은 클래스의 등록 정보, Getter/Setter 메서드를 포함하며, 컨테이너에 사용되는 설정 메타데이터로 생성된다.

### 빈(Bean) 접근 방법

먼저, `ApplicationContext(스프링 컨테이너)`를 사용하여 bean을 정의를 읽고 액세스 할 수 있다.

```java
// create and configure beans
ApplicationContext context = new ClassPathXmlApplicationContext("services.xml", "daos.xml");

// retrieve configured instance
PetStoreService service = context.getBean("memberRepository", memberRepository.class);

// use configured instance
List<String> userList = service.getUsernameList();
```

## BeanDefinition

스프링의 다양한 설정 형식(Java, XML 등)은 BeanDefinition이라는 추상화 덕분에 할 수 있는 것이다.
Bean은 BeanDefinition(빈 설정 메타정보)으로 정의되고, BeanDefinition에 따라 활용하는 방법이 달라진다.

BeanDefinition은 속성에 따라 컨테이너가 Bean을 어떻게 생성하고 관리할지를 결정한다.
@Bean 어노테이션 또는 <bean> 태그당 1개씩 메타 정보가 생성된다.

Spring이 설정 메타정보를 BeanDefinition 인터페이스를 통해 관리하기 때문에 컨테이너 설정을 XML, Java로 할 수 있는 것이다.
스프링 컨테이너는 설정 형식이 XML인지 Java 코드인지 모르며, BeanDefinition만 알면 된다.

### BeanDefinition 정보

- BeanClassName : 생성할 빈의 클래스 이름
- factoryBeanName : 팩토리 역할의 빈을 사용할 경우의 이름 (appConfig, … 등)
- factoryMethodName : 빈을 생성할 팩토리 메서드 지정 (memberService, … 등)
- Scope : 싱글톤(기본값)
- lazyInit: 스프링 컨테이너를 생성할 때 빈을 생성하는 것이 아니라, 실제 빈을 사용할 때까지 최대한 생성을 지연 처리하는지 여부
- InitMethodName: 빈을 생성하고, 의존관계를 적용한 뒤에 호출되는 초기화 메서드 명
- DestroyMethodName: 빈의 생명주기가 끝나서 제거하기 직전에 호출되는 메서드 명
- Constructor arguments, Properties: 의존관계 주입에서 사용한다. (자바 설정처럼 팩토리 역할의 빈을 사용하면 없음

### 빈 설정 메타정보 확인하기

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
String[] beanDefinitionNames = ac.getBeanDefinitionNames();

for (String beanDefinitionName : beanDefinitionNames ) {
    BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);

    if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
        System.out.println("beanDefinitionName = " + beanDefinitionName + ", beanDefinition = " + beanDefinition);
    }
}
```

- 빈 이름 조회

  - ac.getBeanDefinitionNames(); : 스프링에 등록된 모든 빈 이름을 조회

- 빈 객체 조회
  - ac.getBean(빈이름, 타입) : 빈 인스턴스 조회
  - ac.getBean(타입) : 빈 인스턴스 조회(같은 타입의 스프링 빈이 둘 이상이면 예외 발생)
  - ac.getBeansOfType(타입) : 해당 타입의 모든 빈 조회
- getRole() : 스프링 내부에서 사용하는 빈과 사용자가 등록한 빈을 구분할 수 있다.
- BeanDefinition.ROLE_APPLICATION : 일반적으로 사용자가 정의한 빈
- BeanDefinition.ROLE_INFRASTRUCTURE : 스프링이 내부에서 사용하는 빈

# 30 스프링 빈 상속관계
- 부모 타입으로 조회하면 자식 타입도 함께 조회된다.
- 그래서 모든 자바 객체인 최고 부모인 object 타입으로 조회하면 스프링 빈으로 등록된 모든 자바 객체가 조회된다.

# 31 BeanFactory와 ApplicationContext
- BeanFactory: 스프링 컨테이너의 최상위 인터페이스
  - 스프링 빈을 관리하고 조회하는 역할
- ApplicationContext: BeanFactory를 상속받은 하위 인터페이스
  - ApplicationContext는 BeanFactory의 기능을 모두 포함하면서 부가적인 기능을 제공한다.

## ApplicationContext의 부가 기능
- 국제화 지원: 메시지 소스를 활용한 국제화 기능 제공, 다국어 지원
- 환경변수 : 애플리케이션 실행 환경에 따른 프로파일 및 속성 관리 기능 제공
- 이벤트 발행 및 구독: 애플리케이션 이벤트를 발행하고 구독하는 기능 제공
- 편리한 리소스 조회: 파일, 클래스패스 등 다양한 리소스를 편리하게 조회하는 기능 제공

## Reference

[스프링 빈이란 무엇인가?](https://ittrue.tistory.com/221dd)

```

```
