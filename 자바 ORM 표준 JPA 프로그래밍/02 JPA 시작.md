# 2.4 객체 매핑 시작

- `@Entity`
  - 이 클래스를 테이블과 매핑한다고 JPA에게 알려준다.
- `@Table`
  - 엔티티 클래스에 매핑할 테이블 정보를 알려준다.
  - 이 어노테이션을 생략하면 클래스 이름을 테이블 이름으로 매핑한다(더 정확히는 엔티티 이름을 사용한다)
- `@Id`
  - 엔티티 클래스의 필드를 테이블의 기본 키에 매핑한다.
  - @Id가 사용된 필드를 **식별자 필드**라고 한다.
- `@Column`
  - 필드를 컬럼에 매핑한다.
- 매핑 정보가 없는 필드
  - 매핑 어노테이션을 생략하면 필드명을 사용해서 컬럼명으로 매핑한다.
  - 대소문자를 구분하는 데이터베이스를 사용하면 `@Column(name="AGE")`처럼 명시적으로 매핑해야 한다.
 
<hr/>

# 2.5 persistence.xml 설정

- JPA는 `persistence.xml`을 사용해서 필요한 설정 정보를 관리한다.
- 이 파일이 `META-INF/persistence.xml` 클래스 패스 경로에 있으면 별도의 설정 없이 JPA가 인식할 수 있다.

◼ JPA 표준 속성
- `javax.persistence.jdbc.driver`: JDBC 드라이버
- `javax.persistence.jdbc.user`: 데이터베이스 접속 아이디
- `javax.persistence.jdbc.password`: 데이터베이스 접속 비밀번호
- `javax.persistence.jdbc.url`: 데이터베이스 접속 URL

◼ 하이버네이트 속성
- `hibernate.dialect`: 데이터베이스 방언 설정

<br/>

- 이름이 `javax.persistence`로 시작하는 속성은 JPA 표준 속성으로 특정 구현체에 종속되지 않는다.
- `hibernate`로 시작하는 속성은 하이버네이트 전용 속성이므로 하이버네이트에서만 사용할 수 있다.

## 2.5.1 데이터베이스 방언

- 각 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다르다.
  - 데이터 타입
  - 다른 함수명
  - 페이징 처리
- SQL 표준을 지키지 않거나 특정 데이터베이스만의 고유한 기능을 JPA에서는 방언이라 한다.
- 특정 데이터베이스에 의존적인 SQL은 데이터베이스 방언이 처리해준다.

<hr/>

# 2.6 애플리케이션 개발

## 2.6.1 엔티티 매니저 설정
- 엔티티 매니저 팩토리 생성
  - > EntityManagerFactory emf = Persistence.createEntityManagerFactory("jpabook");
  - `META-INF/persistence.xml`에서 이름이 `jpabook`인 영속성 유닛을 찾아서 엔티티 매니저 팩토리를 생성한다.
  - `persistence.xml`의 설정 정보를 읽어서 JPA를 동작시키기 위한 기반 객체를 만들고 JPA 구현체에 따라서는 데이터베이스 커넥션 풀도 생성하므로 엔티티 매니저 팩토리를 생성하는 비용은 아주 크다.
  - 엔티티 매니저 팩토리는 애플리케이션 전체에서 딱 한 번만 생성하고 공유해서 사용해야 한다.
- 앤티티 매니저 생성
  - > EntityManager em = emf.createEntityManager();
  - JPA의 기능 대부분은 엔티티 매니저가 제공한다.
  - 엔티티 매니저는 내부에 데이터베이스 커넥션을 유지하면서 데이터베이스와 통신한다.
  - 엔티티 매니저는 데이터베이스 커넥션과 밀접한 관계가 있으므로 스레드간에 공유하거나 재사용하면 안 된다.
- 종료
  - 사용이 끝난 엔티티 매니저와 엔티티 매니저 팩토리는 반드시 종료해야 한다.
  - > em.close();<br/>emf.close();

<hr/>

## 2.6.2 트랜잭션 관리
- JPA를 사용하면 **항상 트랜잭션 안에서 데이터를 변경**해야 한다.
```java
EntityTransaction tx = em.getTransaction();
try {
  tx.begin();
  logic(em);
  tx.commit();
} catch (Exception e) {
  tx.rollback();
}
```

<hr/>

## 2.6.3 비즈니스 로직

- 등록
  - > String id = "id1";<br/>Member member = new Member();<br/>member.setId(id);<br/>member.setUsername("지한");<br/>member.setAge(2);<br/><br/>em.persist(member);

- 수정
  - > member.setAge(20);
  - JPA는 어떤 엔티티가 변경되었는지 추적하는 기능을 갖추고 있다.
  - 위처럼 엔티티의 값을 변경하면 UPDATE SQL을 생성해서 데이터베이스에 값을 변경한다.
 
- 삭제
  - > em.remove(member);
- 한 건 조회
  - > Member findMember = em.find(Member.class, id);

<hr/>

## 2.6.4 JPQL
- 테이블이 아닌 엔티티 객체를 대상으로 검색하려면
  - 데이터베이스의 모든 데이터를 애플리케이션으로 불러와서
  - 엔티티 객체로 변경한 다음
  - 검색해야 한다.
- 이는 사실상 불가능하다.
- 애플리케이션이 필요한 데이터만 데이터베이스에서 불러오려면 결국 검색 조건이 포함된 SQL을 사용해야 한다.
- JPA는 JPQL(Java Persistence Query Language)이라는 쿼리 언어로 이 문제를 해결한다.
- JPQL은 엔티티 객체를 대상으로 쿼리한다.
- SQL은 데이터베이스 테이블을 대상으로 쿼리한다.
