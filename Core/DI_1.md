# DI-1

- IOC → 프로그램의 제어 흐름이 뒤바끼는 것
    - 스프링 관점에서 바라보자면 스프링 컨테이너가 빈 오브젝트에 대한 생성 및 생명 주기를 관리하는 것이다.
    - 이러한 IOC는 조금 더 포괄적인 개념으로 바라볼 수 있다. DI는 스프링에서 IOC 기능의 대표적인 동작 방식으로 바라 볼 수 있다.
    - IOC 자체를 Principle 관점에서 바라보고 이를 구현한 많은 Pattern이 존재한다.

### DI ( 종속성 주입 , 의존성 주입 )

> 종속성 주입(DI)은 개체가 생성자 인수, 팩터리 메서드에 대한 인수 또는 개체 인스턴스가 생성된 후 **개체 인스턴스에 설정된 속성을 통해서만 종속성(즉, 함께 작동하는 다른 개체)을 정의**하는 프로세스입니다. (디자인 패턴)
>

> Dependency Injection (DI) is a design pattern that **allows you to remove the hard-coded dependencies between objects in your application, and instead, provide them with their dependencies through a central location**.
>
- 팩토리 메서드
    - 구체적으로 사용할 오브젝트를 결정해주는 메서드 (추상적인 개념)
- 종속성
    - 의존 관계 (Dependency Relationship) 관점에서 바라볼 수 있다.

### 의존 관계

- 의존 관계에는 항상 방향성이 존재하며 **방향성**을 기준으로 오브젝트가 변경되었을 때 의존 관계를 맺고 있는 다른 오브젝트에도 영향이 생긴다는 것이다.
- A가 B에 의존하고 있다 ( **A → B** )
    - B에 변화가 생긴다면 변화에 대한 영향이 A에도 있다.
    - 의존 관계에 대한 예시 → A에서 B의 메소드를 호출해서 사용하는 경우

> 구체적인 **클래스에 대한 의존관계**를 가지고 있는 경우 의존 관계의 정도가 강하다. 즉 결합도가 강한데 이는 소프트웨어 공학 관점에서 올바르지 않다. 그렇기 때문에 의존 관계에 대한 결합도를 낮추는 것이 중요하다.
>
- 의존 관계의 결합도를 낮추는 방법으로는 구체적인 클래스와의 의존 관계를 형성하는 것이 아닌 인터페이스와 의존 관계를 형성하도록 하는 것이다.

> The object does not look up its dependencies and does not know the location or class of the dependencies.
>

### 런타임 의존 관계

- 모델 , 설계 과정에서 형성된 것이 아닌 런타임 시점에 오브젝트 사이에 형성되는 의존 관계

```kotlin
@Service
class MemoService(
    private val memoRepository: MemoRepository, // Interface
    private val matchRepository: MatchRepository // Interface
)
```

- 런타임 시점에 형성되는 의존 관계는 무엇인가? (조건 2가지 ?)

  프로그램이 실행되기 전까지 구체적으로 의존 관계가 형성되는 오브젝트를 알 수 없다는 의미인데 기본적으로 런타임 의존관계는 인터페이스를 주입 받는 경우를 말한다.

  또한 생성자나 팩토리 메서드를 통해 주입을 받아야 한다.


### DI의 조건

1. 코드나 설계에는 런타임 의존 관계가 드러나지 않는다. 즉 인터페이스에 대한 의존관계가 형성되어야 한다.
2. 컨테이너나 팩토리 등 제 3의 존재가 의존 관계를 결정해준다. ( IOC 관점 )
3. 외부에서 사용할 오브젝트에 대한 레퍼런스를 제공(주입)

### DI의 정의

- 컨테이너나 팩토리등 제 3의 존재가 **오브젝트 사이의 런타임 의존 관계를 결정하고 오브젝트를 주입**해주는 것, 종속성을 결정하는 것
- 스프링에서 DI
    - 스프링에서 빈 오브젝트에 종속성을 주입해주기 위해서는 주입되는 오브젝트도 빈이여야 한다.
- **이유**

  스프링 프레임워크에서 빈간의 주입을 해주는 것은 컨테이너이다. 컨테이너에서 관리되는 오브젝트여야지만 주입이 가능하다.


### 스프링 컨테이너

- IOC 컨테이너이면서 DI 컨테이너
    - 즉 빈 오브젝트의 생명 주기를 관리하면서 빈 사이의 종속성을 주입한다.

### 생성자 기반 DI

```kotlin
class SimpleMovieLister(private val movieFinder: MovieFinder) {}
```

- 객체의 생성과 종속성 주입이 동시에 일어난다.
    - 생성자 호출 시점에 1회 호출되는 것이 보장됨
        - 주입 받는 빈이 null 일 수 있는 가능성이 배제된다.
        - 객체의 불변성 확보
    - 순환 참조 방지 가능
        - 순환 참조가 발생할 경우 컴파일 에러의 형태로 알 수 있기 때문에 수정하기 편리하다.
    - Test 하기 편리하다.
        - 필드 주입 방식의 경우 Spring 프레임워크 위에서만 테스트가 가능하다. 순수 java 코드로 테스트하기 위해서는 생성자 주입 방식이 필요하다.

### 수정자(Setter) 기반 DI

```kotlin
class SimpleMovieLister {
    lateinit var movieFinder: MovieFinder
}
```

- 실행 시점
    - 빈에서 No-args 생성자를 호출 한 후 컨테이너에서 빈의 수정자(setter)메소드를 호출한다.
- 수정자 주입은 기본적으로 선택적 종속성에 해당(변화 가능성)
    - 기본값을 설정해줘야만 불필요한 null 체크를 피할 수 있다.

### Bean 주입 과정

