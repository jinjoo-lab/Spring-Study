# AOP - 프록시

### 서론

> AOP의 등장 배경으로는 **핵심 기능**과 **부가 기능**의 분리였다. 그리고 AOP를 학습하다 보면 **프록시**라는 개념이 많이 등장한다. 오늘은 프록시에 대한 설명과 Spring AOP에서 프록시를 사용하는 방법에 대해 기술하겠다.
>

### 프록시

**목표** : 핵심 기능이 작성된 클래스와 부가 기능이 작성된 개체들을 **분리**해야 한다.

- **분리 과정에 있어 두 클래스(핵심 ↔ 부가)가 양방향으로 알아야 할까?**

  **NO ! 부가 기능을 담당하는 쪽에서만 핵심 기능의 개체 정보를 알면 된다.**


**구체화**

- 부가 기능이 핵심 기능을 사용하는 것처럼 만들어야 한다 !

**프록시**

- 클라이언트가 접근하려는 타켓인 것처럼 위장하여 요청을 대신 받는 대리자
    - 프록시는 **요청을 가로채 타깃에게 다시 요청을 위임**한다 !
- 프록시에서는 부가 기능만 수행 → 핵심 기능은 타깃이 수행

![Untitled](https://user-images.githubusercontent.com/84346055/273457666-afe368ee-6227-4b3d-86de-b1581ed0b7ce.png)

프록시 사용 목적

1. 클라이언트가 타깃에 **접근하는 방식을 제어 ( 프록시 패턴 )**
    - Ex ) JPA에서의 지연 로딩 → 직접 타깃에 접근하기 전까지 접근을 지연
2. 타깃에 **부가적인 기능을 부여 ( 데코레이터 패턴 )**
    - Ex ) 트랜잭션 , 시간 측정

### 프록시 적용

