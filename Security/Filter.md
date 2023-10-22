# Spring Security (인증)

### Spring Security

- 인증(Authentication) , 인가(Authorization)를 지원하는 프레임워크
- **Filter**의 흐름에 따라 인증과 인가를 처리

### 인증 (Authentication)

- 사용자의 **신원을 입증**하는 과정
    - 회원가입과 로그인을 통해 사용자를 입증한다.

### 인가

- 인증된 사용자에 대한 **자원 접근 권한** 확인 (권한 확인)
    - 인가를 위해서는 인증이 먼저 수반되어야 한다.

> 사용자는 서비스 형태에 따라 종류가 다양할 수 있고 사용자 유형에 따라 다른 접근 권한이 필요할 수 있다. 해당 자원에 대한 접근을 확인하기 위해서는 인가가 필요하다.
>
- **권한**

  인가란 말 그대로 **권한 확인, 부여**이다. 특정 서비스에 대한 진입이 가능하도록 하는 것이 권한의 역할이다.


### Filter

> 스프링 시큐리티에서는 인증과 인가에 대한 처리를 **Filter의 흐름**에 따라 처리한다.
>
- 디스패처 서블릿 앞 단에 존재하여 URL에 대한 요청을 가장 먼저 접근 , 응답은 가장 마지막에 처리

