# AOP

### 로직

- 비즈니스 로직 : 핵심 기능으로서 클래스 내에서 실행되는 전체적인 로직
- 인프라 로직 (부가기능) : 핵심 기능을 지원하는 기능 , 그 자체로 단일 사용은 하지 않는다.
    - 대표적인 부가 기능 : 로깅 , 시간 측정 , 트랜잭션

### SRP

- **단일 책임의 원칙**
    - SOLID 원칙 중 하나
        - 클래스가 변경되는 이유는 오직 하나 ! , 하나의 액터에 대해서만 책임을 가지고 있어야 한다.
        - 이상적인 OOP(객체 지향 프로그래밍)을 위해서는 지켜져야 한다.
- **비즈니스 로직이 작성된 클래스에 부가 기능에 대한 코드까지 있다면 ?**

  이상적인 OOP는 불가능하다. 또한 중복 코드가 발생하고 코드 수정시 불필요한 작업이 필요 !


### AOP

- AOP(Aspect-Oriented Programming)
    - 관점 지향 프로그래밍
- 부가 기능을 **횡단 관심사**로서 바라보고 모듈화하는 프로그래밍 패러다임
    - 비즈니스 로직과 부가 기능을 **분리**
        - 분리의 의미를 구체화 시켜보자면 비즈니스 로직을 담당하는 클래스는 부가 기능 담당 클래스에 대해 알지 못한다
- **장점**
    - 각 코드간의 결합도가 낮아진다.
    - 모듈화하기 때문에 재사용성이 증가
    - 불필요한 중복 코드 방지 , 수정 용이

> 스프링 공식 문서에 따르면 AOP는 OOP를 훼손하는 것이 아닌 지원하는 개념이라 기술하고 있다.
>

![Untitled](https://user-images.githubusercontent.com/84346055/273565821-8febe921-4392-4dc4-85e5-d29d042246d9.png)

### AOP의 적용 시점

**AspectJ**

1. **컴파일 시점**
    - 자바 코드의 컴파일 시점에 AOP를 적용하기 위해서는 컴파일러에 대한 별도의 처리가 필요하다.
2. **클래스 로딩 시점**
    - 자바를 실행할 때 특별한 옵션(java -javaagent)을 통해 클래스 로더 조작기를 지정해야 하는데, 이 부분이 번거롭고 운영하기 어렵다./

**Spring AOP**

1. **런타임 시점**
    - 런타임 시점에 AOP를 적용하기 위해서는 기본적으로 타겟의 코드는 그대로 유지되어야 한다.
    - **프록시 사용**
        - 타깃에 AOP를 한번 감싸는 방식으로 AOP를 지원한다 !

### 프록시

- **대리자**
    - 클라이언트가 특정 객체에 접근을 가로챈다. (접근 제어)
        - 객체의 핵심 기능을 수행하지는 않는다!
            - 객체의 로딩 시점을 미룰 수 있다 !

![Untitled](https://user-images.githubusercontent.com/84346055/273565843-85d8d2fc-9181-48b7-b36d-46d423c19503.png)

### AOP 주요 용어

- **Target**
    - 부가 기능을 부여할 대상
    - 스프링 AOP를 적용하기 위해서는 Target은 **Bean**이어야 한다.
- **JoinPoint**
    - 프로그램 실행 중 Advice가 적용될 수 있는 지점
        - **메소드** , 필드 등등 …
    - 스프링 AOP는 프록시 방식을 사용 → 조인 포인트는 **메소드 실행 지점으로 제한**
- **PointCut**
    - Adivce가 적용될 JoinPoint 선별 , AspectJ 표현식 사용
    - **메소드 실행 지점**만 포인트 컷으로 선별 가능
- **Advice**
    - 부가 기능
    - 스프링 AOP에서 제공하는 부가 기능
        - Around , Before , After Returning , After Throwing
- **Aspect**
    - Adivce + PointCut을 모듈화한 것 (@Aspect)

- **Spring AOP 프록시 사용 이유 ?**
    - AOP도 Bean이다. 객체의 역할로서만 존재하기 때문에 해당 클래스에는 Aspect와 JoinPoint들만 함수로 정의 되어 있고 Aspect와 Target을 연결해주는 역할이 없다 그렇기에 그런 역할을 Proxy 클래스가 대신해준다.

- AOP 프록시
    - AOP 기능을 구현하기 위해 만든 프록시 객체, 스프링에서 AOP 프록시는 JDK 동적 프록시 또는 CGLIB 프록시이다. → 참고로 스프링 부트 2.0 부터 CGLIB를 기본으로 사용

**예시 코드**

```java
@Slf4j
@Aspect
@Component
public class LogIntroduction {
    @Pointcut("execution(* com.dragonguard.backend..*Controller*.*(..))")
    public void allController() {
    }

    @Pointcut("execution(* com.dragonguard.backend..*Service*.*(..))")
    public void allService() {
    }

    @Pointcut("execution(* com.dragonguard.backend..*Repository*.*(..))")
    public void allRepository() {
    }

    @Before("allController()")
    public void controllerLog(JoinPoint joinPoint) {
        log.info(
                "METHOD : {}, ARGS : {}",
                joinPoint.getSignature().toShortString(),
                joinPoint.getArgs());
    }

    @Before("allService() || allRepository()")
    public void serviceAndRepositoryLog(JoinPoint joinPoint) {
        log.debug(
                "METHOD : {}, ARGS : {}",
                joinPoint.getSignature().toShortString(),
                joinPoint.getArgs());
    }
}
```