![Untitled](https://user-images.githubusercontent.com/84346055/273457672-7ba6f3c6-266c-4147-ab1a-429c8be87669.png)

- **클라이언트에게 타깃에 대한 레퍼런스를 넘긴다 할 때 !**

  실제 타깃 대신에 프록시를 넘긴다 !


> 프록시의 메소드를 통해 타깃 접근 시 → 타깃을 생성하고 요청 위임
>
- 프록시와 타겟을 동일한 인터페이스를 사용하도록 !

**간단한 예제**

- **User Interface**

![Untitled](https://user-images.githubusercontent.com/84346055/273457675-2b196394-d20b-48ac-b8c5-95b721bac237.png)

- **UserTarget (Target 클래스)**

![Untitled](https://user-images.githubusercontent.com/84346055/273457677-686a5120-d758-47e0-a751-54fb3f70b2f5.png)

- UserProxy (**Proxy 클래스)**

![Untitled](https://user-images.githubusercontent.com/84346055/273457678-9588077c-07d2-4bd4-bed8-7adc1064ecb8.png)

1. 프록시 클래스는 **타겟에 대한 레퍼런스**를 가지고 있어야 한다.
    1. 동일한 인터페이스를 구현하고 있기 때문에 인터페이스로 대체
2. 다음과 같이 say라는 메소드 호출 시
    1. 부가 기능을 사용하고 실제 메소드 호출은 타겟에게 위임 !

### 고찰

- 메서드의 개수가 1개가 아닌 **무수히 많아진다면 ?**
    - **코드의 길이가 길어지고 중복 코드가 발생 !**
- **매번 새로운 클래스**를 정의 !
    - 인터페이스 메소드 구현 + 위임 기능

### 동적 프록시

- 동적 프록시란 Proxy 객체를 직접 생성하는 것이 아닌 **Runtime중 Interface를 구현하는 인스턴스 생성**

### JDK Dynamic Proxy

- 프록시 클래스를 직접 구현하지 않아도 된다 !
- **Invocation Handler**를 통한 처리

**방식**

- Interface를 기반으로 **런타임 중 Proxy를 생성**하는 방식
    - **리플렉션**을 활용한 Proxy 클래스를 제공한다.
- `Java.lang.reflect.Proxy` 클래스의 `newProxyInstance()` 메소드를 이용해 프록시 객체를 생성

### 리플렉션 ( Reflection )

- 구체적인 클래스 타입을 알지 못하더라도 **클래스 정보에 접근**할 수 있는 자바 API

> `JVM`이 실행되면 사용자가 작성한 자바 코드가 컴파일러를 거쳐 바이트 코드로 변환되어 `JVM Memory`에 저장된다. `Reflection API`는 이 정보를 활용해 필요한 정보를 가져온다.
>

**클래스 정보**

- Class 클래스
    - 자바의 모든 클래스는 클래스 정보를 가지고 있는 **Class 타입 오브젝트**를 가지고 있다.
        - 클래스이름.class
        - 오브젝트.getClass()
- Method
    - Class 오브젝트에서 **메소드에 대한 정보**를 추출 !
    - Method 타입의 invoke() 메소드 호출 !

```java
import java.io.*;
import java.lang.reflect.InvocationTargetException;
import java.lang.reflect.Method;

public class Main {
    public static void main(String[] args) throws IOException, NoSuchMethodException, InvocationTargetException, IllegalAccessException {

        Class<BTS> mClass = BTS.class; // BTS 클래스의 Class 오브젝트

        Method method = mClass.getMethod("sayHello"); // 메소드 정보 가져오기

        method.invoke(new BTS()); // 메소드 정보를 바탕으로 메소드 호출
    }
}
class BTS
{
    public void sayHello()
    {
        System.out.println("안녕하세요 BTS입니다 !");
    }
}
```

**주의사항**

- 일반적으로 메소드를 호출 시 컴파일 시점에 분석된 클래스를 사용하지만 리플렉션은 **런타임에 클래스를 분석**
    - Type Check가 컴파일 시점에 불가능하다.
    - 속도가 느리고 Reflection API 자체의 비용이 크다.

**JDK Dynamic Proxy**

**Proxy 생성 방법**

```
Object newProxyInstance(ClassLoader loader,Class<?>[] interfaces,InvocationHandler h)
```

- 클래스 로더 , 타깃 인터페이스 , 타깃 정보가 있는 Handler

**동작 방법**

![Untitled](https://user-images.githubusercontent.com/84346055/273457679-d5199c87-a053-4241-b8c5-5a72a0eea34a.png)

- 다이내믹 프록시 → **런타임 시** 프록스 팩토리에 의해 만들어지는 동적 오브젝트 (프록시)
    - 다이내믹 프록시는 **타깃의 인터페이스와 같은 타입**
    - 다이내믹 프록시 객체에 InvocationHandler를 포함시켜 하나의 객체로 반환
        - 부가기능을 InvocationHandler에 작성
- **내가 만든 프록시하고의 차이 ?**

  부가기능에 대한 구현을 프록시 내부에 작성하는 것이 아니라 → **InvocationHandler에게 위임 !**

- **InvocationHandler**

```
public Object invoke(Object proxy, Method method, Object[] args)
```

1. 다이내믹 프록시는 클라이언트의 모든 요청을 리플렉션 정보로 변환
2. InvocationHandler 구현 오브젝트의 invoke() 메소드로 넘김

![Untitled](https://user-images.githubusercontent.com/84346055/273457681-88630c7b-aa8c-4072-a203-86bac3c771f4.png)

**사용 방법**

```java
public class UserHandler implements InvocationHandler {

    Object target;

    UserHandler(Object target)
    {
        this.target = target; // 다이내믹 프록시로부터의 요청을 다시 타깃에게 위임(타깃 오브젝트 주입)
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if(method.getName().equals("say"))
            System.out.println("프록시가 사용자의 say 감지");

        else
            System.out.println("프록시가 사용자의 eat 감지");

				return method.invoke(target,args); // 타깃에게 요청 위임
    }
}
```

**결과**

![Untitled](https://user-images.githubusercontent.com/84346055/273457683-5f69d16c-740b-4961-acdd-26892934fa8b.png)

### CGLIB

- **클래스의 바이트 코드**를 조작하여 **Proxy 객체를 생성**
    - 인터페이스가 아닌 타깃의 클래스에 대해서도 프록시 생성 가능 (상속을 이용하는 방식)

```
TeamService teamService = (TeamService) Enhancer.create(
                TeamService.class,
                new TeamInterceptor(new TeamService())
        );
```

![Untitled](https://user-images.githubusercontent.com/84346055/273457685-adb52dc8-96c2-481c-a1aa-f39aa36c4aa6.png)

**공식 문서**

> Classes in Java are loaded dynamically at runtime. *Cglib* is using this feature of Java language to make it possible to add new classes to an already running Java program.
>
- Hibernate의 지연 로딩
- Mockito의 mocking method

**MethodIntercepter**

```java
package com.example.toyjava.module.account.service;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

public class TeamInterceptor implements MethodInterceptor {

    private Object target;
    public TeamInterceptor(Object target)
    {
        this.target = target; // target을 주입받는다.
    }

    @Override
    public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {

        System.out.println("CGLIB Proxy 감지"); // 부가 기능
				return methodProxy.invoke(target,objects); // 위임
    }
}

// TeamService : class com.example.toyjava.module.account.service.TeamService$$EnhancerByCGLIB$$582c0b44
```

- 메소드가 처음 호출되는 경우 **동적으로 타깃의 클래스의 바이트 코드를 조작**
- 이후 호출 시 조작된 바이트 코드를 재사용

![Untitled](https://user-images.githubusercontent.com/84346055/273457686-46eece4d-9e45-4813-ae06-5d5e1555c7ab.png)

**CGLIB 고찰**

- 구체 클래스를 상속받기 때문에 final 키워드가 사용될 수 없다.
- 대상 클래스의 기본 생성자 필수
    - 생성자가 2번 호출된다.

### 서론 -2

> JDK 다이나믹 프록시와 CGLIB을 통해 우리는 부가 기능과 핵심 기능의 개체들을 분리할 수 있었다. 또한 위의 방법들은 런타임 중 동적으로 프록시 객체를 생성해주었기 때문에 불필요한 양의 코드 작성 또한 줄일 수 있었다.
>
- 그렇다면 우리에게 남은 과제는 이러한 프록시를 *‘어떻게 Spring에 녹여낼 것인가’* 이다.

### 문제점

- 한번에 **여러 개의 클래스에 공통적인 부가기능 제공** 할 수 없다.
    - JDK 다이나믹 프록시 , CGLIB의 핸들러와 인터셉터가 타겟 오브젝트를 프로퍼티로 가지고 있다.
        - 동일한 기능이라도 ? → **타겟이 다르다면 별도의 오브젝트를 또 만들어야 한다.**
- **스프링 관점 !**

  Handler를 Bean으로 등록한다 했을 때 **타겟의 개수만큼 중복 등록(생성)** → 비효율적

  **중복을 없애고 모든 타깃에 적용 가능한 싱글톤 빈으로 만들자 !**


> **중복을 없애고 모든 타깃에 적용 가능한 싱글톤 빈으로 만들자 !**
>

### ProxyFactoryBean

![Untitled](https://user-images.githubusercontent.com/84346055/273457688-85a5e615-b358-4d0f-b729-a2189799bd97.png)

- JDK Dynamic Proxy와 CGLIB 등 스프링이 사용하는 프록시 방식은 다양
    - 일관된 방법으로 **프록시를 만들수 있게 추상 레이어 제공**
- 클라이언트의 요청이 들어오면 인터페이스 유무에 따라
    - 있다면 → JDK Dynamic Proxy
    - 없다면 → CGLIB

**ProxyFactoryBean**

- **프록시를 생성해서 빈 오브젝트로 등록하게 해주는 팩토리 빈**
- 순수하게 프록시를 생성 , 부가 기능은 별도의 빈으로 둘 수 있다.
    - **프록시 추상화 기능 제공**

**MethodIntercepter**

- 이름은 동일하지만 CGLIB의 MethodIntercepter와는 조금 차이가 있다.
- MethodIntercepter는 ProxyFactoryBean으로부터 타깃 오브젝트에 대한 정보까지 함께 제공
    - 타깃 오브젝트에 상관없이 독립적 존재 가능 → **싱글톤 빈**으로 등록 가능

```java
@FunctionalInterface
public interface MethodInterceptor extends Interceptor {
    @Nullable
    Object invoke(@Nonnull MethodInvocation invocation) throws Throwable;
}

// MethodInvocation (콜백 오브젝트) : proceed() 메소드 호출 시 -> 타깃 오브젝트의 메소드를 내부적 실행
```

![Untitled](https://user-images.githubusercontent.com/84346055/273457689-3f1cba33-df38-41a6-b6d5-6900a28714be.png)

**예제**

**TeamAdvice**

```java
package com.example.toyjava.module.account.service;
import org.aopalliance.intercept.MethodInterceptor;
import org.aopalliance.intercept.MethodInvocation;

public class TeamAdvice implements MethodInterceptor {
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("MethodInterceptor 동작");
        invocation.proceed(); // 타깃 오브젝트 메소드 실행
        
				return null; // proceed()의 결과값을 반환하는 것도 가능
    }
}
```

**프록시 팩토리 빈 사용**

```java
public class ProxyTest {
    @Test
    public void test() {

        ProxyFactoryBean pf = new ProxyFactoryBean();
        pf.setTarget(new TeamService()); // 타켓 저장
        pf.addAdvice(new TeamAdvice()); // 부가 기능 저장 (별도의 싱글톤 빈으로 관리 가능)

        TeamService ts = (TeamService) pf.getObject(); // 프록시 객체 반환
        ts.testing();
     }
}
```

- **프록시 팩토리 빈 !**

  부가 기능을 담당하는 핸들러에서 타깃의 정보를 가지고 있지 않아도 되었다. → **타깃에 구애받지 않고 적용 가능**


### 마지막 욕심

- ProxyFactoryBean을 사용 시 타겟과 부가기능의 적용에 유연성은 확대했지만 결국 **개발자가 계속해서 생성해야 한다는 것이다.**
    - 불필요한 중복 작업이 방대 !
- **타깃 빈의 목록을 제공 → 스프링이 자동으로 타깃 빈에 대한 프록시를 만들어줘 !**

### 자동 프록시 생성기 (빈 후처리기)

- **빈 후처리기**의 사용
    - 스프링 빈 오브젝트가 생성되고 나서 → 빈 오브젝트를 다시 가공할 수 있다 !
        - 빈 오브젝트의 **프로퍼티를 강제 수정**
            - 아예 빈 자체를 바꿔치기 가능
        - 별도의 **초기화 작업 수행**
- **빈 후처리기 사용 !**

  스프링이 생성한 빈 오브젝트의 일부를 프록시로 포장 → 프록시를 빈으로 대신 등록하자 !

- `AnnotationAwareAspectJAutoProxyCreator` 라는 빈 후처리기가 스프링 빈으로 자동 등록됨 !

**동작 과정**

1. **생성**: 스프링이 스프링 빈 대상이 되는 객체를 생성
2. **전달**: 빈 후처리기에 생성된 객체를 전달
3. **모든 Advisor 빈 조회**: 자동 프록시 생성기가 스프링 컨테이너에서 모든 `Advisor`을 조회
4. **프록시 적용 대상 체크**: `Advisor`내에 있는 `Pointcut`을 이용해 모든 클래스와 메서드를 매칭 , 조건이 하나라도 만족하면 프록시 적용 대상
5. **프록시 생성**: 프록시 적용 대상이면 프록시를 생성하고 `Advisor` 연결
    - 컨테이너에 프록시를 전달 (바꿔치기)
        - 컨테니어는 최종적으로 빈 후처리기가 돌려준 오브젝트를 빈으로 등록 , 사용

### Spring Bean VS CGLIB

- 기본적으로 Bean 객체를 프록시로 바꿔치기 하는 과정에서 **비용**이 발생
    - Spring Context에서 관리되는 **빈이 모두 프록시 객체는 아니다 !**

### 스프링 컨텍스트 관점

![Untitled](https://user-images.githubusercontent.com/84346055/273457692-608a429e-3cd0-476e-80c1-675ac6faef38.png)

**참고 서적**

- 토비의 스프링 VOL 1 (AOP , 프록시)

**참고 블로그**

[https://www.youtube.com/watch?v=MFckVKrJLRQ&t=922s](https://www.youtube.com/watch?v=MFckVKrJLRQ&t=922s)

[https://www.baeldung.com/cglib](https://www.baeldung.com/cglib)

[https://woooongs.tistory.com/99](https://woooongs.tistory.com/99)

[https://velog.io/@hyun6ik/프록시-기술과-한계-CGLIB](https://velog.io/@hyun6ik/%ED%94%84%EB%A1%9D%EC%8B%9C-%EA%B8%B0%EC%88%A0%EA%B3%BC-%ED%95%9C%EA%B3%84-CGLIB)

[https://velog.io/@gmtmoney2357/스프링-부트-빈-후처리기와-프록시-자동-생성기](https://velog.io/@gmtmoney2357/%EC%8A%A4%ED%94%84%EB%A7%81-%EB%B6%80%ED%8A%B8-%EB%B9%88-%ED%9B%84%EC%B2%98%EB%A6%AC%EA%B8%B0%EC%99%80-%ED%94%84%EB%A1%9D%EC%8B%9C-%EC%9E%90%EB%8F%99-%EC%83%9D%EC%84%B1%EA%B8%B0)
