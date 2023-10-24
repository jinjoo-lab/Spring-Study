# Authentication

![Untitled](https://user-images.githubusercontent.com/84346055/277730138-0dbb5307-62ab-4495-8212-08fe9c3ee06a.png)

### Authentication

- 사용자의 인증 정보를 저장하는 토큰 (인증 정보 + 권한)
    - 인증 시 : id와 password를 담고 인증 검증을 위해 전달되어 사용
    - 인증 후 : 인증 결과를 담고 **SecurityContext**에 저장되어 **전역적으로 참조가 가능**하다.

```
Authentication authentication = SecurityContextHolder.getContext().getAuthentication()
```

- 구조
    - **Principal** : 사용자 id or User 객체를 저장
        - 여기서의 User객체란 UserDetails를 구현한 클래스를 의미한다.
    - **credentials** : 사용자 비밀번호
    - **authorities** : 인증된 사용자의 권한 목록
    - **details** : 인증 부가 정보
    - **Authenticated** : 인증 여부

### SecurityContext

- **Authentication 객체가 저장**되는 임시 보관소
- **ThreadLocal** (Thread마다 할당된 고유 공간)
    - 로직 상에서 필요할 때마다 Authentcation 객체를 꺼내어 참조 가능

> Spring Security에서는 특정 시점에 SecurityContext를 사용할 수 없도록 자체를 삭제한다.
>
- **시점 ?**

  **클라이언트의 인증 요청을 처리한 후** 해당 Authentcation 객체는 더이상 쓸모가 없다.

- **삭제**

  **SecurityContextHolderFilter**

    - 기존의 SecurityContextPersistenceFilter의 경우 세션 방식에서 사용된다.
        - HttpSession에 SecurityContext를 저장

[JWT 자격 검증 시, SecurityContext는 언제 비워(clear)질까?](https://itvillage.tistory.com/60)

```
private void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain)
			throws ServletException, IOException {
		if (request.getAttribute(FILTER_APPLIED) != null) {
			chain.doFilter(request, response);
			return;
		}
		request.setAttribute(FILTER_APPLIED, Boolean.TRUE);
		Supplier<SecurityContext> deferredContext = this.securityContextRepository.loadDeferredContext(request);
		try {
			this.securityContextHolderStrategy.setDeferredContext(deferredContext);
			chain.doFilter(request, response);
		}
		finally {
			this.securityContextHolderStrategy.clearContext();
			request.removeAttribute(FILTER_APPLIED);
		}
	}
```

### SecurityContextHolder

- SecurityContext 객체를 보관하고 있는 wrapper 클래스
    - SecurityContext 의 유지 방식을 결정한다 → Thread와 관련
- **3가지 mode**
    - MODE_THREADLOCAL
        - 스레드 당 SecurityContext 할당 ( Default )
    - MODE_INHERITABLETHREADLOCAL
        - 메인 스레드와 자식 스레드에서 동일한 SecurityContext 유지
    - MODE_GLOBAL
        - 응용 프로그램에서 단 한개의 SecurityContext 유지
