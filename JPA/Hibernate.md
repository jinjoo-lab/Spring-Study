# JPA -2 (Hibernate)

## Hibernate

- JPA의 구현체 중에 하나
- Hibernate를 통해 **Repository 계층**을 관리 → **객체 지향적**으로 데이터를 관리

### 아키텍처

![Untitled](https://user-images.githubusercontent.com/84346055/276440173-8331b1ee-1a65-4a61-83ac-ab3263e45175.png)

- **Configuration**
    - 응용 프로그램 초기화 중 **한 번만 생성**
    1. Datatbase Connection
        - 데이터베이스 **Connection 관리**를 하나 이상의 구성 파일을 통해 처리

       | hibernate.connection.driver_class
       The JDBC driver class. |
       | --- |
       | hibernate.connection.url
       The JDBC URL to the database instance. |
       | hibernate.connection.username
       The database username. |
       | hibernate.connection.password
       The database password. |
       | hibernate.connection.pool_size
       Limits the number of connections waiting in the Hibernate database connection pool. |
    2. Class Mapping Setup
        - Java Class 와 데이터베이스 테이블간의 기본적 연결

![Untitled](https://user-images.githubusercontent.com/84346055/276440186-1524ba88-fa18-4176-995c-14cb7a9f03c6.png)

- **SessionFactory (EntityManagerFactory)**
    - 데이터베이스 당 하나의 SessionFactory
    - Session 객체를 얻기 위한 팩토리 메소드 제공 → 싱글 톤 인스턴스
- **Session (EntityManager)**
    - 응용 프로그램과 데이터베이스 사이 상호 작용 (인터페이스 제공) → 물리적 연결
        - 수명이 짧은 객체 (JDBC 연결 래핑)
        - **1차 수준 캐시(영속성 컨텍스트)**를 보유
    - 데이터 처리를 위한 다양한 메소드 제공
- **Transaction**
    - 데이터의 영속성을 달성시키고 롤백 기능 또한 제공

---

### Persistent Object

- RDB 테이블의 데이터를 자바의 POJO로 표현한 것

> POJO란 객체지향적인 원리에 충실하면서, 환경과 기술에 종속되지 않고 필요에 따라 재활용될 수 있는 방식으로 설계된 오브젝트
>

### 1차 , 2차 캐시

- 1차 캐시
    - Session객체가 데이터베이스와 상호작용하는 동안 유지되는 기본 캐시 (영속성 컨텍스트)
    - **트랜잭션이 시작하고 종료**하는 동안만 유효 !
- 2차 캐시
    - Application 단위의 캐시
        - 사용할 경우 데이터 조회 시 2차 캐시 참조후 없으면 데이터베이스에 접근
        - 동시성 극대화 → **캐시 객체의 복사본 생성 후 반환**

[[JPA & Hibernate] First Level Cache & Second Level Cache](https://velog.io/@dnjscksdn98/JPA-Hibernate-First-Level-Cache-Second-Level-Cache)

---

# Domain Model

### 매핑 유형

- Hibernate는 아래 유형으로 데이터를 구분
    - **Value Type**
        - 기본 타입
        - Embeddable 타입
        - Collection 타입
    - **Entity Type**

## Value Type

- 원시 자료형과 Wrapper 클래스와 몇가지 기본 클래스를 포함
- @Basic을 사용하지만 묵시적으로 인식되기 때문에 생략 !

```java
@Entity(name = "Product")
public class Product {

	@Id
	@Basic
	private Long id;

	@Basic
	private String name;
}
```

### @Column

- 데이터베이스의 컬럼과 직접 매핑 시 사용하는 Annotation
- **속성**
    - name=””
    - unique=false, nullable=true
    - insertable=true, updatable=true
    - columnDefinition=””
        - 데이터베이스 컬럼 정보를 직접 줄 수 있다.
    - length=255
        - String 타입에만 사용
    - precision=0, scale=0
        - precision은 소수점을 포함한 전체 자리수를, scale은 소수의 자리수다.

### @Formula

- **조회용**으로 사용 → 서브 쿼리를 포함할 때 사용 !
    - **해당 필드에 접근 시 미리 지정해놨던 쿼리가 수행 !**

![Untitled](https://user-images.githubusercontent.com/84346055/276440188-460309e1-b8de-450f-8e1d-727784028d07.png)

- **Hibernate가 기본 값을 SQL에 매핑하는 방식 !**

  **리플렉션**을 사용 → 최악의 경우 Serializable인 경우 binary 직렬화를 통해 처리


### @Enumerated

- 열거형 매핑에 사용
- 옵션
    - ORDINAL : 열거형의 본래 값인 숫자로 저장 → 오류 발생 위험이 다분히 높다 (권장하지 않는다)
    - STRING : **열거형 자체의 문자열로 저장**

### 알아두면 좋은 변환 타입

- LocalDate ⇒ `DATE`
- LocalDateTime ⇒ `TIMESTAMP`
- LocalTime ⇒ `TIME`
- UUID ⇒ `BINARY , VARCHAR`

### Embeddable & Embedded

- **세분성**에 의해 필드들을 또 다른 객체로 분리할 수 있다. → 주소가 대표적인 예
- 하지만 **RDB에서는 해당 필드들을 전부 분리해서 가지고 있기 때문에 연결해주는 작업**이 필요하다 !
- Embedded 장점
    - 재사용
    - 높은 응집도
    - 객체로 관리되기 때문에 별도의 메소드를 추가

**주소**

```java
@Embeddable
public class Address
{
		private String road_address;
		private String address1;
		private String address2;
}
```

**사용자**

```java
@Entity
public class User
{
		private String name;
		private String nick_name;
		@Embedded
		private Address address;
}
```

데이터베이스 관점에서는 User 테이블에 Address의 3개의 필드가 저장된다.

**주의 !**

- 하나의 엔티티 클래스 내에 **동일한 Embedded Type 이 2개 이상**인 경우 별도의 처리를 하지 않으면 오류

```
@Embedded
@AttributeOverride(name = "road_address", column = @Column(name = "home_road"))
@AttributeOverride(name = "address1", column = @Column(name = "home_address1"))
@AttributeOverride(name = "address2", column = @Column(name = "home_address2"))
private Address home_address;
```

## Entity Type

- 엔티티 POJO 작성 시 !
    - @Entity Annotation 명시
    - enum이나 interface는 사용할 수 없다 !
    - 기본 생성자가 **Public 혹은 Protected**

> **JPA 프록시 객체**는 결국 엔티티 클래스의 상속분이기 때문에 상속 가능한 Public 이나 Protected로 정의해야 한다. → **다음 장 상세히**
>

### 주요 어노테이션

- @Id → 식별자 PK 지정
- @Entity → 엔티티임을 명시 !
- **엔티티에서 Equals와 HashCode 정의 ?**

  트랜잭션 범위 내에서 영속성 컨텍스트는 **동일한 ID에 대해 항상 같은 객체**를 가져온다 !

  하지만 1차 캐시가 초기화되었다면 초기화 전의 객체와 초기화 후의 객체가 다르다 !

    - 재정의 방식
        - PK로 구분 ! (당연한 방식)
        - **@NaturalId 사용 → Spring -data-jpa에서는 사용을 자제**

[How to Use Hibernate Natural IDs in Spring Boot - DZone](https://dzone.com/articles/how-to-use-hibernate-natural-ids-in-spring-boot)

### 식별자

**@Id**

- PK로 사용할 필드 위에 붙임
- 모든 원시 타입 , Wrapper 타입 String, java.util.Date, java.sql.Date, BigDemical, BigInteger에 사용 가능

**@GeneratedValue**

- 기본키(PK) 값에 대한 **생성 전략 제공**
    - @Id 필드에 지정하여 사용
1. AUTO(default)
    - JPA 구현체가 자동으로 생성 전략을 결정
    - hibernate.dialect에 설정된 DB 종류에 따라 자동으로 선택
2. IDENTITY

```
@Id @GeneratedValue(strategy = GenerationType.IDENTITY) 
private Long id;
```

- 기본 키 생성을 **데이터베이스에 위임**한다.
- em.persist()로 객체를 영속화 시키는 시점에 insert 쿼리가 DB로 전송
    - 보통의 경우 트랜잭션이 commit되는 시점에 SQL 한번에 전송 , 하지만 PK 생성을 DB에 위임했기 때문에 미리 쿼리를 보내 영속성 컨텍스트에서 PK를 통해 관리
1. SEQUENCE
    - sequence generator는 @SequenceGenerator를 사용해 정의하고, 이 어노테이션은 name, sequenceName, allocationSize등의 속성을 가진다.
    - **DB의 시퀀스를 활용하여 ID값을 증가시킨다.**
        - 시퀀스란 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트

> SEQUENCE 전략은 em.persist() 호출 전에 **먼저 DB Sequence를 먼저 조회**한다.
>
>
> 그 후 **조회한 식별자를 Entity에 할당한 후 Entity를 영속상태로 저장**한다.
>
> 그 후 Transaction을 Commit하여 Flush가 발생할 때 해당 Entity를 DB에 저장한다.
>
- **allocationSize 지정 !**

  시퀀스에 접근하는 횟수를 줄이기 위해 한번 접근 후 해당 값만큼 증가 → 범위 안의 값은 메모리에서 할당

  그 과정에서 시퀀스 값을 선점 → JVM이 동시에 동작해도 기본 키 값 충돌 X


```java
@SequenceGenerator(
		name = "sequence-generator",
		sequenceName = "explicit_product_sequence",
		allocationSize = 5
	)
public class Member{

		@Id
		@GeneratedValue(startegy = GenerationType.SEQUENCE,
		generator = "sequence-generator")
		private Long id;
}
```

1. TABLE 전략
- 키 생성 전용 테이블을 하나 만들어 데이터베이스 시퀀스를 흉내내는 것 !
- **IDENTITY 와 SEQUENCE 주의 !**

  분산 DB 환경일 경우에, PK값이 중복되어 INSERT될 가능성이 있다.


**복합키**

- @EmbeddedId
    - @Embeddable 을 붙여서 구현한 클래스를 @EmbeddedId와 함께 필드로 사용하면 된다.
- @IdClass

### @IdClass 예시

```java
@IdClass(UserId.class)
@Entity
public class User {

    @Id
    @Column(name = "USER_ID")
    private String id;

    @Id
    @Column(name = "REG_DATE")
    private LocalDateTime regDate;

    // 생략

}
```

```java
@AllArgsConstructor
@NoArgsConstructor
public class UserId implements Serializable {

    private String id;
    private LocalDateTime regDate;

    // equals & hashCode 부분 생략
}
```

- id로 사용할 클래스 요구사항
    - **public** 클래스일 것
    - 기본 생성자 필수
    - 엔티티 클래스에서 작성한 필드 명과 동일하게 작성할 것 (컬럼명이 아님에 주의)
    - **Serializable**을 구현해야 함
    - **equals**와 **hashCode**를 구현해야 함

### 상속

- 상속은 기본적으로 **객체 지향 프로그래밍에 존재하는 개념**이다.
    - 데이터베이스의 테이블을 상속이 적용된 클래스들에게 적용시키는 방법이 존재

### @MappedSuperclass

- 객체 입장에서 **상위 클래스를 명시**할 때 사용
    - 해당 정보는 기본적으로 테이블에는 반영되지 않는다.
    - 대표적인 예시로 BaseEntity가 있다.

![Untitled](https://user-images.githubusercontent.com/84346055/276440190-2e00a994-85e3-4a49-922a-1399b80832ac.png)

### 단일 테이블 전략

- @Inheritance(strategy = InheritanceType.SINGLE_TABLE)

![https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/23_super_sub.PNG?raw=true](https://github.com/namjunemy/TIL/blob/master/Jpa/inflearn/img/23_super_sub.PNG?raw=true)

- 하나의 테이블에 모든 필드(상속 받은 클래스)를 저장 → **DTYPE으로 구분 !**
- insert , select 쿼리가 전부 1번이다. ( join을 사용하지 않는다.)

```
where
      movie0_.id=?
       and movie0_.DTYPE='Movie'
```

### 조인 테이블 전략

- @Inheritance(strategy = InheritanceType.JOINED)
    - 하위 테이블들이 상위 테이블의 id를 가지는 방식 !

![Untitled](https://user-images.githubusercontent.com/84346055/276440192-4984158a-db86-4f20-bf1e-1135b23f6a97.png)

```

Hibernate:
   alter table Album
      add constraint FKcve1ph6vw9ihye8rbk26h5jm9
      foreign key (id)
      references Item
```

- insert 쿼리가 2번 실행 → 부모 클래스 , 자식 클래스
    - **부모 클래스의 PK 가 자식 클래스의 PK이자 FK이다.**
- inner join을 통해서 결과를 조회한다.

```
Hibernate:
  select
      movie0_.id as id2_2_0_,
      movie0_1_.name as name3_2_0_,
      movie0_1_.price as price4_2_0_,
      movie0_.actor as actor1_3_0_,
      movie0_.director as director2_3_0_
  from
      Movie movie0_
  inner join
      Item movie0_1_
          on movie0_.id=movie0_1_.id
  where
      movie0_.id=?
```

### 구현클래스 별 테이블 전략

- @Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
- 각 하위 클래스들이 상위 클래스의 필드를 전부 가지고 있다.

![Untitled](https://user-images.githubusercontent.com/84346055/276440194-54779229-37af-4877-8181-a61b1e4bf193.png)

## Mutability

- 불변 엔티티 → **@Immutable** 을 클래스 레벨에 붙인다.
- 가질 수 있는 이점
    - 더티 체킹을 위해 로드된 상태로 유지시킬 메모리 감소
    - 영속성 컨텍스트 플러시 단계의 속도 향상

불변 속성 → `@Immutable` 을 필드 레벨에 붙인다.

불변 컬렉션 → `@Immutable` 을 컬렉션 필드에 붙인다.

### 참고

[Hibernate Architecture](https://dejavuhyo.github.io/posts/hibernate-architecture/)

[TreeWiki](https://chanwookpark.github.io/jpa/hibernate/번역/2016/09/26/hibernate-jpa-best-practices/)
