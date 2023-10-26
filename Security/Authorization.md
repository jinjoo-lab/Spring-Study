# Authorization (6.1.3 기준)

### 권한

- 클라이언트가  **특정 서비스** , **리소스**에 접근 할 수 있도록 허용된 것

### 인가 (Authorization)

- 특정 클라이언트에 대해 권한을 허용하는 것 , **인증이 먼저 수행**되고 인가가 수행된다.
    - 실제로 FilterChain에서 AuthenticationFilter가 먼저 수행되고 AuthorizationFilter가 수행된다.

### 인가 수행

- **AuthorizationFilter**
    - 기존에 사용되던 FilterSecurityInterceptor는 Deprecated되었다.

[FilterSecurityInterceptor (spring-security-docs 6.1.3 API)](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/web/access/intercept/FilterSecurityInterceptor.html)

- An authorization filter that restricts access to the URL using [AuthorizationManager](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/authorization/AuthorizationManager.html).
    - URL 방식으로 접근하는 경우 **AuhtorizationFilter**가 동작하여 권한을 처리한다.

### 아키텍처

- **GrantedAuthority** 개체는 AuthenticationManager에 의해 인증 개체에 삽입되고 나중에 권한 부여 결정을 내릴 때 **AuthorizationManager** 인스턴스에서 읽혀집니다.

```
Authenication Interface
Collection<? extends GrantedAuthority> getAuthorities(); // 권한 목록
```

### GrantedAuthority

```java
public interface GrantedAuthority extends Serializable {
	String getAuthority();
}
```

- getAuthority() 메소드는 AuthorizationManager 인스턴스에서 권한의 문자열 표현을 얻는데 사용

### AuthorizationManager

```java
@FunctionalInterface
public interface AuthorizationManager<T> {

	default void verify(Supplier<Authentication> authentication, T object) {
		AuthorizationDecision decision = check(authentication, object);
		if (decision != null && !decision.isGranted()) {
			throw new AccessDeniedException("Access Denied");
		}
	}

	@Nullable
	AuthorizationDecision check(Supplier<Authentication> authentication, T object);

}
```

- Spring Security **Request 기반, Method 기반 및 Message 기반** **권한 부여 구성 요소**에 의해 호출되며 최종 액세스 제어 결정을 내리는 역할을 담당합니다.
- check
    - 인가 결정을 내리는데 필요한 모든 정보 전달하여 판단 → AuthorizationDecision 객체 반환
- verify
    - 내부적으로 check 함수 호출 → 반환된 AuthorizationDecision 객체를 바탕으로 인가 결정
        - 인가가 허용되지 않은 경우 **AccessDeniedException** 발생

