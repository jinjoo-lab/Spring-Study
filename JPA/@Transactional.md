# @Transactional

### 트랜잭션

- 데이터베이스의 상태를 변화시키는 하나의 **논리적 기능 수행 단위**
- 분해할수 없는 작은 단위의 작업
    - 전부 다 처리되거나 전부 다 취소되거나 (All Or Not)
- **스프링에서의 트랜잭션 ?**

  데이터 접근 기술에는 순수 JDBC를 사용하거나 Hibernate , Spring Data JPA등 ORM을 사용하는 등 다양한다.

  그렇다면 이렇게 다양한 **데이터 접근 방식에 따라 트랜잭션이 다르게 적용되야 할까?**


### ACID

**A** : **원자성(Atomicity)**

- All or Nothing : 한 트랜잭션 내의 모든 연산들이 **전부 수행**되거나 **전부 수행되지 않거나**
- DBMS 회복 모듈
    - 부분적으로 갱신한 트랜잭션 : 영향을 취소 (UNDO)
    - 완료된 트랜잭션 : 트랜잭션의 영향을 재수행 (REDO)

**C : 일관성(Consistency)**

- 트랜잭션이 수행되기 전에 일관된 상태 → 트랜잭션이 수행된 후에는 또 다른 일관된 상태를 가진다

**I : 고립성(Isolation)**

- 한 트랜잭션이 데이터를 갱신하는 동안 갱신 중인 **데이터를 다른 트랜잭션이 접근하지 못하도록** 해야함
    - 다수의 트랜잭션이 동시 수행 → 순서에 상관없이 하나씩 수행한 결과와 동일
- DBMS 동시성 제어 모듈
    - 트랜잭션의 고립성 보장
    - 고립 수준 (Isolation Level)

**D : 지속성(Durability)**

- 완료된 트랜잭션의 효과는 시스템이 고장나더라도 데이터베이스에 반영되어야한다.
- DBMS의 회복 모듈

### 공통의 동작방식

1. 트랜잭션을 **생성**
2. 정상 처리되었다면 **트랜잭션 commit**
3. 정상 처리되지 않았다면 **트랜잭션 rollback**

**기본 코드**

```
// 1. 트랜잭션 생성(가져오기)
Connection c = dataSource.getConnection();
c.setAutoCommit(false); // 트랜잭션을 시작하겠다는 뜻 ! (commit or rollback을 사용자 지정에 따라 처리)

try{
	PreparedStatement st1 = c.preparedStatement("update ~~~");
	st1.executeUpdate();

	PreparedStatement st2 = c.preparedStatement("update ~~~");
	st2.executeUpdate();

	c.commit(); // 2. 정상 처리되었다면 commit
}catch(Execption e){
	c.rollback(); // 3. 오류가 발생했다면 rollback
}finally{
	c.close();
}
```

- **트랜잭션에 대한 고찰**

  스프링에서 트랜잭션이란 기본적으로 데이터베이스에 대한 **connection을 공유하는 것**에서 작업이 수행된다.


**코드 (Connection을 공유)**

```
class UserService {
    public void upgradeAll() throw Exception {
        Connection c = dataSource.getConnection();
        c.setAutoCommit(false);
        try {
            upgradeSimple(c,user); // 트랜잭션을 담고 있는 Connection을 공유하기 위해 파라미터 전달
            c.commit();
        } catch (Execption e) {
            c.rollback();
        } finally {
            c.close();
        }
    }
}
```

- **문제점 !**
    1. 비즈니스 로직을 담고 있는 메소드에 connection 파라미터를 전부 전달 !
    2. 데이터 접근 기술이 변경된다면 ? → EntityManager 나 Session의 경우 코드가 전부 변경 !

### 트랜잭션 동기화

- 트랜잭션간의 공통의 connection을 공유하는 것
    - 이 과정에 있어 파라미터로 계속 connection을 넘기지 않아도 된다 !

> 트랜잭션을 시작하기 위한 **Connection 오브젝트를 저장소에 보관 이후** 호출되는 메소드들에서 접근 가능하도록 하자 !
>
- 동기화 과정에 있어 핵심은 **멀티 쓰레드 환경**에서도 안전하도록 작동하게 하는 것 !

### TransactionSynchronizationManager

- 스프링에서 제공하는 트랜잭션 동기화 관리 클래스
- DataSourceUtils → **Connection** 오브젝트 생성 + 트랜잭션 동기화에 사용할 수 있도록 **저장소에 바인딩 !**

