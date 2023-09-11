# 영속성 컨텍스트

### 목차

- 영속성 + 사전 지식
- 엔티티
    - 정의
    - 생명 주기
- 영속성 컨텍스트
    - 정의
    - 관계
    - 트랜잭션과의 관계
    - 영속성 컨텍스트의 이점(특징)
        - 지연 로딩
        - 1차 캐시
        - 쓰기 지연
        - 변경 감지
        - 동일성 보장
- 코드

### 영속성

- (Persistence) 데이터를 생성한 프로그램이 종료되더라도 데이터가 사라지지 않는 특성
- 영속성을 유지하기 위해서는 ?
    - 파일 시스템이나 데이터 베이스를 사용하여 데이터를 관리해야 한다.

### 엔티티

- 데이터 베이스의 테이블과 1:1로 대응되는 객체 지향 관점의 클래스
    - 데이터 베이스 테이블의 필드 값을 모두 포함한다.
- **질문 : 엔티티와 DTO , DAO 에 대한 설명**

  엔티티 : 데이터 베이스의 테이블을 클래스 관점에서 기술한 것 ( 객체 지향 관점 ) → POJO

  DTO : Data Transfer Object

    - 계층간 데이터 전송에 사용되는 객체
    - DTO는 순수하게 데이터를 저장하고, 데이터에 대한 getter, setter 만을 가져야한다고 한다. **DTO는 어떠한 비즈니스 로직을 가져서는 안되며**, 저장, 검색, 직렬화, 역직렬화 로직만을 가져야 한다고 한다.

  DAO : Data Access Object

    - 데이터베이스의 데이터에 접근하기 위한 객체 , 해당 데이터를 Entity로 접근한다.
    - JPA Repository는 DAO 의 역할을 수행한다.

  [http://egloos.zum.com/aeternum/v/1160846](http://egloos.zum.com/aeternum/v/1160846)


### 엔티티 생명주기

![Alt text](https://user-images.githubusercontent.com/84346055/267040798-c8ca6b26-180d-4421-937a-3f8e9980990d.png)

- 비영속(new/transient)
    - 영속성 컨텍스트에 의해 관리되고 있지 않은 상태 + 식별자가 할당되지 않은 상태
- 영속(managed)
    - 영속성 컨텍스트에 저장된 상태
    - 영속성 컨텍스트에서 제공하는 이점, 기능은 대부분 **영속 상태의 엔티티에 대해서만 적용**된다.
- 준영속(detached)
    - 영속성 컨텍스트에 저장되었다가 분리된 상태
    - **식별자가 있지만** 영속성 컨텍스트와는 연결이 되어 있지 않은 상태
- 삭제(removed)
    - 삭제 상태의 엔티티는 ID값이 있고 영속성 컨텍스트에 연결되어 있으며 **데이터 베이스에서 제거**되도록 예약되어 있는 상태
- **질문 : 삭제 ? ? ? ?**

  (공식 문서) 삭제 상태의 **엔티티는 ID값이 있고 영속성 컨텍스트에 연결**되어 있으며 **데이터 베이스에서 제거되도록 예약**되어 있습니다.

    - 저게 무슨 말이지 ????

      findByID로 조회는 성공하지만 값은 빼올수 없다.

      [https://www.youtube.com/watch?v=kJexMyaeHDs&t=608s](https://www.youtube.com/watch?v=kJexMyaeHDs&t=608s)


### 영속성 컨텍스트

- 엔티티를 영구 저장하는 환경 ( 논리적인 개념 )
- 애플리케이션과 데이터 베이스 사이에서 객체를 보관하는 가상의 데이터베이스 역할
- **질문 : 영속성 컨텍스트가 필요한 이유 ? → 데이터베이스와의 관점**

  영속성을 유지하기 위해서는 **데이터 베이스와의 상호작용**이 필요한데 이러한 작업을 효율적으로 관리하고 엔티티의 상태를 관리하는데 좋기 때문이다.

  엔티티의 **변경을 바로 데이터 베이스에 반영한다면 비용적 측면에서 부담이 심하다**. 그렇기 때문에 변경을 추적해줄 대상이 있어 데이터 베이스에 대한 접근을 효율적으로 수행


> **EntityManager** 인스턴스는 **영속성 컨텍스트와 연결**됩니다. 영속성 컨텍스트는 영구 엔터티 ID에 대해 고유한 엔터티 인스턴스가 있는 **엔터티 인스턴스 집합**입니다. 영속성 컨텍스트 내에서 엔터티 인스턴스와 해당 **생명 주기가 관리**됩니다. EntityManager API는 영구 엔터티 인스턴스를 생성 및 제거하고 기본 키로 엔터티를 찾고 엔티티를 쿼리하는 데 사용됩니다.
>
- 영속성 컨텍스트는 영속성이 부여된 **엔티티의 생명 주기를 관리하는 엔티티 인스턴스들의 집합**
- 영속성 컨텍스트는 엔티티를 **식별자(@Id)**로 구분

### 영속성 컨텍스트의 생명주기 → 트랜잭션

- 영속성 컨텍스트는 일반적으로 **트랜잭션** 범위 내에서 작동한다.
1. 트랜잭션 시작: **트랜잭션이 시작되면 영속성 컨텍스트가 생성**됩니다.
2. 엔티티의 상태 변화:  **엔티티의 상태가 변경되면(추가, 수정, 삭제 등), 영속성 컨텍스트는 해당 변경 사항을 추적**합니다. → 상태 추적은 **영속 상태인 엔티티**만 해당
3. 영속성 컨텍스트의 관리: **영속성 컨텍스트는 트랜잭션 범위 내에서 엔티티를 관리**합니다. 이는 엔티티의 상태를 추적하고, 엔티티 간의 관계를 유지합니다. 이때 엔티티의 변경은 영속성 컨텍스트 내에서만 유효하며, **데이터베이스에는 아직 반영되지 않습니다.**
4. 트랜잭션 커밋: **트랜잭션이 커밋되면, 영속성 컨텍스트는 변경된 엔티티를 데이터베이스에 동기화**하여 저장합니다.
5. 트랜잭션 롤백: 트랜잭션이 롤백되면, 영속성 컨텍스트는 변경 내용을 취소하고 이전 상태로 되돌립니다. 따라서 데이터베이스에는 변경 사항이 반영되지 않습니다.
6. 트랜잭션 종료: **트랜잭션이 완료되면 영속성 컨텍스트도 함께 종료**됩니다. 이때 영속성 컨텍스트는 비워지고, 엔티티의 생명 주기도 종료됩니다.

### Lazy Loding(지연 로딩)

- 엔티티가 실제로 사용될 때까지 접근을 지연하는 것, 즉 **실제 사용하는 시점에 데이터베이스에서 필요한 데이터를 가져오는 것**
- fetch = FetchType.LAZY

> Hibernate에서는 데이터 베이스의 데이터를 로드할 때 일부분만 로드하도록 제공합니다.
>
- **지연로딩은** **어떻게 가능한걸까??**

  정답은 **‘프록시’**

- **지연 로딩은** **왜 필요할까?**

  비용 측면에서 바라볼 수 있다.


### 프록시

- Hibernate에서는 지연 로딩을 지원하기 위해 프록시 객체를 사용한다. 프록시 객체는 실제 클래스의 상속 객체로서 클래스의 형태는 동일하다.
- EntityManager.**getReference**() 메소드를 사용하면 프록시 객체를 얻을 수 있다.

### 프록시 초기화

- 실제 객체의 메서드를 호출 하기 위해 **select 쿼리를 사용하여 실제 객체를 데이터 베이스에서 조회해오고 참조 값을 저장**
- 프록시는 실제 객체를 참조하는 것이지 실제 객체로 변환되는 것은 아니다 !
- 구조적으로 프록시의 초기화는 영속성 컨텍스트를 거치게 된다.
    - 즉 **영속 상태**가 아니라면 프록시 초기화는 이루어지지 않는다. **`LazyInitializationException`**

### 도식화

![Alt text](https://user-images.githubusercontent.com/84346055/267040822-d4d54337-ac60-46c9-806c-8bf075552763.png)

- 프록시 객체의 데이터에 접근 시 **영속성 컨텍스트**가 데이터 베이스로부터 조회하여 실제 Entity 객체를 생성하고 해당 객체를 참조한다.

### code

**Member**

```
@Entity
public class Member {
    @ManyToOne
    @JoinColumn(name = "team_id")
    Team team;
}
```

**Team**

```
@Entity
public class Team {
    @OneToMany(fetch = FetchType.EAGER || LAZY,mappedBy = "team")
    List<Member> members = new LinkedList<>();
}
```

**Test**

```
Optional<Member> tmp = memberRepository.findById(1l);
```

- **findById와 getById의 차이점 ?**

  getById : 기본적으로 프록시 객체를 반환한다. (실제 데이터 접근이 일어날때까지 select 문은 호출되지 않는다) → **getReferenceById()**

  findById : select문 바로 호출


### 즉시 로딩

![Alt text](https://user-images.githubusercontent.com/84346055/267040827-a685a233-96cc-4b77-97c5-62c17b5f50c4.png)

- join을 통해 연관관계가 있는 team의 정보도 가지고 온다.

### 지연 로딩

![Alt text](https://user-images.githubusercontent.com/84346055/267040834-ce531a94-a288-4a10-8ab5-9f720b0ff1de.png)

- team에 대한 정보는 가지고 있지 않다.
- `tmp.get().getTeam().getName()` 호출 시 ( 연관된 Team에 대한 직접 접근 시 )

![Alt text](https://user-images.githubusercontent.com/84346055/267040837-52ad0dfe-a35e-42cb-bb31-db420679d0df.png)

- select 문을 통해 데이터베이스에 접근한다.

### 식별자

- 위의 테스트 상황에서 만약 `tmp.get().getTeam().getId()`가 호출된다면 ?
- **나타나는 결과와 왜 그런지 ?**

  결과 : team을 조회하는 select문이 호출되지 않고 바로 team에 대한 Id를 가져올 수 있다.

  왜 : **Id를 이용한 조회시 프록시 객체가 초기화 되지 않기 때문이다.**


### AbstractLazyInitalizer

- 프록시 객체 내부의 프록시 초기화 로직을 담당하는 추상 클래스

```
    @Override
	public final Serializable getIdentifier() {
		if ( isUninitialized() && isInitializeProxyWhenAccessingIdentifier() ) {
			initialize();
		}
		return id;
	}
```

> Id 식별자 접근시 프록시를 초기화 하는 옵션을 켰을 때 true가 되는 조건문 기본적으로 false `hibernate.jpa.compliance.proxy`
>

- `tmp.get().getTeam().id` 는 ?
    - null 반환
- **이유는**

  자바 빈 규약에 맞는 get + Id 형태일 경우에만 getIdentifier() 가 호출된다.

  만약 `findId`와 같이 getter에 대한 자바 빈 규약을 만족시키지 못하거나, `getTeamId`와 같이 식별자 이름과 매칭되지 않을 경우 getIdentifier 메서드를 호출하지 못하고 프록시가 초기화

  **id값은 프록시가 아니라 프록시 내부의 인터셉터에 들어있고, 프록시 객체가 가진 필드값들은 모두 null**이라는 것이다. 필드가 public이어서 바로 접근한다면 null에 접근하게 되는 것입니다.


### equals() 난제

```
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Team team = (Team) o;

        return Objects.equals(id, team.id);
    }

    @Override
    public int hashCode() {
        return id != null ? id.hashCode() : 0;
    }
```

```
Team t1 = member.get().getTeam();
Team t2 = member.get().getTeam();
System.out.println(t1.equals(t2)); // false
```

- **이유는 ?**

  정답은 **프록시** 차이 t1 객체는 equals() 메소드를 호출할 때 프록시 초기화된다. 하지만 t2 객체는 여전히 프록시 객체, 즉 getClass() 값이 다르기 때문에 둘의 동등성은 보장되지 않는다.

  → instanceOf를 사용하도록 하자


[https://tecoble.techcourse.co.kr/post/2022-10-17-jpa-hibernate-proxy](https://tecoble.techcourse.co.kr/post/2022-10-17-jpa-hibernate-proxy/)

### 동일성 보장

- 영속성 컨텍스트에 찾는 Entity가 이미 있으면, em.getReference()해도 실제 Entity가 반환된다-> JPA는 동일 객체임을 보장하기 위해 프록시를 먼저 조회하면 프록시 객체로 맞추고, 아니면 실제 Entity에 맞추는 내부로직을 가지고 있음
- ( JPA는 하나의 트랜잭션에서 같은 객체의 조회는 항상 동일 객체를 보장한다!)

### 1차 캐시

- JPA의 1차 캐시는 EntityManager가 관리하는 엔티티 매니저 내부의 캐시(Persistent Context)를 말한다.
    - 엔티티 매니저는 Thread-safe 하지 않기 때문에 per Thread 로 바인딩된다.
    - 1차 캐시의 동작 범위는 **트랜잭션의 범위**와 동일하다.
        - 트랜잭션이 종료될 경우 저장된 캐시도 모두 소멸된다.
    - 1차 캐시는 (Id, Instance)의 Map 형태이다.

![Alt text](https://user-images.githubusercontent.com/84346055/267040839-d7c8fdc0-5f44-4982-85cd-c9492f6eb7f8.png)

### 동작 과정

1. 식별자(PK)를 기준으로 **1차 캐시에 데이터가 있는지 확인** , 있으면 데이터를 가져온다.
2. 1차 캐시에 데이터가 없으면 **데이터 베이스에서 데이터를 요청**
3. 데이터 베이스에서 가져온 데이터를 **1차 캐시에 저장**

### Example

```
    @Test
    public void test2()
    {
        Optional<Member> member = memberRepository.findById(1l);
        Optional<Member> member2 = memberRepository.findById(1l);
    }
```

- **위의 경우에는 캐시가 적용되지 않아 Select 문이 2번 호출된다. 이유는 ?**

  정답은 **“트랜잭션”** , 캐시의 동작 범위는 동일한 트랜잭션 내에서 지속된다. @Transactional 이 붙지 않아 2번의 조회마다 별도의 트랜잭션이 수행되기 때문이다.


### 1차 캐시와 식별자

- 1차 캐시에서의 데이터 **반환** 조건은 오직 PK(식별자)를 이용한 조회다.
- 주의 ! : 1차 캐시에 데이터를 저장하는 것이 아닌 **1차 캐시로부터 데이터를 반환**하는 조건이다.

### Example

```
    @Test
    @Transactional
    public void test2()
    {
        Optional<Member> member = memberRepository.findByName("진주원");
        Optional<Member> member2 = memberRepository.findById(1l);
    }
```

- 위 예시에서 Select 문 쿼리는 1번만 실행된다. 첫번째 member를 조회할 때 캐시에 엔티티가 저장되어 두번째에 Id로 이용한 조회를 실행했기 때문이다.

### DefaultLoadEventListener

- 하이버 네이트의 **캐시 작업을 담당**하는 클래스(LoadEventListener의 구현체)
- 이론적으로 하이버 네이트에서 해당 데이터에 대한 조회 순서
    - 1차 캐시 → 2차 캐시 → 데이터 베이스
1. loadFromSessionCache(): 해당 키를 가진 엔티티가 1차 캐시에 존재하는지 확인한다. 존재한다면 그 엔티티를 반환한다.
2. loadFromSecondLevelCache(): 1차 캐시에 엔티티가 존재하지 않는다면, 2차 캐시에 같은 방식으로 엔티티 존재 여부를 확인하고, 존재한다면 그 엔티티를 반환한다.
3. loadFromDataSource(): 2차 캐시에 엔티티가 존재하지 않는다면, 데이터소스(DB)에 SQL을 실행하여 ResultSet을 얻고, 여기서 데이터를 읽어 Object[] 를 생성한다. 여기까지가 엔티티가 '읽혀진' 상태가 된다.
4. 읽어온 Object 를 1차 캐시와 2차 캐시에 저장한다. 그리고 이 Object를 엔티티 인스턴스로 변환하고, 이를 1차 캐시에 저장한다. **즉 dirty checking을 위한 데이터는 사실 엔티티로 저장되지 않는다.** (이 저장된 데이터가 후에 데이터 변경 여부 확인(dirty checking)에 그대로 사용된다.)

```
private Object doLoad(
			final LoadEvent event,
			final EntityPersister persister,
			final EntityKey keyToLoad,
			final LoadEventListener.LoadType options) {

		final EventSource session = event.getSession();
		final boolean traceEnabled = LOG.isTraceEnabled();
		if ( traceEnabled ) {
			LOG.tracev(
					"Attempting to resolve: {0}",
					MessageHelper.infoString( persister, event.getEntityId(), session.getFactory() )
			);
		}

		CacheEntityLoaderHelper.PersistenceContextEntry persistenceContextEntry = CacheEntityLoaderHelper.INSTANCE.loadFromSessionCache(
				event,
				keyToLoad,
				options
		); // 1차 캐시로부터 확인
		Object entity = persistenceContextEntry.getEntity();

		if ( entity != null ) {
			return persistenceContextEntry.isManaged() ? entity : null;
		}
		// 2차 캐시로부터 확인
		entity = CacheEntityLoaderHelper.INSTANCE.loadFromSecondLevelCache( event, persister, keyToLoad );
		if ( entity != null ) {
			if ( traceEnabled ) {
				LOG.tracev(
						"Resolved object in second-level cache: {0}",
						MessageHelper.infoString( persister, event.getEntityId(), session.getFactory() )
				);
			}
		}
		else {
			if ( traceEnabled ) {
				LOG.tracev(
						"Object not resolved in any cache: {0}",
						MessageHelper.infoString( persister, event.getEntityId(), session.getFactory() )
				);
			}
			entity = loadFromDatasource( event, persister ); // 데이터베이스로부터 조회
		}

		if ( entity != null && persister.hasNaturalIdentifier() ) {
			final PersistenceContext persistenceContext = session.getPersistenceContextInternal();
			final PersistenceContext.NaturalIdHelper naturalIdHelper = persistenceContext.getNaturalIdHelper();
			naturalIdHelper.cacheNaturalIdCrossReferenceFromLoad(
					persister,
					event.getEntityId(),
					naturalIdHelper.extractNaturalIdValues(
							entity,
							persister
					)
			);
		}

		return entity;
	}
```

### 쓰기 지연

- 영속성 컨텍스트에서 엔티티에 대한 변경 감지시, 해당 변화를 바로 데이터베이스에 반영하는 것이 아닌 SQL 쿼리를 버퍼에 저장하고 **flush 시점에 SQL 쿼리를 데이터 베이스에 보냄**
- flush 시점
    - 직접 flush 호출 시
    - JPQL 실행 시
    - 트랜잭션 commit

![Alt text](https://user-images.githubusercontent.com/84346055/267040841-91d847cb-3979-468b-9395-1091a44758b1.png)

### 변경 감지(dirty checking)

- **update() 어디갔어 ?**
    - SimpleJpaRepository나 EntityManager를 확인해봤을 때 이상한 점
        - save , delete는 있는데 update가 없다 ?
- 변경 감지란 **트랜잭션 범위 안에서 엔티티에 변경**이 일어났을 때 변경 내용을 **자동적으로 데이터베이스에 반영**하는 것
- **변화를 어떻게 감지하는 것일까?**

  정답은 “스냅샷”

- 영속성 컨텍스트에 영속 상태로 **처음 진입하는 엔티티(조회)의 스냅샷**을 함께 저장한다.
- 엔티티가 변경될 때마다 변경된 기록이 영속성 컨텍스트에 기록
    - flush 시점에 영속성 컨텍스트 내의 엔티티와 스냅샷을 비교하여 Update 쿼리가 전송
- 기본적으로 Update 쿼리는 대상의 모든 필드에 대해 실행된다.
    - @DynamicUpdate 사용 : 실제 값이 변경된 컬럼에 대해서만 update 쿼리 실행

### JPQL

- JPQL을 이용한 쿼리문 사용시 **영속성 컨텍스트**를 거치지 않는다. 이 때 SQL 임시 저장소에 쿼리문이 존재할 때 JPQL 쿼리문과 관련된 엔티티에 대해서만 flush가 진행된다.

[https://www.youtube.com/watch?v=kJexMyaeHDs&t=854s](https://www.youtube.com/watch?v=kJexMyaeHDs&t=854s)

## Code

- 영속 상태 만들기
    - entityManager.persist(member);
- 삭제 상태 만들기
    - entityManager.remove(member);
    - Session : managed + detached 상태의 엔티티를 삭제할 수 있다. (하이버네이트)
    - EntityManager : managed 상태의 엔티티만 삭제할 수 있다. (자카르타)

### 식별자를 이용한 다중 조회

```
List<Person> samePersons = session
		.byMultipleIds( Person.class )
		.enableSessionCheck( true )
		.multiLoad( 1L, 2L, 3L );
```

- `enableSessionCheck( true )` : 영속성 컨텍스트에 이미 로드된 엔터티를 건너뛰도록 Hibernate에 지시

### Filtering entities and association

- 정적인 방법 : 매핑 될 때 정의되며 런타임 변경이 불가능하다.
    - @Where
    - @WhrerJoinTable
- 동적인 방법 : 런타임에 적용될 수 있다.
    - @Filter
    - @FilterJoinTable

### @Where

- 엔티티나 컬렉션(연관관계)에 대하여 select 에 대한 where문 설정 시 사용

```
@Where(clause = "account_type = 'DEBIT'")
@OneToMany(mappedBy = "client")
private List<Account> debitAccounts = new ArrayList<>();

@Where(clause = "account_type = 'CREDIT'")
@OneToMany(mappedBy = "client")
private List<Account> creditAccounts = new ArrayList<>();
```

- 데이터베이스의 칼럼에 대해 필터링을 통해 특정 데이터만 추출할 수 있다.
- 엔티티 타입 자체에도 지정할 수 있다. (대표적인 예시)

  account에 대해 @Where(clause = “active = true”) 사용


### @Filter

- filtering information is not stored in the second-level cache.

### Refresh

- 현재 데이터베이스의 상태를 영속성 컨텍스트에 반영할 때 사용

### 준영속

- em.detach(entity)
    - 특정 엔티티만 준영속 상태로 전환
        - 1차 캐시에서 빠진다라고 생각하면 쉬움
- em.clear()
    - 영속성 컨텍스트 완전히 초기화
    - 테스트 케이스 작성시 사용
- em.close()
    - 영속성 컨텍스트를 종료

### 준영속 → 영속

- em.merge()
    - 준영속 상태의 엔티티를 새로운 영속 상태의 인스턴스로 복사하는 과정

![Alt text](https://user-images.githubusercontent.com/84346055/267040843-a7e33728-dace-45d2-8f16-72bca1706b13.png)

- 비영속 엔티티에 대해서도 병합 진행 가능
    - 데이터베이스에서 발견하지 못하면 새로운 엔티티 생성하여 병합하기 때문

[Spring 다중 데이터소스 설정 및 트랜잭션 동기화](https://yousrain.tistory.com/50)
