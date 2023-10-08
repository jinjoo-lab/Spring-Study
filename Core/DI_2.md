# DI-2

### StereoType - Annotation

- 아키텍처 관점에서 특정 계층에 사용하는 어노테이션
    - 명명을 통해 직관적으로 구조를 파악할 수 있다.
- 스프링은 자동으로 StereoType 어노테이션이 붙은 클래스들을 빈으로 등록한다.
    - @ComponentScan의 감지 대상
- AOP 적용시 관점 적용을 위해 역할에 맞게 사용해야 한다.

### 종류

- @Repository
    - 검사 되지 않은 예외(DAO 메소드에서 발생)를 Spring DataAccessException으로 변환 할 수 있게 해준다.
- @Controller
- @Service
- @Component

> @Repository, @Service 및 @Controller는 보다 구체적인 사용 사례(각각 지속성, 서비스 및 프레젠테이션 계층에서)를 위한 @Component의 특수화입니다
>

### @Component + @Bean VS @Configuration + @Bean

- @Compoent + @Bean 에서는 메소드 및 필드의 호출을 위해 CGLIB 프록싱을 사용하지 않는다.
- CGLIB 프록싱
    - @Configuration 클래스의 @Bean 메소드 내에서 메소드 또는 필드를 호출하여 협업 객체에 대한 빈 메타데이터 참조를 생성하는 수단
    - 스프링의 빈이 싱글톤으로 보장될수 있는 이유
        - **CGLIB**라는 라이브러리가 `@Configuration` 을 적용한 클래스를 상속받은 임의의 다른 클래스를 만들어 그 클래스를 스프링 빈으로 등록
    - Bean 메소드에 대한 직접 호출 시에 빈이 싱글톤으로 유지되는가 ? , 새로운 빈을 생성하는가에 대한 차이이다.

### @Inject

- Spring이 아닌 Java에서 제공하는 의존성 주입 어노테이션
- 생성자 , 수정자 , 필드 주입에 사용할 수 있다. ( @Autowired와 동일 )

### @Named , @ManagedBean

- @ComponentScan의 탐지 대상이며 @Component의 Java에서 제공하는 어노테이션이다.
- Custom Component 어노테이션을 만들 때에는 사용하지 못한다. 만들고 싶다면 StereoType 어노테이션 사용

### Spring Java Configuration

- @Configuration + @Bean
    - @Bean은 메소드가 Spring IoC 컨테이너에 의해 관리될 새 객체를 인스턴스화하는데 사용
    - @Configuration
        - 클래스에 사용 빈 정의의 경로임을 나타내는 목적 , 동일한 클래스내에 여러 빈 간의 종속성을 정의

```kotlin
@Configuration
class AppConfig {
	@Bean
	fun myService(): MyServiceImpl {
		return MyServiceImpl()
	}}
```

- @Configuration 이 붙은 클래스 자체가 빈으로 등록되고 내부의 @Bean 메소드도 빈으로 등록된다.

> 모든 @Configuration 클래스는 시작 시 CGLIB로 서브클래싱됩니다. 하위 클래스에서 하위 메소드는 상위 메소드를 호출하고 새 인스턴스를 작성하기 전에 먼저 캐시된(범위 지정) 빈이 있는지 컨테이너를 확인합니다.
>

### @Configuration 를 사용하지 않는 경우의 @Bean

- lite mode로 동작
- bean간의 종속성을 정의할 수 없다. → @Component 불가
- No CGLIB 프록싱

### AnnotationConfigApplicationContext

- 자바 설정 정보를 바탕으로 빈 객체의 설정 정보를 관리하고 사용자로 하여금 가져올 수 있다.

### Scope Proxy Mode

- @Scope(value = "prototype", proxyMode = ScopedProxyMode.TARGET_CLASS)

`ScopedProxyMode.TARGET_CLASS`, `ScopedProxyMode.INTERFACES` , `ScopedProxyMode.NO`

### @Import

- 설정 파일간의 계층 생성 시 사용