1. ApplicationContext 가 생성되고 초기화
2. 빈이 생성되고 의존성이 주입된다.
    - 생성자 주입의 경우에는 동시에 발생
    - 수정자 주입 , 필드 주입은 빈 생성 → 의존성 주입

> Singleton 범위에 사전 인스턴스화가 설정된 빈은 컨테이너가 생성되면서 빈이 생성되고 주입 , 그렇지 않으면 요청될 때 빈이 생성
>

### 순환 참조

> **순환참조 문제란 A 클래스가 B 클래스의 Bean 을 주입받고, B 클래스가 A 클래스의 Bean 을 주입받는 상황처럼 서로 순환되어 참조할 경우 발생하는 문제를 의미**
>
- 생성자 주입의 방식을 사용할 경우 어떠한 Bean도 생성하지 못하게 되어 무한 반복
- 필드 주입이나 수정자 주입의 경우 해당 오브젝트에 대한 메소드가 호출되어야 순환 호출의 문제가 발생
- 애초에 **설계 시점에 순환 참조가 일어나지 않도록 하는 것이 중요**하다.

### Lazy - initalized Bean

- 컨테이너가 생성되고 빈을 생성하는 것이 아닌 빈에 대한 요청이 들어왔을 때 빈을 생성하는 방식
- 스프링에서 권장하는 방식은 아니다.
- @Lazy 어노테이션 사용 → 순간적인 순환 참조를 방지할 수 있지만 말 그대로 회피 느낌이다.

### Autowiring

- 오토와이어링은 스프링이 빈의 요구사항과 매칭되는 애플리케이션 컨텍스트상에서 다른 빈을 찾아 빈 간의 의존성을 자동으로 만족하게 하도록 하는 수단
- 스프링에서 Autowirig을 제공하는 방법
    - 생성자 주입
    - 수정자 주입
    - @Autowired 기반의 필드 주입

### Singleton 빈에 Prototype 빈을 주입할 때

- 컨테이너는 싱글톤 빈 A를 한 번만 생성하므로 속성을 설정할 수 있는 기회는 한 번뿐입니다. 컨테이너는 필요할 때마다 Bean B의 새 인스턴스를 Bean A에 제공할 수 없습니다.
- 해결 방법

  해결책은 제어의 역전을 포기하는 것입니다. ApplicationContextAware 인터페이스를 구현하고 컨테이너에 대한 getBean("B") 호출을 수행하여 Bean A가 필요할 때마다 (일반적으로 새로운) Bean B 인스턴스를 요청함으로써 Bean A가 컨테이너를 인식하도록 할 수 있습니다.

- 다른 방법으로는 @Lookup 어노테이션을 활용하는 것

### Annotation 기반 설정

- 구성 요소간의 연결을 위해 바이트 코드 메타데이터에 의존하는 어노테이션 기반 설정

> Annotation injection is performed before XML injection.
>
- Annotation 기반 주입이 먼저 일어나고 XML 주입이 일어난다 !

### @Autowired

- Spring에서 의존성 주입을 위해 지원하는 어노테이션
- BeanPostProcessor의 구현체인 AutowiredAnnotationBeanPostProcessor가 빈의 초기화 라이프 사이클 이전, 즉 빈이 생성되기 전에 @Autowired가 붙어있으면 해당하는 빈을 찾아서 주입해주는 작업을 하는 것
- 찾는 순서

  타입 -> 이름 -> @Qualifier -> 실패


생성자

```kotlin
class MovieRecommender @Autowired constructor(
    private val customerPreferenceDao: CustomerPreferenceDao)
```

- 정의된 생성자가 1개인 경우 @Autowired 생략 가능

수정자

```kotlin
class SimpleMovieLister {
		@set:Autowired
    lateinit var movieFinder: MovieFinder
}
```

- 컴포넌트나 빈의 로드 순서를 정렬할 때 `@Order` 또는 `@Priority` 사용 가능
    - 같은 타입의 빈이 여러 개인 경우
- Autowired(required = false)
    - 해당 속성이 Autowiring 될 수 없는 경우 무시된다.
- @Autowired 어노테이션이 붙은 생성자가 여러 개인 경우
    - Bean으로 만족할 수 있는 가장 많은 종속성을 가지고 있는 생성자가 선택된다.

<aside>
📖 @Autowired , @Inject , @Value , @Resource 주석은 스프링의 BeanPostProcessor 구현에 의해 처리된다. 따라서 해당 주석을 사용자 정의 BeanPostProcessor 타입에 사용할 수 없다.

</aside>

### @Qualifier

- 빈의 이름을 사용하여 특정 빈을 주입하도록 할 수 있다.

```kotlin
class MovieRecommender {
    @Autowired
    @Qualifier("main")
    private lateinit var movieCatalog: MovieCatalog
}
```

- MovieCatalog의 구현체 중에서 main이라는 이름의 Bean을 주입하도록 한다.

### @Resource

- Spring은 `필드` 또는 빈 주입 `수정자 메소드`에 @Resource 어노테이션을 사용하여 주입을 지원한다. 즉 생성자에는 적용할 수 없다.
- Java에서 지원하는 어노테이션
- 찾는 순서

  이름 -> 타입 -> @Qualifier -> 실패


```kotlin
class SimpleMovieLister {
		@Resource(name="myMovieFinder") 
    private lateinit var movieFinder:MovieFinder
}
```

### @Value

- 외부 프로퍼티 파일의 속성을 주입하는데 사용한다.

```kotlin
@Component
class MovieRecommender(@Value("\${catalog.name}") private val catalog: String)
```

- 클래스 단에 @PropertySource를 사용하여 외부 프로퍼티가 정의된 파일을 찾아줄 수 있다.
