# 1.1 SQL을 직접 다룰 때 발생하는 문제점

- 데이터베이스의 데이터를 관리하려면 SQL을 사용해야 한다.
- 자바로 작성한 애플리케이션은 JDBC API를 사용해서 SQL을 데이터베이스에 전달한다.

<hr/>

## 1.1.1 반복, 반복 그리고 반복
- 회원 객체 `Member`를 만든다.
- 회원 객체를 데이터베이스에서 관리할 목적으로 회원용 DAO(데이터 접근 객체)인 `MemberDAO`를 만든다.
   
◼ `MemberDAO`의 `find()` 메서드 개발(회원 조회 기능)
1. 회원 조회용 SQL을 작성한다.
   > ```
   > SELECT MEMBER_ID, NAME FROM MEMBER M WHERE MEMBER_ID = ?
   > ```
2. JDBC API를 사용해서 SQL을 실행한다.
   > ```
   > ResultSet rs = stmt.executeQuery(sql);
   > ```
3. 조회 결과를 `Member` 객체로 매핑한다.
   > ```
    > String memberId = rs.getString("MEMBER_ID");
    > String name = rs.getString("NAME");
    >
    > Member member = new Member();
    > member.setMemberId(memberId);
    > member.setName(name);
    > ```
   
◼ 회원 등록 기능
1. 회원 등록용 SQL을 작성한다.
   > ```
   > String sql = "INSERT INTO MEMBER(MEMBER_ID, NAME) VALUES(?, ?)";
   > ```
2. 회원 객체의 값을 꺼내서 등록 SQL에 전달한다.
   > ```
   > pstmt.setString(1, member.getMemberId());
   > pstmt.setString(2, member.getName());
   > ```
3. JDBC API를 사용해서 SQL을 실행한다.
   > ```
   > pstmt.executeUpdate(sql);
   > ```   
   
- 다음으로 회원을 수정하고 삭제하는 기능을 추가하려면 이와 같이 SQL을 작성하고 JDBC API를 사용하는 비슷한 일을 반복해야 한다.

<hr/>

## 1.1.2 SQL에 의존적인 개발

- 개발 완료 후, 필드 추가 요청이 생길 경우 기존에 작성한 SQL 코드를 대대적으로 수정해아 한다.
- SQL을 따라 가며 일일이 수정하는 경우 언제 어디서 버그가 발생할지 모른다.
- 이런 방식의 가장 큰 문제는 데이터 접근 계층을 사용해서 SQL을 숨겨도 어쩔 수 없이 DAO를 열어 어떤 SQL이 실행되는지 확인해야 한다는 점이다.
- SQL에 모든 것을 의존하는 상황에서는 엔티티를 신뢰하고 사용할 수 없다.
- 즉, 완전한 의미의 계층 분할이 아니다.

✔ 애플리케이션에서 SQL을 직접 다룰 때 발생하는 문제점 요약
> - 진정한 의미의 계층 분할이 어렵다.
> - 엔티티를 신뢰할 수 없다.
> - SQL에 의존적인 개발을 피하기 어렵다.

<hr/>

## 1.1.3 JPA와 문제 해결

◼ JPA가 제공하는 CRUD API
- 저장 기능
> ```
> jpa.persist(member);
> ```

<br/>

- 조회 기능
> ```
> String memberId = "helloId";
> Member member = jpa.find(Member.class, memberId);
> ```

<br/>

- 수정 기능
> ```
> Member member = jpa.find(Member.class, memberId);
> member.setName("이름변경");
> ```

<br/>

- 연관된 객체 조회
> ```
> Member member = jpa.find(Member.class, memberId);
> Team team = member.getTeam();
> ```

<hr/>

# 1.2 패러다임의 불일치

- 애플리케이션은 발전하면서 내부 복잡성도 점점 커진다.
- 비즈니스 요구사항을 정의한 도메인 모델을 객체로 모델링하면 객체지향 언어가 가진 장점들을 활용할 수 있다.
- 객체를 영구적으로 보관하고자 할 때, 관계형 데이터베이스에 저장하게 된다.
>   - 관계형 데이터베이스는 데이터 중심 구조화되어 있고, 집합적인 사고를 요구한다.
>   - 추상화, 상속, 다형성 같은 개념이 없다.
> 객체와 관계형 데이터베이스는 지향하는 목적이 서로 다르므로 둘의 기능과 표현 방법도 다르다. (패러다임의 불일치 문제)