![Untitled](https://user-images.githubusercontent.com/84346055/278261361-dd406500-6055-4f57-a430-649306694048.png)

- AuthorizationManager 의 구현체들

## 스프링 시큐리티의 권한 계층

- 클라이언트의 권한을 확인하여 접근을 제어하는 방법으로 **시큐리티는 3가지를 지원**한다.
1. 웹 계층
    - **URL 요청에 따른 단위**의 권한 관리
2. 서비스 계층
    - **메소드 단위**의 권한 관리
3. 도메인 계층
    - **객체 단위**의 권한 관리

### 웹 계층

- **HTTP Request 단위**로 권한을 확인하여 인가 처리

![Untitled](https://user-images.githubusercontent.com/84346055/278261385-cee9be08-e78f-4c9a-9c23-0dc989e7d696.png)

**동작 과정**

1. AuthorizationFilter는 SecurityContextHolder에서 Authentication 객체를 검색하는 공급자를 구성
2. Supplier<Authentication>과 HttpServletRequest를 AuthorizationManager에 전달
    - AuthorizationManager는 Request를 AuthorizeHttpRequests의 패턴과 일치하는지 확인하고 해당 규칙을 실행
3. Authorization 승인 시
    - `AuthorizationFilter` continues with the [FilterChain](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-filters-review) which allows the application to process normally.
4. Authorization 거부 시
    - `AccessDeniedException` is thrown. In this case the [ExceptionTranslationFilter](https://docs.spring.io/spring-security/reference/servlet/architecture.html#servlet-exceptiontranslationfilter) handles the AccessDeniedException.

```
@Override
	public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain)
			throws ServletException, IOException {

		HttpServletRequest request = (HttpServletRequest) servletRequest;
		HttpServletResponse response = (HttpServletResponse) servletResponse;

		try {
			AuthorizationDecision decision = this.authorizationManager.check(this::getAuthentication, request);
			this.eventPublisher.publishAuthorizationEvent(this::getAuthentication, request, decision);
			if (decision != null && !decision.isGranted()) {
				throw new AccessDeniedException("Access Denied");
			}
			chain.doFilter(request, response);
		}
		finally {
			request.removeAttribute(alreadyFilteredAttributeName);
		}
	}
```

- 위 코드는 **AuthorizationFilter의 doFilter() 메소드**이다. 내부적으로 주입받은 AuthorizationManager의 check() 메소드를 호출하는 것을 알 수 있다.

```
@Override
	public AuthorizationDecision check(Supplier<Authentication> authentication, HttpServletRequest request) {
		
		for (RequestMatcherEntry<AuthorizationManager<RequestAuthorizationContext>> mapping : this.mappings) {

			RequestMatcher matcher = mapping.getRequestMatcher();
			MatchResult matchResult = matcher.matcher(request);
			if (matchResult.isMatch()) {
				AuthorizationManager<RequestAuthorizationContext> manager = mapping.getEntry();
				
				return manager.check(authentication,
						new RequestAuthorizationContext(request, matchResult.getVariables()));
			}
		}
		return DENY;
	}
```

- 위 코드는 URL 권한 계층에서 사용되는 AuthorizationManager 인터페이스를 구현한 `RequestMatcherDelegatingAuthorizationManager` 이다.
- 내부적으로 RequestMatcher를 통해 **HttpRequest에 대한 처리**를 수행하는 것을 알 수 있다.
- **RequestMatcher의 정보는 무엇?**

  SecurityConfiguration에서 **.antMatchers("/admin/**").hasRole("ADMIN")**와 같은 메서드 체인 정보를 기반으로 생성


### AuthorizationFilter 위치

- Default 설정에 의하면 AuthorizationFilter의 위치는 FilterChain의 마지막에 위치한다.
- 즉 권한 부여는 앞 단의 FilterChain에서는 수행되지 않는다는 것이다.
- AuthorizationFilter에서 권한이 확인되어 인가 승인된 경우 요청은 **DispatcherServlet으로 전달**된다.

### Request - Spring Configuration

- HttpRequest 별로 승인

```
http
    .authorizeHttpRequests((authorize) -> authorize
        .requestMatchers("/resource/**").hasAuthority("USER")
        .anyRequest().authenticated()
    )
```

- Http 메소드 종류에 따라 승인

```
http
    .authorizeHttpRequests((authorize) -> authorize
        .requestMatchers(HttpMethod.GET).hasAuthority("read")
        .requestMatchers(HttpMethod.POST).hasAuthority("write")
        .anyRequest().denyAll()
    )
```

## 메서드 방식 인가 + 권한

- AOP 기반 동작
- 프록시와 어드바이스로 메소드 인가처리 수행
- @PreAuthorize(), @PostAuthorize(”hasRole(’USER’)”), @Secured()

### 인가처리를 위한 초기화 과정

1. 보안 설정 메소드 있는지 확인
2. 빈의 프록시 객체 생성
3. 인가 처리(권한 심사)를 수행하는 Advice 등록
4. 빈 참조시 실제 빈이 아닌 프록시 빈 객체 참조

### 진행 과정

1. 메소드 호출시 프록시 객체를 통해 메소드 호출
2. Advice가 등록되어 있다면 Advice를 작동하게 하여 인가 처리
3. 권한 심사를 통과하면 빈의 메소드를 호출
