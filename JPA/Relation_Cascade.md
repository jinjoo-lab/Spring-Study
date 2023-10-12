# 연관관계 + 영속성 전이

### 연관관계

- **불일치의 문제**

  RDB에서 테이블간의 연관관계는 기본적으로 외래키를 사용한다 ! 객체 지향 프로그래밍에서 객체간의 연관관계는 참조를 바탕으로 한다.

- 테이블은 외래키를 통한 조인 연산을 통해 양방향 탐색이 가능하다. 하지만 **엔티티들은 기본적으로 단방향 탐색**만 가능하다.
- 엔티티간의 양방향 탐색은 근본적으로 **단방향 연결이 2개인 것 !**

### 연관관계의 종류

- @OneToOne
- @OneToMany
- @ManyToOne
- @ManyToMany

## 핵심 개념

### 주인

- 주인 엔티티란 양방향 연관관계를 설정 시 실제 데이터베이스에서 **외래키를 가지고 있는 테이블**을 의미한다.
    - 주인 엔티티에 변경 사항을 반영해야지만 실제 DB에 반영된다.
- 설정 방법
    - **mappedBy** 설정을 통해 주인 설정 → 주인인 쪽에 사용하지 않고 반대쪽에 설정한다.
- **연관관계의 주인만이 데이터 베이스의 연관관계와 매핑되고 외래 키를 사용할 수 있다.**
- 주인이 아닌 쪽은 ReadOnly만 가능 !

### @JoinColumn

- 외래키를 매핑할 때 사용
    - name
        - 매핑할 외래 키 칼럼명 (**자기 테이블의 외래 키 필드명**)
    - referencedColumnName
        - **외래 키가 참조하는 대상 테이블**의 칼럼명
        - default값이 참조하는 테이블의 PK명으로 조인된다.

### 양방향 연관관계

> 객체 간의 양방향 연관관계는 사실 존재하지 않는 개념이다. 서로 다른 2개의 단방향 연관관계를 애플리케이션 로직으로 묶는 것 뿐 !
>
- 엔티티간의 양방향 관계는 **기본적으로 단방향 관계 2개 (2개의 참조)를 설정**해줘야 한다.
    - 양방향 연관관계에 있어서는 주인 엔티티를 설정해줘야 한다.
- @JoinColumn
    - 양방향 연관관계에서 @JoinColumn을 통한 설정을 하지 않을 경우 **중간 테이블이 생성**된다.

### @OneToOne

- 1:1 연관관계를 표현하는데 사용

### @OneToMany

- 1:N 연관관계를 표현하는데 사용
- 절대 주인이 될 수 없다 !
    - 1인 테이블에서 외래키를 관리할 수 없는 구조이기 때문에 연관관계에서 주인일 수 없다.
    - 즉 mappedBy 속성을 이용해야 한다.

### @ManyToOne

- N:1 연관관계를 표현하는데 사용
- 항상 주인 엔티티 (외래키를 관리한다.)
    - mappedBy 속성이 지원되지 않는다.

### @ManyToMany

- N:M 연관관계를 표현하는데 사용
    - 잘 사용하지는 않는다. (중간 테이블에 다른 필드 값을 추가할 수 없기 때문이다.)
    - 보통 N:M 연관관계를 **1:N ↔ M : 1 인 2개의 관계로 쪼개서 사용**

## 영속성 전이

- 하나의 엔티티를 저장하거나 변경하였을 때 연관관계가 설정된 다른 엔티티에도 영속성 컨텍스트의 작업이 일어나도록 하는 것
- **전이 반응**

  기본적으로 RDB의 **참조 무결성 제약조건**을 지키기 위해 사용

- **삭제 시**
    - 참조되는 테이블의 데이터 삭제 시 참조하는 테이블에서 참조 무결성 제약조건을 어기는 상황 발생 가능
- **참조 무결성 제약 조건**

  외래 키의 값은 NULL이거나 참조되는 테이블의 기본 키와 동일해야 한다.