<hr/>

## 1.2.1 상속
- 객체는 상속이라는 기능을 가지고 있지만 테이블은 상속이라는 기능이 없다
- JPA는 상속과 관련된 패러다임의 불일치 문제를 개발자 대신 해결해준다.
- 개발자는 자바 컬렉션에 객체를 저장하듯 JAP에 객체를 저장하면 된다.

<hr/>

## 1.2.2 연관관계
- 객체는 참조를 사용해서 다른 객체와 연관관계를 가지고 참조에 접근해서 연관된 객체를 조회한다.
- 테이블은 외래 키를 사용해서 다른 테이블과 연관관계를 가지고 조인을 사용해서 연관된 테이블을 조회한다.
- 객체는 참조가 있는 방향으로만 조회할 수 있다.
- 테이블은 외래 키 하나로 양방향 조인이 가능하다.

```java
class Member {

   String id;     // MEMBER_ID 컬럼 사용
   Long teamId;   // TEAM_ID FK 컬럼 사용
   String username; // USERNAME 컬럼 사용
}

class Team {

   Long id;   // TEAM_ID PK 사용
   String name; // NAME 컬럼 사용
}
```
- 관계형 데이터베이스는 조인이라는 기능이 있으므로 외래 키의 값을 그대로 보관해도 된다.
- 하지만 객체는 연관된 객체의 참조를 보관해야 다음처럼 참조를 통해 연관된 객체를 찾을 수 있다.
> ```java
> Team team = member.getTeam();
> ```
- `TEAM_ID` 외래 키까지 관계형 데이터베이스가 사용하는 방식에 맞추면 `Member` 객체와 연관된 `Team` 객체를 참조를 통해서 조회할 수 없다.
- 좋은 객체 모델링을 기대하기 어렵고 객체지향의 특징을 잃게 된다.

<br/>

◼ 객체지향 모델링
```java
class Member {

   String id;     // MEMBER_ID 컬럼 사용
   Team team;   // 참조로 연관관계를 맺는다.
   String username; // USERNAME 컬럼 사용

   Team getTeam() {
      return team;
   }
}

class Team {

   Long id;   // TEAM_ID PK 사용
   String name; // NAME 컬럼 사용
}
```
- `Member.team` 필드는 외래 키의 값을 그대로 보관하는 것이 아니라 연관된 `Team`의 참조를 보관한다.
- 이처럼 객체지향 모델링을 사용하면 객체를 테이블에 저장하거나 조회하기가 쉽지 않다.

| |객체 모델|테이블|
|:---:|:---:|:---:|
| 외래키 |X| O|
|참조|O|X|

- 이 둘 사이에 발생하는 괴리 때문에 개발자가 중간에서 변환 역할을 해야 한다.
- `객체 -> 데이터베이스 저장`: `team` 필드 -> `TEAM_ID` 외래키 값으로 변환
- `조회`: `TEAM_ID` 외래키 -> `Member` 객체의 `team` 참조로 변환해서 객체에 보관
- => 패러다임 불일치를 해결하는데 소모되는 비용
- 이런 문제를 해결해주는게 JPA

<hr/>

## 1.2.3 객체 그래프 탐색
- 객체를 조회할 때는 참조를 사용해서 연관된 객체를 찾으면 되는데, 이를 **객체 그래프 탐색**이라고 한다.
- 객체는 마음껏 객체 그래프를 탐색할 수 있어야 하는데, SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다.
- 특정 로직만 보고는 탐색이 가능한 방향인지 알 수 없다.
- 결국, 어디까지 객체 그래프 탐색이 가능한지 알아보려면 데이터 접근 계층인 `DAO`를 열어서 SQL을 직접 확인해야 한다.
- => 엔티티가 SQL에 논리적으로 종속되어서 발생하는 문제
- 그렇다고 연관된 모든 객체 그래프를 데이터베이스에서 조회해서 애플리케이션 메모리에 올려둘 수는 없다.
- 결국 상황에 따라 조회하는 메서드를 여러 벌 만들어서 사용해야 한다.
> ```
> memberDAO.getMember();
> memberDAO.getMemberWithTeam();
> memberDAO.getMemberWithOrderWithDelivery();
> ```
