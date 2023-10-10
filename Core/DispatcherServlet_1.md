# Dispatcher Servlet 1 - 프론트 컨트롤러 , 동작 방식

- 참고하면 좋은 블로그

[ServletContainer와 SpringContainer는 무엇이 다른가?](https://sigridjin.medium.com/servletcontainer와-springcontainer는-무엇이-다른가-626d27a80fe5)

### 주요 쟁점

- 톰캣 인스턴스와 Spring Bean의 연관관계

### Servlet

- 자바를 사용하여 동적으로 웹페이지를 생성하는 서버 단의 프로그램, 사양을 지칭(API)
    - 서버는 크게 web server 와 was로 구분할 수 있다. 차이점은 정적인 처리, 동적인 처리의 가능 여부
- 서블릿은 단독으로 동작할 수 없으며 서블릿 컨테이너를 통해서 관리됨
- 서블릿은 자바로 구현되어 있으며 GenericServlet이나 HttpServlet 클래스를 상속받아 구현된다.
    - 해당 클래스를 상속받음으로서 단순한 메소드 호출을 통해 사용자의 요청을 처리하고 데이터를 반환할수 있다.

### Servlet Container

- ServletContainer - per - process
- 서블릿 컨테이너는 서블릿의 생명 주기를 관리하고  요청마다 쓰레드를 통해 처리(Thread - per - request)

![Untitled](https://user-images.githubusercontent.com/84346055/273890775-4ac59426-0bd7-4d45-8737-111542e5c3e0.png)

- WAS(애플리케이션 서버)는 서블릿 컨테이너의 확장 개념

> **서블릿 컨테이너는 오직 서블릿 API 만 지원**하는 것을 말한다 (JSP, JSTL 까지 포함해서)
>
>
> **애플리케이션 서버는 Java EE (EJB, JMS, CDI, JTA, 서블릿 API) 의 전체를 지원**한다
>

![Untitled](https://user-images.githubusercontent.com/84346055/273890853-1314f626-0aeb-4dcb-a853-47994c663113.png)

### 서블릿 컨테이너에서 서블릿을 관리하는 과정

![Untitled](https://user-images.githubusercontent.com/84346055/273890858-772934ec-b5c9-454e-9f7e-36ca22778460.png)

1. 사용자의 요청(url)에 대해 적절한 서블릿을 매핑
2. 서블릿에 매핑되면 서블릿 컨테이너는 서블릿 인스턴스가 생성되었는지 체크하여 없을 경우 JVM에 의해 서블릿이 실행될 수 있도록 서블릿 인스턴스 생성
3. 서블릿 컨테이너는 init → service → destroy 순으로 메소드 호출
4. destroy : 서블릿 객체가 삭제되는 시점은 **웹서버에서 웹 애플리케이션 서비스가 중지되는 시점**

### HttpServlet( Abstract Class)

```
protected void doGet(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException
 {
        String msg = lStrings.getString("http.method_get_not_supported");
        sendMethodNotAllowed(req, resp, msg);
 }

protected void doPost(HttpServletRequest req, HttpServletResponse resp)
        throws ServletException, IOException 
{
        String msg = lStrings.getString("http.method_post_not_supported");
        sendMethodNotAllowed(req, resp, msg);
}
```

> 서블릿이 GET , POST 요청을 처리할 수 있도록 서버가 서비스 메소드를 호출합니다. 해당 메소드를 overriding 하여 다양한 form의 데이터를 다루고 반환할 수 있다.
>

### DispatcherServlet

- what is Dispatch
    - 무언가를 ‘보내다’ 라는 뜻을 가지고 있다 . 하지만 개인적으로 ‘할당하다’라는 의미가 더 이해하기 쉽다고 생각한다.
- 등장 배경
    - 클라이언트 요청(url)마다 서블릿을 생성하고 해당 컨트롤러에 요청을 보내야 했다. 그 과정에서 web.xml에 서블릿을 등록하고 매핑하는 과정의 양이 많고 복잡했다.
    - 이를 해결하기 위해 등장한 것이 FrontController 패턴이고 해당 패턴을 적용하여 탄생한 것이 DispatcherServlet이다.

### FrontController

[Front Controller Design Pattern - GeeksforGeeks](https://www.geeksforgeeks.org/front-controller-design-pattern/)

> application의 모든 요청은 앞단의 handler에 의해 처리되며 해당 handler에서 적절한 다음 handler로 요청이 발송된다.
>
- 서블릿 컨테이너의 제일 앞에서 모든 요청을 받아 처리( 세부 controller 로 요청을 위임)
- 프론트 컨트롤러를 제외한 나머지 컨트롤러들은 서블릿을 사용하지 않아도 됨
- 장점 : 한 곳에서 사용자의 모든 요청을 다룰수 있기 때문에 공통 패턴에 대한 적용이 중복적으로 사용되지 않을 수 있다.

### 한줄 정리

<aside>
❓ DispatcherServlet이 Bean으로 등록되어 package를 scan하고 @Controller, @RestController 애노테이션을 확인하여 어떠한 요청이 들어왔을 때 적절한 Handler Method에 위임

</aside>

![Untitled](https://user-images.githubusercontent.com/84346055/273890862-f1b44ef5-24d8-4ee0-af53-6ac154402ed0.png)

- 서블릿이다. (스프링 MVC의 중앙 서블릿)
    - 상속 형태 : DispatcherServlet → FrameworkServlet → HttpServletBean → HttpServlet

  ### FrameworkServlet

  > 서블릿 별로 WebApplciationContext 인스턴스를 관리한다. 서블릿의 구성은 서블릿의 네임 스페이스에 있는 빈에 의해 결정된다. 요청이 성공적으로 처리되었는지에 관계 없이 요청 처리시 이벤트를 게시합니다.


### 동작 흐름

![Untitled](https://user-images.githubusercontent.com/84346055/273890864-c22b2dda-7564-4ad3-b1de-549261ec3482.png)

1. DispatcherServlet으로 request가 들어온다 . request → HttpServletRequest
2. 공통 로직을 처리한 다음 HandlerMapping에 위임하여 요청을 처리할 Handler(Controller)를 찾는다.
3. 해당 Handler를 처리할 수 있는 HandlerAdapter를 탐색한다.
    - HandlerAdapter : @Controller or @RestController
4.  HandlerAdapter를 통해 핸들링 메소드를 실행한다. 반환값은 Model and View이다.
5. 반환 된 View 이름을 ViewResolver에 전달하고 ViewResolver는 해당 View객체를 반환한다.
6. DispatcherServlet은 반환된 View에게 Model(데이터)를 전달한다. 해당 결과가 종합된 화면이 클라이언트에게 제공된다.
7. DispatcherServlet은 위에서 종합된 결과를 클라이언트에게 HttpServletResponse형태로 제공한다.
- @Controller와 달리 @RestController를 사용하면 위의 과정중에 5,6이 제외된다.
    - MessageConverter를 이용

### 코드적 흐름

1. WAS는 DispatcherServlet에게 HttpServletRequest , HttpServletResponse 객체를 전달
2. doService() 메소드 호출
    - 내부적으로 doDispatch() 메소드 호출
- doService()
    - DispatcherServlet에 대한 작업만 수행 , HttpServletRequest 에 대한 공통 작업

      attribute setting


```
	@Override
	protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
		logRequest(request);
		// attribute setting 

		try {
			doDispatch(request, response); // 내부적으로 doDispatch 호출
		}
		finally {
			if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
				// Restore the original attribute snapshot, in case of an include.
				if (attributesSnapshot != null) {
					restoreAttributesAfterInclude(request, attributesSnapshot);
				}
			}
			if (this.parseRequestPath) {
				ServletRequestPathUtils.setParsedRequestPath(previousRequestPath, request);
			}
		}
	}
```

- doDispatch()
    - getHandler() 메소드를 호출하여 적절한 Controller를 찾는다.

```
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;
		boolean multipartRequestParsed = false;

		WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

		try {
			ModelAndView mv = null;
			Exception dispatchException = null;

			try {
				processedRequest = checkMultipart(request);
				multipartRequestParsed = (processedRequest != request);

				// Determine handler for the current request.
				mappedHandler = getHandler(processedRequest); // 적절한 Handler(Controller)를 찾음
				if (mappedHandler == null) {
					noHandlerFound(processedRequest, response);
					return;
				}

				// Determine handler adapter for the current request.
				HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

				// Process last-modified header, if supported by the handler.
				// resource cache 여부 확인 , 동일한 request 재요청시에 대한 선 처리
				String method = request.getMethod();
				boolean isGet = HttpMethod.GET.matches(method);
				if (isGet || HttpMethod.HEAD.matches(method)) {
					long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
					if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
						return;
					}
				}
				// intercepter의 선 처리 ( 통과할 경우 true , 다음 작업 실행 )
				if (!mappedHandler.applyPreHandle(processedRequest, response)) {
					return;
				}

				// Actually invoke the handler.
				mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

				if (asyncManager.isConcurrentHandlingStarted()) {
					return;
				}
				// view 를 찾고 model을 매핑 , @RestController일 경우 null이기 때문에 실행 x
				applyDefaultViewName(processedRequest, mv);
				// interceptor 의 후 처리
				mappedHandler.applyPostHandle(processedRequest, response, mv);
			}
			catch (Exception ex) {
				dispatchException = ex;
			}
			catch (Throwable err) {
				// As of 4.3, we're processing Errors thrown from handler methods as well,
				// making them available for @ExceptionHandler methods and other scenarios.
				dispatchException = new NestedServletException("Handler dispatch failed", err);
			}
			processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
		}
		catch (Exception ex) {
			triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
		}
		catch (Throwable err) {
			triggerAfterCompletion(processedRequest, response, mappedHandler,
					new NestedServletException("Handler processing failed", err));
		}
		finally {
			if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}
			else {
				// Clean up any resources used by a multipart request.
				if (multipartRequestParsed) {
					cleanupMultipart(processedRequest);
				}
			}
		}

```

- getHandler()
    - DispatcherServlet을 통해 등록되어있는 핸들러중 요청에 맞는 핸들러 정보 반환

```
protected HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
		if (this.handlerMappings != null) {
			for (HandlerMapping mapping : this.handlerMappings) {
				HandlerExecutionChain handler = mapping.getHandler(request);
				if (handler != null) {
					return handler;
				}
			}
		}
		return null;
	}
```

- getHandlerApdater()
    - 반환받은 핸들러 정보를 바탕으로 핸들러 어댑터를 반환

```
protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		if (this.handlerAdapters != null) {
			for (HandlerAdapter adapter : this.handlerAdapters) {
				if (adapter.supports(handler)) {
					return adapter;
				}
			}
		}
		throw new ServletException("No adapter for handler [" + handler +
				"]: The DispatcherServlet configuration needs to include a HandlerAdapter that supports this handler");
	}
```

```
mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
```

- 핸들러 어댑터에서 핸들링 메소드를 실행하여 request를 처리
- @Controller일 경우 ModelAndView 객체 반환
- @RestController일 경우 null 반환

> ModelAndView object with the name of the view and the required model data, or null if the request has been handled directly
>

### 코드 상의 DispatcherServlet 작동 순서

1. Dispatcher Servlet에서 doService() 호출
2. doService() 내부적으로 doDispatch() 호출
3. doDispatch()

   4 : getHandler()  : HandlerMapping

   5 : getHandlerApdater() : Handler 를 처리할 수 있는 HandlerAdapter를 찾는다.

   `Intercepteer preHandle`

   6 : handle() : HandlerApdater에서 해당 컨트롤러를 통해 요청 처리

   `Intercepter postHandle`

   7 : applyDefaultViewName :

   8 : processDispatchResult :

    <aside>
    ❓ @RestController일 경우 동작하지 않는다. mv값이 null이기 때문

    </aside>

- processDispatchResult()

```
private void processDispatchResult(HttpServletRequest request, HttpServletResponse response,
			@Nullable HandlerExecutionChain mappedHandler, @Nullable ModelAndView mv,
			@Nullable Exception exception) throws Exception {

		// Did the handler return a view to render?
		if (mv != null && !mv.wasCleared()) {
			render(mv, request, response);
			if (errorView) {
				WebUtils.clearErrorRequestAttributes(request);
			}
		}
}
```

- render()

```
protected void render(
    ModelAndView mv,
    HttpServletRequest request,
    HttpServletResponse response) throws Exception {

    View view;
    String viewName = mv.getViewName(); // view 의 논리 이름
    if (viewName != null) {
				// view의 논리 이름 ViewResolver에 전달하여 물리 이름으로 변환 , View 객체를 반환받는다.
        view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
    }
    try {
        if (mv.getStatus() != null) {
            response.setStatus(mv.getStatus().value());
        }
				// 반환받은 view 객체에 model(data)를 매핑
        view.render(mv.getModelInternal(), request, response);
    }
}
```

- resolveViewName()

```
@Nullable
protected View resolveViewName(String viewName, @Nullable Map<String, Object> model,
        Locale locale, HttpServletRequest request) throws Exception {

    if (this.viewResolvers != null) {
        for (ViewResolver viewResolver : this.viewResolvers) {
						// view 이름을 바탕으로 viewResolver를 통해 view 객체를 반환
            View view = viewResolver.resolveViewName(viewName, locale);
            if (view != null) {
                return view;
            }
        }
    }
    return null;
}
```

### Intercepter가 한번 더 동작을 한다고 ?????

```
// doDispatch()
if (asyncManager.isConcurrentHandlingStarted()) {
				// Instead of postHandle and afterCompletion
				if (mappedHandler != null) {
					mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
				}
			}

//applyAfterConcurrentHandlingStarted()
void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) {
		for (int i = this.interceptorList.size() - 1; i >= 0; i--) {
			HandlerInterceptor interceptor = this.interceptorList.get(i);
			if (interceptor instanceof AsyncHandlerInterceptor) {
				try {
					AsyncHandlerInterceptor asyncInterceptor = (AsyncHandlerInterceptor) interceptor;
					asyncInterceptor.afterConcurrentHandlingStarted(request, response, this.handler);
				}
				catch (Throwable ex) {
					if (logger.isErrorEnabled()) {
						logger.error("Interceptor [" + interceptor + "] failed in afterConcurrentHandlingStarted", ex);
					}
				}
			}
		}
	}
```

- 비동기 요청 시 afterConcurrentHandlingStarted가 수행됩니다.

### 참고

[DispatcherServlet - Part 1](https://tecoble.techcourse.co.kr/post/2021-06-25-dispatcherservlet-part-1/)

[DispatcherServlet - Part 2](https://tecoble.techcourse.co.kr/post/2021-07-15-dispatcherservlet-part-2/)
