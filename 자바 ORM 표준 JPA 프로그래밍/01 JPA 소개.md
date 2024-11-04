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
