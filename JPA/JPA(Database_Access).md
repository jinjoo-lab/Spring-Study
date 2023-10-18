# JPA -1

## 데이터베이스 접근 기술

### 3-tier Architecture

![Untitled](https://user-images.githubusercontent.com/84346055/276047784-fb63e2d3-480e-452a-a55f-40c802ac09ae.png)

- Presentaition Layer ( 사용자 인터페이스 )
- Application (Bussibess) Layer ( 데이터 처리 )
    - 데이터 접근 기술을 Application Layer 와 Data Layer 사이의 연결을 통해 이루어진다.
- Data Layer ( 데이터 저장, 관리 ) → **Persistence Layer**

### 영속성

- 데이터를 생성한 프로그램이 종료되더라도 데이터가 사라지지 않는 속성
- 영속성을 유지하는 방법 → **DB에 데이터를 반영하고 관리**

### 데이터 접근 기술

- **프로그래밍** 기술과 **데이터 저장** 기술 사이 연결을 통해 유기적인 동작이 가능하도록 하는 기술
    - Ex) **객체 지향 ↔ RDB 사이 연결 기술**

![Untitled](https://user-images.githubusercontent.com/84346055/276047799-c99912fa-4364-41a8-9db6-dc048a6a64dc.png)

- **동작 방식**
    1. 데이터베이스와 자바 프로그램 사이 연결 (**Make Connection**)
    2. **SQL 전달 (데이터 처리)**
    3. 연결 종료 (**Quit Connection**)
- **첫 번째 문제 : DB가 변경된다면 ?**

  DB마다 연결 방식 , 접근 방식이 다르다. 서비스 중 다른 DB를 추가하거나 변경한다면 **데이터 접근 기술에 이를 반영**해줘야 한다.

- **해결 방법**
    - **데이터 접근 기술을 추상화된 인터페이스로 분리**하여 일관된 접근이 가능하도록 하자 !

### JDBC

- Java Database Connectivity (자바와 데이터베이스 사이 **연결 프로그래밍 인터페이스**)
    - JDBC API : DBMS에 종속적이지 않다 ! → DB 변경 , 확장 시 **드라이버만 변경**

![Untitled](https://user-images.githubusercontent.com/84346055/276047802-fd015fe3-8bc5-46b8-92ac-844c3c13c71b.png)

- 동작 방식 !
    - JDBC 드라이버 Load
    - Connection 객체 생성
    - Statement 객체 생성 → SQL를 담는다.
    - Query 전달 → 데이터 처리
    - Connection 객체 Close

**예제 코드**

```
// JDBC 드라이버 Load -> 아래 방식에서는 Mysql 드라이버 연결
Class.forName("com.mysql.Jdbc.Driver");

// DB와의 Connection 연결
String jdbc_url = "jdbc:mysql://localhost:3306/datebase?serverTimezone=UTC";
Connection con = DriverManager.getConnection(URL, "user", "password");

// Statement 생성 
Statement stmt = con.createStatement();

// SQL를 Statement에 담아 질의 수행 -> 결과값 ResultSet
String sql = "select * from student";
ResultSet result = stmt.executeQuery(sql);

// Connection 종료
rs.close();
st.close();
con.close();
```

- **두 번째 문제 :**
    1. 데이터베이스에 데이터 처리 시 위의 **작업이 반복**된다. → Connection 관리
    2. **SQL과 자바 코드의 공존**
        1. ***나는 데이터 처리 작업을 원해 ! 허나 위의 코드에는 불필요한 작업(연결)등이 포함되어 있어 !***
    3. SQL를 **단순한 문자열**로 처리
        1. 만약 SQL이 잘못되었다면 ? → 오류를 찾기 쉽지 않다.
    4. 결과값을 **ResultSet으로 관리**된다. → 불필요한 반복적 데이터 바인딩
        1. 매번 **클래스의 객체에 데이터를 바인딩하는 작업**이 필요하다.

### SQL Mapper

- 자바 Persistence Framework
- SQL문과 **객체 필드를 매핑하여 데이터를 객체화 !**
    - 가장 대표적인 구현체 → **MYBATIS**

### **MYBATIS**

1. MYBATIS에서는 **Connection** 관리와 statement 세팅 작업 **자동**으로 처리
2. 자바 코드와 SQL의 분리
    1. **SQL**를 **XML로 관리**하여 자바 코드내 불필요한 SQL문 제거
3. 데이터 바인딩을 통해 **데이터를 객체화**
    - Ex ) **RDB(TEST_TABLE)을 TestDTo로 변환**

```
<select id="findAll" resultType="TestDto">
        SELECT
               MC_ID AS id
             , P_ID AS promotionId
             , ITEM_ID AS itemId
             , MC_FG AS type
             , YH_E_YMD AS expiredEndDate
             , YH_S_YMD AS expiredStartDate
             , USE_YN AS useYn
        FROM
            TEST_TABLE
        WHERE
            MC_ID IN ('5716', '5717')
</select>
```

- **근본적 문제**
    - **객체지향 패러다임 불일치 !**

> RDB에서는 데이터를 **테이블 단위**로 관리한다. Java에서는 데이터를 **클래스 단위**로 관리한다. 결국 데이터 접근 기술 자체는 Java 즉 객체 지향 프로그래밍 기반이다. 하지만 SQL를 반영하는 작업을 위해 **데이터 중심적 설계를 하여 객체 지향적 설계가 깨지고 있다.**
>

### ORM (Object Relational Mapping)

- 객체 - 관계 매핑
    - 객체와 DB의 테이블을 자동으로 연결(매핑)
        - **RDB의 테이블을 객체지향적으로 사용**하게 해주는 기술 !

> MYBATIS는 **SQL구문과 객체를 매핑**했다면 ORM은 **RDB 테이블 자체와 객체를 매핑**
>
- ***테이블 자체를 객체로 매핑했기 때문에 SQL 사용 없이 데이터에 접근 , 관리할 수 있다 !!!!***

### JPA

- 자바 ORM 기술에 대한 API 표준 명세, Java에서 제공하는 API
- JPA에 대표적인 구현체 → **Hibernate**
- **JPA에서 JDBC 사용?**

  ![Untitled](https://user-images.githubusercontent.com/84346055/276047806-9136389f-ef4b-4f08-95c8-b99f2bb34688.png)

  JPA 내부적으로 데이터베이스와 통신하기 위해서는 JDBC를 사용한다. 하지만 그 접근 자체가 사용자에게 보여지지는 않는다 !


### 참고

[[10분 테코톡] 범고래, 소주캉의 JDBC, SQL Mapper, ORM](https://www.youtube.com/watch?v=NFK9qLWpujY)