![Untitled](https://user-images.githubusercontent.com/84346055/277170019-51485424-23e2-4407-bc26-89f70ae6850b.png)

- **Filter는 Bean일까? 아닐까?**

  Filter는 기본적으로 스프링 컨텍스트에 위치하지 않는다. **웹 컨텍스트(서블릿 컨텍스트)**에 위치한다.

  스프링 빈으로 등록은 되지만 **톰캣과 같은 웹 컨테이너**에 의해 관리된다.

  [[Spring] 필터(Filter)가 스프링 빈 등록과 주입이 가능한 이유(DelegatingFilterProxy의 등장) - (2)](https://mangkyu.tistory.com/221)


![Untitled](https://user-images.githubusercontent.com/84346055/277170026-9071181c-9066-4049-bfe9-46f2cf1f025e.png)

**Filter Interface**

```java
public interface Filter {

    public default void init(FilterConfig filterConfig) throws ServletException {}

    public void doFilter(ServletRequest request, ServletResponse response,
            FilterChain chain) throws IOException, ServletException;

    public default void destroy() {}
}
```

- doFilter의 실행 지점은 웹 컨테이너임을 매개변수를 통해 알 수 있다.
- **FilterChain**
    - Filter들은 기본적으로 chain 구조를 가지고 있다. 자신의 요청을 처리하고 다음 대상(필터)에게 요청을 전달하는 구조이다.
- **왜 스프링은 인증의 과정을 Filter에서 수행할까?**

  핵심은 **웹 컨텍스트와 스프링 컨텍스트의 영역 차이**이다. 사용자에 대한 식별을 스프링 컨텍스트에서 진행한다면?

  ***집 안으로 들어와 누군지 확인하는 것과 같은 것이 아닐까?***

- **핵심 전략**

  **Filter는 서블릿 컨테이너단의 기술이지만 인증 과정에 대한 구체적인 로직은 스프링 기반의 기술로 사용하고 싶어 !**


### **DelegatingFilterProxy**

> **요청이 오면 DelegatingFilterProxy가 요청을 받아서 우리가 만든 필터(스프링 빈)에게 요청을 위임**
>
- **대리자 역할의 서블릿 필터 (서블릿에서 관리되는 프록시 필터)**
    - springSecurityFilterChain의 이름으로 생성된 빈을 Application Context 에서 찾아 **요청을 위임**
    - 실제 보안 처리 로직은 수행하지 않는다. → 말 그대로 위임만 담당
- **수행 과정**
    1. Filter 구현체가 스프링 빈으로 등록
    2. ServletContext가 Filter 구현체를 가지는 DelegatingFilterProxy 생성
    3. ServletContext가 DelegatingFilterProxy를 서블릿 컨테이너에 필터로 등록
    4. Request에 대하여 DelegatingFilterProxy가 Filter 구현체에게 요청을 위임
        - **Filter 구현체 → FilterChainProxy**

```
public class DelegatingFilterProxy extends GenericFilterBean {

	@Nullable
	private String contextAttribute;

	@Nullable
	private WebApplicationContext webApplicationContext;

	@Nullable
	private String targetBeanName;

	private boolean targetFilterLifecycle = false;

	@Nullable
	private volatile Filter delegate;

	private final Object delegateMonitor = new Object();
    
    
  @Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		// Lazily initialize the delegate if necessary.
		Filter delegateToUse = this.delegate;
		if (delegateToUse == null) {
			synchronized (this.delegateMonitor) {
				delegateToUse = this.delegate;
				if (delegateToUse == null) {
					WebApplicationContext wac = findWebApplicationContext();
					if (wac == null) {
						throw new IllegalStateException("No WebApplicationContext found: " +
								"no ContextLoaderListener or DispatcherServlet registered?");
					}
					delegateToUse = initDelegate(wac);
				}
				this.delegate = delegateToUse;
			}
		}

		invokeDelegate(delegateToUse, request, response, filterChain);
	}
    
  protected void invokeDelegate(
			Filter delegate, ServletRequest request, ServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {

		delegate.doFilter(request, response, filterChain);
	}
```

### FilterChainProxy

- 인증 , 인가에 대한 **각 필터들을 호출**하여 순서대로 처리
    - 자동으로 등록되는 필터가 있고 사용자가 정의한 필터 추가 가능
- DelegatingFilterProxy로부터 요청을 받아 실제 인증, 인가 처리
    - 마지막 필터까지 인증 및 인가 예외가 발생하지 않는다면 성공적으로 로직 처리

```
@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {
    private final JwtFilter jwtFilter;
    private final JwtAccessDeniedHandler jwtAccessDeniedHandler;
    private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;

    @Bean
    public SecurityFilterChain configure(HttpSecurity http) throws Exception {
        return http.cors(cors -> cors.disable())
                .csrf(csrf -> csrf.disable())
                .httpBasic(h -> h.disable())
                .formLogin(f -> f.disable())
                .authorizeHttpRequests(
                        x ->
                                x.requestMatchers(
                                                "/v3/api-docs/**",
                                                "/swagger-ui/**",
                                                "/swagger-ui.html",
                                                "auth/**")
                                        .permitAll())
                .authorizeHttpRequests(
                        x -> x.requestMatchers("/test/**").permitAll().anyRequest().authenticated())
                .sessionManagement(x -> x.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
                .exceptionHandling(
                        x -> {
                            x.authenticationEntryPoint(jwtAuthenticationEntryPoint);
                            x.accessDeniedHandler(jwtAccessDeniedHandler);
                        })
                .build();
    }
}
```

### Spring Security Architecture (인증)

![Untitled](https://user-images.githubusercontent.com/84346055/277170028-d51182f5-f8e3-43b7-bac2-df82af7063ae.png)

### 스르이 시큐리티 인증 동작 구조

1. **AuthenticationFilter** : 인증을 담당하는 필터
    - Request에서 **사용자 정보를 추출**
    - 자격 증명 과정을 바탕으로 **UsernamePasswordAuthenticationToken** 개체 생성
        - Authentication을 상속받은 클래스 → SecurityContext에 저장되는 Authentication 객체

> **개발자가 직접 Filter를 구현하여 추가**할 수 있다.
>

```
@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
			throws ServletException, IOException {
		if (!this.requestMatcher.matches(request)) {
			if (logger.isTraceEnabled()) {
				logger.trace("Did not match request to " + this.requestMatcher);
			}
			filterChain.doFilter(request, response);
			return;
		}
		try {
			Authentication authenticationResult = attemptAuthentication(request, response);
			if (authenticationResult == null) {
				filterChain.doFilter(request, response);
				return;
			}
			HttpSession session = request.getSession(false);
			if (session != null) {
				request.changeSessionId();
			}
			successfulAuthentication(request, response, filterChain, authenticationResult);
		}
		catch (AuthenticationException ex) {
			unsuccessfulAuthentication(request, response, ex);
		}
	}
```

**attemptAuthentication**

```
private Authentication attemptAuthentication(HttpServletRequest request, HttpServletResponse response)
			throws AuthenticationException, ServletException {

		// request에서 정보를 바탕으로 Authentication 객체 생성
		Authentication authentication = this.authenticationConverter.convert(request);
		
		if (authentication == null) {
			return null;
		}
		
		AuthenticationManager authenticationManager = this.authenticationManagerResolver.resolve(request);
		Authentication authenticationResult = authenticationManager.authenticate(authentication);
		if (authenticationResult == null) {
			throw new ServletException("AuthenticationManager should not return null Authentication object.");
		}
		return authenticationResult;
	}
```

**successfulAuthentication**

```
private void successfulAuthentication(HttpServletRequest request, HttpServletResponse response, FilterChain chain,
			Authentication authentication) throws IOException, ServletException {
		SecurityContext context = this.securityContextHolderStrategy.createEmptyContext();
		context.setAuthentication(authentication); // SecurityContext에 인증 정보 저장
		this.securityContextHolderStrategy.setContext(context);
		this.securityContextRepository.saveContext(context, request, response); // session 을 사용할 경우
		this.successHandler.onAuthenticationSuccess(request, response, chain, authentication);
	}
```

1. **AuthenticationMangaer**
    - Interface → 인증을 처리하는 방법을 정의한 API
    - 위 인터페이스를 구현한 개체 → ProviderManager

```
Authentication authenticate(Authentication authentication) throws AuthenticationException;
```

- 파라미터 Authentication : 인증되지 않은 정보
- 반환 Authentication : 인증된 정보
1. **ProviderManger**
    - AuthenticationProiver 목록을 위임받는다.
- **목록 ?**

  **인증 방식에는 여러 형태가 존재**한다.

  서로 다른 인증 방식을 하나의 AuthenticationManager를 통해 처리할 수 있도록 목록을 가지고 있는 것이다.


```
	// 생성자에서 AuthenticastionProvier 객체들을 주입받는다.
	public ProviderManager(List<AuthenticationProvider> providers, AuthenticationManager parent) {
		Assert.notNull(providers, "providers list cannot be null");
		this.providers = providers;
		this.parent = parent;
		checkState();
	}

	@Override
	public Authentication authenticate(Authentication authentication) throws AuthenticationException {
		Class<? extends Authentication> toTest = authentication.getClass();
		AuthenticationException lastException = null;
		AuthenticationException parentException = null;
		Authentication result = null;
		Authentication parentResult = null;
		int currentPosition = 0;
		int size = this.providers.size();
		for (AuthenticationProvider provider : getProviders()) {
			
			if (!provider.supports(toTest)) {
				continue; 
				// AuthentcationProvier 목록중 해당 Authentication 인증 방식을 지원하는지 확인
			}
			
			try {
				result = provider.authenticate(authentication); // 지원한다면 인증 !
				if (result != null) {
					copyDetails(authentication, result);
					break;
				}
			}
		throw lastException;
	}
```

1. **AuthenticationProvider**
    - 인증 방식에 따른 **인증을 수행**하는 개체

```
public interface AuthenticationProvider {
	
	Authentication authenticate(Authentication authentication) throws AuthenticationException;

	boolean supports(Class<?> authentication);

}
```

- authenticate
    - AuthenticationManager의 authenticate 메소드와 동작 방식은 일치한다.
        - 파라미터로 인증이 완료되지 않은 Authentication 객체를 받아 인증이 완료되었다면 완료된 Authentication 객체를 반환한다.
- supports
    - 앞서 설명했든 인증 방식은 다양하고 해당 방식들이 정의된 필터들이 Chain형태로 관리된다고 말했다.
    - 해당 필터 방식이 지원하는 인증방식인지를 확인하는 메소드이다.