### 영속성 전이의 종류

```java
public enum CascadeType{
		ALL, // 모두적용
    PERSIST, // 영속
    MERGE, // 병합
    REMOVE, // 삭제
    REFRESH, // REFRESH
    DETACH // DETACH
}
```

### code

**Member**

```java
@Entity
public class Member {
    @ManyToOne
    @JoinColumn(name = "team_id")
    Team team;
}
```

**Team**

```java
@Entity
public class Team {
    @OneToMany(mappedBy = "team",cascade = CascadeType.PERSIST)
    List<Member> members = new LinkedList<>();
}
```

**Test**

```
@Test
@Transactional
public void test()
{
Team team = new Team();

		Member member = new Member();
		member.setName("Jinjooone");
		member.setAge(21);

		Member member2 = new Member();
		member2.setName("Jinjooone2");
		member2.setAge(21);

		Member member3 = new Member();
		member2.setName("Jinjooone3");
		member2.setAge(21);

		team.getMemberList().add(member);
		team.getMemberList().add(member2);
		team.getMemberList().add(member3);

		teamRepository.save(team); // 부모만 영속화 -> 자식도 영속화
 }
```

- **위의 코드에서 잘못된 점?**

  ![Untitled](https://user-images.githubusercontent.com/84346055/274512041-9028cb6b-f240-4b66-a4de-001bfa006efa.png)

  영속성 전이에 의해 Member도 저장되는 것을 알 수 있다. 하지만 각 member에 team_id는 저장되지 않는다.

    - **영속성 전이는 연관관계 매핑과 아무련 관련이 없는 것을 알 수 있다 !**
- 연관관계 매핑은 외래키를 가지고 있는 엔티티에서 관계를 설정해줘야만 한다. 하지만 영속성 전이는 위에서 일대다의 경우에 일에 해당하는 엔티티에서 전이를 일으키기 때문에 동작하지 않는다.

**영속성 전이 2개 이상 적용 시**

```
cascade = {CascadeType.PERSIST, CascadeType.REMOVE}
```

- **적용 시점**
    - PERSIST , REMOVE는 실행 시 바로 전이가 일어나는 것이 아닌 flush 호출 시점에 전이가 발생한다.

**문제**

- @ManyToOne에서 Cascade 사용 시

**Member**

```java
@Entity
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    public long id;

    public String name;

    public int age;
    
    @ManyToOne(cascade = {CascadeType.PERSIST , CascadeType.MERGE})
    @JoinColumn(name = "team_id")
    public Team team;
}
```

**Test Code**

```
	@Test
	void contextLoads() {
		Team team = new Team();

		Member member = new Member();
		member.setName("Jinjooone");
		member.setAge(21);

		Member member2 = new Member();
		member2.setName("Jinjooone2");
		member2.setAge(21);

		member.setTeam(team);
		member2.setTeam(team);

		memberRepository.save(member);
		memberRepository.save(member2); // member2 save 과정에서 Error 발생
	}
```

- 위의 코드에서 member2가 저장되지 못하는 이유는?

  우선 트랜잭션의 범위가 각 메소드안에서만 유효하다.

    1. 첫 번째 save 메소드 호출 시 → member 저장 , team 저장
        1. 중요한 점은 team이 저장되고 나서 **준영속(detach) 상태**가 된다는 것이다 !
    2. 두 번째 save 메소드가 호출할 때 영속성 전이는 CascadeType.PERSIST이다.
        1. 왜냐하면 **member2는 처음 저장되는 것이니까 !!!!**
        2. 하지만 team은 준영속 상태이기 때문에 PERSIST가 아닌 MERGE가 발동되야만 한다 !
- **해결 방법**

  @Transactional 사용

    - 영속성 컨텍스트의 범위를 메소드 단위로 확장하여 team 객체의 정보를 유지하는 것이다 !

### 고아 객체

- orphanRemoval = true
    - **ManyToOne에서는 지원하지 않는다 !**
- 부모와 **연관관계가 끊어진** 엔티티 → 고아 객체

### CascadeType.REMOVE  Vs OrphanRemoval = true

**cascade**

- 부모 엔티티가 삭제되면 자식 엔티티도 삭제된다.
    - 즉 부모가 자식의 생명주기 관리
- 부모 엔티티와 자식 엔티티의 관계를 제거해도 자식 엔티티는 삭제되지 않는다.

**orhanRemoval**

- 부모 엔티티가 삭제되면 자식 엔티티도 삭제된다. → Cascade.PERSIST와 함께 사용시
- 부모엔티티와 자식 엔티티의 관계 제거시 자식 엔티티는 고아 객체가 되어 삭제

- **관계 제거 ?**

  **부모 객체에서 자식 객체의 정보를 제거하는 것 !**

    ```
    teamRepository.getById(1l).getMembers().remove(0); // 고아 객체 제거
    ```


- @SQLDelete 어노테이션을 모든 엔티티에 사용하는 경우 혹은 SoftDelete를 직접 구현하는 경우, Cascade.REMOVE,  orphanRemoval=true를 통한 삭제가 가능한가?
    - 상위 엔티티(부모) , 하위 엔티티(자식) 모두 구현할 시 가능하다. 하위 엔티티만 설정을 해놓은 경우 상위 엔티티가 Hard Delete로 작용하여 에러 발생
    - **엥 되던데 ?**

      상위 엔티티가 삭제된다면 하위 엔티티의 **외래키는 null**로 변한다.

      ![Untitled](https://user-images.githubusercontent.com/84346055/274512061-9ef452f8-084b-4988-9e42-0000cfb0b4b6.png)


    **Test  1 (cascade)**
    
    ```java
    부모 : @SQLDelete(sql = "UPDATE toy.team SET name= 'xxxx' WHERE id = ?")
    자식 : @SQLDelete(sql = "UPDATE toy.member SET deleted = true WHERE id = ?")
    // 양 쪽 다 구현한 경우
    ```
    
    ```java
    public void test()
    {
        Team t = new Team();
        Member m1 = new Member();
        Member m2 = new Member();
    
        t.getMembers().add(m1);
        t.getMembers().add(m2);
    
        teamRepository.save(t);
    		teamRepository.deleteById(1l); // 영속성 전이 (REMOVE) test
    }
    ```
    
    **Result**
    
    ```java
    Hibernate: 
        UPDATE
            toy.member 
        SET
            deleted = true 
        WHERE
            id = ?
    Hibernate: 
        UPDATE
            toy.member 
        SET
            deleted = true 
        WHERE
            id = ?
    Hibernate: 
        UPDATE
            toy.team 
        SET
            name= 'xxxx' 
        WHERE
            id = ?
    ```
    
    [https://unluckyjung.github.io/jpa/2022/11/29/JPA-cascade-orphanRemoval/](https://unluckyjung.github.io/jpa/2022/11/29/JPA-cascade-orphanRemoval/)
    
    **Test 2 (orphanRemoval)**
    
    ```java
    teamRepository.getById(1l).getMembers().remove(0);
    ```
    
    **Result**
    
    ```java
    Hibernate: 
        UPDATE
            toy.member 
        SET
            deleted = true 
        WHERE
            id = ?
    ```
    
    - 고아 객체의 제거도 적용되는 것을 알 수 있다.
    
    **SoftDelete를 직접 구현하는 경우**
    
    - Repository 가 아닌 해당 함수를 통해 삭제하는경우에는 `cascade.REMOVE` 옵션이 작동하지 않아, 부모 객체(Team) 만 삭제되게 됩니다.
    
    [JPA CascadeType.REMOVE vs orphanRemoval = true](https://tecoble.techcourse.co.kr/post/2021-08-15-jpa-cascadetype-remove-vs-orphanremoval-true/)