```
class UserService {
    public void upgradeAll() throw Exception {
        TransactionSynchronizationManager.initSynchronization(); // 동기화 시작
        Connection c = DataSourceUtils.getConnection(datasource);
        c.setAutoCommit(false);
        try {
            upgradeSimple(user); // 더이상 connection을 파라미터로 공유 x
            c.commit();
        } catch (Execption e) {
            c.rollback();
        } finally {
            DataSourceUtils.releaseConnection(c,datasource); // DB 커넥션 종료
            TransactionSynchronizationManager.unbindResource(datasource);
            TransactionSynchronizationManager.clearSynchronization();
        }
    }
}
```

- **근본적 문제 !**

  데이터 접근 기술에 대하여 ….

    - 데이터 접근 기술은 JDBC , ORM에 따라 다르다 (Connection or EntityManager , Session)
    - 데이터 접근 기술에 종속적인 코드이다

> 데이터 접근 기술은 모두 트랜잭션 처리를 지원한다. 이러한 트랜잭셔션 처리를 **추상화된 트랜잭션 관리 계층**으로 만들자 !
>

### 트랜잭션 추상화

![Untitled](https://user-images.githubusercontent.com/84346055/275476148-8da446dc-b750-411e-9e01-c9b631019827.png)

- PlatformTransactionManager
    - 위의 동작 방식을 명세를 통한 Interface로 정의하여 추상화
    - 하위 구현체에서는 **데이터 접근 기술에 따라 트랜잭션 동기화 수행 !**
- 구현체를 주입받도록 코드를 작성한다면 데이터 접근 기술에 종속적이지 않은 코드 작성 가능 !

```java
public interface PlatformTransactionManager extends TransactionManager {
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

- AbstractPlatformTransactionManager

```
protected abstract Object doGetTransaction() throws TransactionException;
```

**적용 코드**

```
class UserService {
	
	PlatformTransactionManager transactionManager;
	
	UserService(PlatformTransactionManager transactionManager)
	{
		this.transactionManager = transactionManager;
	}
	public void upgradeAll() throw Exception {
		TransactionStatus status = this.transactionManager.getTransaction(
				new DefaultTransactionDefinition()
		);
		
		try {
			upgradeSimle(user); // 더이상 connection을 파라미터로 공유 x
			this.transactionManager.commit(status);
		} catch (Execption e) {
			this.transactionManager.rollback(status);
		} 
	}
}
```

- **마지막 문제 !**

  **AOP → 관심사의 분리**


### 고찰

- Connection을 동기화 하는 작업과 추상화하였지만 여전히 불필요한 코드가 존재한다 !
- Try ~ Catch 자체 !!
    - 위 기능은 User에 대한 비즈니스 로직을 담아야 하지만 **트랜잭션 처리 코드 또한 포함하고 있다 !**

## @Transactional

- 스프링에서 지원하는 트랜잭션 지원 방식 (선언적 트랜잭션)
    - Annotation 형태로 **스프링 AOP**를 통해 동작한다.
- Spring Configuration에 @EnableTransactionManager 어노테이션 필요 → **스프링 부트는 자동 추가**
- **CGLIB 바이트 코드 조작 방식** 사용
    - 타깃 클래스에 대해 프록시 적용

### 동작방식

1. 타깃 메소드 , 타깃 클래스 , 선언 메소드 , 선언 타입 (클래스 , 인터페이스) 순서대로 **@Transactional이 적용되었는지 확인**
2. 트랜잭션 처리 코드를 포함하여 프록시 객체로 반환
3. 프록시 객체에 접근하여 트랜잭션 처리 !

![Untitled](https://user-images.githubusercontent.com/84346055/275476168-28d12a25-5d3b-4b70-adf9-0bff08abfd1d.png)

### 트랜잭션 전파

- 트랜잭션의 **경계 설정**과 시작에 대한 **기준을 정하는 것**이다 !
    - 트랜잭션이 실행중일 때 다른 트랜잭션이 호출된다면 어떻게 처리할 것인지 결정
- @Transactional의 propagation 설정

**코드**

```java
@Service
public class TeamService{
    @Autowired
    private TeamRepository teamRepository;
    @Transactional()
    public void team1()
    {
        System.out.println("team1 : "+ TransactionSynchronizationManager.getCurrentTransactionName());
    }
}

@Service
public class MemberService {
    @Autowired
    private MemberRepository memberRepository;

    @Transactional
    public void member1()
    {
        System.out.println("member1 : "+ TransactionSynchronizationManager.getCurrentTransactionName());
    }
}
```

1. **Propagation.REQUIRED**
    - 진행중인 트랜잭션이 **없다면 새로 시작**하고 **있다면 참여 !** (default 속성)

```
@Test
@Transactional
public void test1()
{
   teamService.team1();
   memberService.member1();
}
```

> team1 : com.example.toyjava.**ToyJavaApplicationTests.test1**
member1 : com.example.toyjava.**ToyJavaApplicationTests.test1**
>
- 각 트랜잭션이 이미 진행된 test1 트랜잭션에 참여하는 것을 알 수 있다 !

1. **Propagation.REQUIRES_NEW**
    - 별도의 **트랜잭션이 보장**되어야 하는 코드에서 사용 !
    - 항상 새로운 트랜잭션을 시작한다.
        - 각각의 트랜잭션은 독자적 동작 → 서로에게 영향 x
    - *REQUIRES_NEW*는 별도의 새로운 트랜잭션(커넥션도 실제 다르다)을 만들 뿐, 쓰레드는 동일하다.
- 위 코드에서 team1() 메소드에 **Propagation.REQUIRES_NEW** 적용

> team1 : com.example.toyjava.module.account.**TeamService.team1**
member1 : com.example.toyjava.**ToyJavaApplicationTests.test1**
>

1. **Propagation.NESTED**
    - **중첩 트랜잭션** 적용 → Save Point 활용
        - 상위 트랜잭션의 커밋 , 롤백의 영향을 받음
        - 하위(적용된) 트랜잭션의 커밋 , 롤백은 상위 트랜잭션에 영향을 주지 못한다 !

![Untitled](https://user-images.githubusercontent.com/84346055/275476180-345ea7c6-981a-432c-a1ce-1c030d604137.png)

- **주의 !**
    1. 트랜잭션 전파는 **단순히 예외를 던지는 것과는 다른 영역**이다 !
        - 하위 트랜잭션에서 예외가 발생한다면 → 부모 트랜잭션에도 예외가 발생할 수 있다.
    2. Propagation도 @Transactional의 속성이기 때문에 **프록시 동작**이다!

**문제상황 1**

[[Spring] Transactional Propagation 정리하기](https://devlog-wjdrbs96.tistory.com/424)

**문제상황 2**

```

@Transactional
public void test1()
{
   test1();
}

@Transactional(propagation = Propagation.REQUIRES_NEW)
public void test2()
{
   System.out.println("test2 : "+ TransactionSynchronizationManager.getCurrentTransactionName());
}
```

- 동일한 클래스 내부에서 다른 트랜잭션을 호출하면 트랜잭션이 적용이 되지 않는다 !
    - 즉 트랜잭션 **전파 속성 또한 적용되지 않는다 !**

### 트랜잭션 격리

- 트랜잭션의 동시성을 정의
    - 트랜잭션이 수행될때 다른 트랜잭션이 동일한 데이터에 대해서 어떻게 보일지에 대한 범위
- @Transactional의 isolation 설정
- JPA상에서 DB의 격리 수준을 정의할 때 사용

1. **ISOLATION_DEFAULT**
    - DB자체에 **정의되어 있는 격리 수준을 그대로 따른다**는 것!
- **세션 별 격리 수준 분리**

  데이터베이스 전체적인 격리 수준을 설정할 수 있지만 하나의 요청 즉 하나의 세션에 대하여 세션 별로 트랜잭션 격리 수준을 다르게 적용할 수 있다 !

  이 개념이 @Transactional에서 격리 수준을 변경하는 것이다 ! (세션별 적용)


### 스프링 지원 격리 수준

```
DEFAULT(-1),
READ_UNCOMMITTED(1),
READ_COMMITTED(2),
REPEATABLE_READ(4),
SERIALIZABLE(8);
```

### Timeout

- 트랜잭션을 수행하는 제한시간 설정
- `@Transactional(timeout = 1000)`

### read-only

- 트랜잭션을 읽기 전용 모드로 설정

```
@Transactional(readOnly = true)
```

- 해당 옵션이 적용된다면 ?
    - 스냅샷 저장
    - Dirty-Check(변경감지)
- 위 두 작업이 수행되지 않는다. → 메모리 절약 가능 (성능 향상)
