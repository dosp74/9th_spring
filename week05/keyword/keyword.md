# WEEK 5 - 제이/한종서

## 핵심 키워드

### 지연 로딩(LAZY)과 즉시 로딩(EAGER)의 차이

JPA에서 **지연 로딩(LAZY)** 과 **즉시 로딩(EAGER)** 은 연관관계를 가진 엔티티를 언제 조회할 것인가를 결정하는 전략이다.

| 구분 | 지연 로딩(LAZY) | 즉시 로딩(EAGER) |
|------|-----------------|-----------------|
| 로딩 시점 | 실제로 해당 엔티티를 사용할 때 조회 | 엔티티를 조회하는 시점에 함께 조회 |
| SQL 실행 방식 | 1차 쿼리 + 필요한 시점에 추가 쿼리 | JOIN으로 한 번에 가져오는 경우가 많음 |
| 장점 | 불필요한 데이터 로딩 방지 → 성능 최적화 | 즉시 접근 가능 (직접 접근 시 추가 쿼리 없음) |
| 단점 | 처음 접근 시 추가 쿼리 발생 | 필요 없는 데이터까지 한꺼번에 조회 → 성능 저하 가능 |
| 기본 설정 | @ManyToOne, @OneToOne → EAGER 기본 | @OneToMany, @ManyToMany → LAZY 기본 |

실무에서는 거의 모든 연관관계를 LAZY로 설정하고, 필요한 경우에 **fetch join** / **EntityGraph** / **QueryDSL**로 즉시 로딩한다고 한다.

<br>

### 동작 원리 예제

**지연 로딩(LAZY)**

```java
@Entity // 엔티티 구조
class Member {
    @Id
    private Long id;
    
    @ManyToOne(fetch = FetchType.LAZY) // 지연 로딩
    private Team team;
}
```

```java
Member member = em.find(Member.class, 1L); // 여기서는 Member만 SELECT
member.getTeam().getName(); // 이 시점에서 Team을 SELECT (추가 쿼리 발생)
```

**실행되는 쿼리**

```sql
-- 첫 번째 쿼리
SELECT * FROM member WHERE id = 1;

-- 실제 사용할 때
SELECT * FROM team WHERE id = ?;
```

프록시 객체를 먼저 반환하고, 실제 값에 접근할 때 DB에서 조회한다.

<br>

**즉시 로딩(EAGER)**

```java
@Entity // 엔티티 구조
class Member {
    @ManyToOne(fetch = FetchType.EAGER) // 즉시 로딩
    private Team team;
}
```

```java
// Member를 조회하는 순간 Team도 JOIN으로 함께 조회
Member member = em.find(Member.class, 1L);
```

**실행되는 쿼리**

```sql
SELECT m.*, t.*
FROM member m
LEFT JOIN team t ON m.team_id = t.id
WHERE m.id = 1;
```

무조건 즉시 JOIN을 사용해서 가져온다.

실제로 Team 정보가 필요하지 않더라도 매번 JOIN이 발생한다.

그렇다면 왜 지연 로딩이 권장될까?

즉시 로딩은 불필요한 JOIN이 과다 발생하고, 연관관계가 늘어나면 예상치 못한 쿼리가 실행된다. 또한 N + 1 문제를 유발할 가능성이 더 크며 쿼리 최적화와 관련하여 개발자가 제어하기 어렵기 때문이다.

따라서 실무에서는 `@OneToOne`, `@ManyToOne`도 기본을 LAZY로 바꿔 사용한다.

#### 한 줄 정리

LAZY: 필요할 때 가져와(기본적으로 권장)

EAGER: 미리 다 가져와(성능 예측이 어려워서 가급적 피함)

<br>

### JPQL

**엔티티 객체를 대상으로 쿼리를 작성하는 객체 지향 쿼리 언어**로, SQL처럼 보이지만 실제로는 테이블이 아니라 엔티티 클래스와 그 필드를 대상으로 동작한다.

즉, SQL은 데이터베이스 테이블을 대상으로, JPQL은 영속성 컨텍스트에서 관리되는 엔티티 객체를 대상으로 한다.

JPA의 기본 CRUD(`persist`, `find`, `remove`)만으로는 복잡한 조건 검색(ex. 나이가 30 이상인 회원)을 처리하기 어렵다.

이럴 때 JPQL을 쓰면 SQL처럼 복잡한 조회 쿼리를 작성할 수 있으면서도 DB에 의존하지 않는 객체 중심 쿼리를 작성할 수 있다.

#### 기본 문법

SQL과 매우 비슷하지만, 차이점은 테이블 대신 엔티티명과 필드명을 쓴다는 점이다.

```plaintext
// SQL
SELECT * FROM member m WHERE m.age > 30;

// JPQL
SELECT m FROM Member m WHERE m.age > 30;
```

| 구분 | SQL | JPQL |
|------|------|------|
| 대상 | 테이블 | 엔티티 |
| 컬럼 | 컬럼명 | 엔티티의 필드명 |
| 결과 | 레코드(행) | 엔티티 객체 |
| 실행 결과 타입 | ResultSet | Entity, DTO 등 |

<br>

#### 예제 코드

```java
List<Member> result = em.createQuery(
        "SELECT m FROM Member m WHERE m.age > :age", Member.class)
        .setParameter("age", 20)
        .getResultList();

for (Member m : result) {
    System.out.println(m.getName());
}
```

`em.createQuery()`: JPQL 실행 객체 생성

`:age`: 파라미터 바인딩

`getResultList()`: 결과 목록 반환(없으면 빈 리스트 반환)

<br>

### Fetch Join

`Fetch Join`은 JPA 성능 최적화의 핵심 기능 중 하나로, 지연 로딩(LAZY)으로 설정된 연관 엔티티를 한 번의 쿼리로 함께 조회하기 위한 JPQL 문법이다.

JPA의 기본 전략은 **연관 엔티티는 지연 로딩(LAZY)으로 두는 것**이다.

하지만 LAZY 상태에서 연관 엔티티에 접근하면 실제 쿼리가 여러 번 실행되어 N + 1 문제가 발생할 수 있다.

#### N + 1 문제 예시

```java
List<Member> members = em.createQuery("SELECT m FROM Member m", Member.class).getResultList();

for (Member m : members) {
    System.out.println(m.getTeam().getName()); // 여기서 팀 조회 SQL이 계속 추가 발생
}
```

1번: Member 전체 조회

N번: Member 수만큼 각각 Team 조회

→ 총 1 + N개의 쿼리

따라서, **연관된 엔티티를 JOIN해서 한 번에 같이 조회하는 기법**인 `Fetch Join`을 사용한다.

JPQL에서 `join fetch` 문법을 사용하면, 연관된 엔티티를 즉시 로딩(EAGER)하되, Hibernate가 강제로 JOIN으로 한 번에 가져온다.

```java
SELECT m FROM Member m
JOIN FETCH m.team
WHERE m.age > 30
```

- Member와 Team을 JOIN으로 한 번에 조회
- Team은 LAZY로 설정되어 있더라도 즉시 초기화됨
- 1차 캐시(영속성 컨텍스트)에 저장되어 N + 1 문제를 해결

<br>

### @EntityGraph

`@EntityGraph`는 Fetch Join과 같은 효과를 **JPQL을 작성하지 않고** 얻을 수 있게 해주는 JPA/Spring Data JPA의 기능이다.

조회 시 어떤 연관 엔티티를 함께 가져올지를 어노테이션으로 명시하는 방법이다.

즉, 엔티티에 정의된 연관관계(fetch = LAZY)를 유지하되, 특정 조회 메서드에서는 선택적으로 즉시 로딩(EAGER)처럼 되도록 만드는 기능이다.

Fetch Join과 비슷한 기능을 가지지만, JPQL 쿼리를 직접 쓰지는 않고, 리포지토리 메서드 레벨에서 조정할 수 있는 선택적 즉시 로딩 전략이다.

1. JPQL 없이, 메서드 이름 기반으로 조회할 때
2. QueryDSL, Specification 등과 함께 사용할 때
3. Fetch Join처럼 연관 데이터를 한 번에 가져오고 싶을 때

사용한다.

#### 예시

**엔티티 구조**

```java
@Entity
public class Member {
    @ManyToOne(fetch = FetchType.LAZY)
    private Team team;
}
```

**Repository에 `@EntityGraph` 적용**

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    // team도 함께 로딩
    @EntityGraph(attributePaths = {"team"})
    @Query("SELECT m FROM Member m WHERE m.name = :name")
    Member findByName(@Param("name") String name);
}
```

실행 결과, `Fetch Join`과 같은 SQL이 실행된다.

| 기능 | Fetch Join | EntityGraph |
|------|-------------|--------------|
| 작성 방식 | JPQL 직접 작성 | 어노테이션 기반, 선언적 |
| 적용 위치 | 쿼리 단위 | Repository 메서드 단위 |
| 유지보수 | JPQL 수정 필요 | 엔티티 필드명만 유지하면 됨 |

<br>

### flush와 commit의 차이점

`flush` - 영속성 컨텍스트의 변경 내용을 DB에 SQL로 반영만 하는 작업

`commit` - 트랜잭션을 종료하고, DB에 반영된 내용을 확정하는 작업

flush는 **SQL을 DB로 보낸다**

commit은 **그 SQL 결과를 진짜로 DB에 확정한다(트랜잭션 종료)**

| 항목            | flush                                            | commit           |
|---------------|--------------------------------------------------|------------------|
| 목적            | 영속성 컨텍스트 → DB로 SQL 전송                            | 트랜잭션 완료 및 확정     |
| 트랜잭션 종료       | X                                                | O                |
| 롤백 가능 여부      | 가능                                               | 불가능              |
| SQL 실행 시점     | 변경감지(dirty checking) 후 SQL 생성 및 DB에 전송           | flush 자동 실행 후 commit으로 확정 |
| flush 후 DB 상태 | SQL은 실행됐지만 트랜잭션이 commit되지 않음 → 다른 트랜잭션에서는 보이지 않음 | DB에 최종 반영되어 다른 트랜잭션에서도 조회 가능 |
| 주 사용 목적       | JPQL 실행 시 정합성 유지, 중간 테스트                         | 트랜잭션 종료          |

#### flush가 자동으로 발생하는 시점

1. 트랜잭션 commit 시
2. JPQL, QueryDSL 쿼리 실행 전
3. 명시적으로 `em.flush()` 호출 시

즉 commit 전에 flush가 자동으로 호출되며 flush 없이 commit만 해도 결국 flush → commit 순으로 일어난다.

#### 어차피 flush가 반영되는데, 수동으로 왜 사용할까?

1. 중간에 SQL을 강제로 DB에 반영하고 싶은 경우
2. 벌크 연산 후 영속성 컨텍스트와 DB 상태를 맞추고 싶은 경우
3. JPQL 실행 전에 명확한 상태로 만들어야 할 때

<br>

### QueryDSL, OpenFeign의 QueryDSL

`QueryDSL`은 JPA를 위한 쿼리 빌더 도구이고, `OpenFeign`은 HTTP 통신을 위한 클라이언트 라이브러리이다.

### QueryDSL

JPQL이 `문자열 기반 쿼리 언어`라면, QueryDSL은 그것을 타입 안정성 있는 자바 코드로 바꿔주는 쿼리 빌더이다. 즉, JPQL의 발전형이라고 부른다.

주로 JPA와 함께 사용하며, Repository 계층에서 데이터 조회를 최적화하는 용도이다.

#### 예시

```java
List<Member> result = queryFactory
        .selectFrom(member)
        .where(member.age.gt(20)
                .and(member.status.eq(Status.ACTIVE)))
        .fetch();
```

문자열 기반이라는 JPQL의 단점을 보완한다.

JPQL로 쿼리를 작성할 때에는 오타 하나만 나도 컴파일 에러가 아닌 런타임 에러로 터지지만, QueryDSL은 코드로 작성하므로 IDE 자동완성 + 타입 검사를 받을 수 있다.

### OpenFeign

Spring Cloud에서 사용하는 HTTP API 호출용 라이브러리로, REST API를 인터페이스 기반으로 호출할 수 있게 해주는 기술이다.

Feign은 `HTTP 요청을 대신 보내주는 도구`이며, DB나 쿼리와는 관련이 없다.

#### 예시

```java
@FeignClient(name = "userClient", url = "https://api.example.com")
public interface UserClient {
    @GetMapping("/users")
    List<UserResponse> getUsers(@RequestParam("age") int age);
}
```

여기서 파라미터(age)는 HTTP 쿼리 파라미터이고, QueryDSL과는 아무 관련이 없다.

즉, `QueryDSL`은 JPA용 쿼리 빌더이고, `OpenFeign`은 HTTP API 호출 도구이다.

OpenFeign의 QueryDSL이라는 용어는 존재하지 않으며, 두 기술은 전혀 다른 계층에서 동작한다.

<br>

### N + 1 문제를 해결할 수 있는 여러 방안들

N + 1 문제는 한 번의 쿼리로 데이터를 가져온 후, 연관된 엔티티를 조회하기 위해 추가 쿼리가 발생하는 문제로, 지연 로딩(LAZY)으로 인해 반복 쿼리가 발생하는 것이 근본적인 원인이다.

1. Fetch Join

가장 강력하고 많이 쓰이는 해결 방법이다.

```java
SELECT m FROM Member m JOIN FETCH m.team
```

위 예에서는 Member + Team을 한 번에 로딩하여 N + 1 현상을 근본적으로 차단한다.

2. @EntityGraph

```java
@EntityGraph(attributePaths = {"team"})
@Query("SELECT m FROM Member m")
List<Member> findAll();
```

Fetch Join과 동일한 SQL을 생성한다.

Spring Data JPA에서 매우 자주 사용하는 해결 방법이다.

3. Batch Size 설정(IN 쿼리로 최적화)

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 100
```

```java
@OneToMany(fetch = LAZY)
@BatchSize(size = 100)
private List<Order> orders;
```

연관된 엔티티를 한꺼번에 IN 쿼리로 로드한다.

완벽한 Fetch Join은 아니지만 페이징과 함께 사용할 수 있는 가장 현실적인 전략이다.

1차 쿼리 후, LAZY 로딩 시 다음과 같이 실행된다.

```sql
SELECT * FROM order WHERE member_id IN (?, ?, ?, ...);
```

4. DTO Projection(JPQL/QueryDSL)

```java
SELECT new com.example.MemberDTO(m.id, m.name, t.name)
FROM Member m
JOIN m.team t
```

엔티티가 아니라 DTO로 바로 조회하여 불필요한 연관 로딩을 방지한다.

응답 전용 API에서 많이 사용한다.

5. 2차 캐시 활용

변하지 않는 데이터에 적합하다.

중복 SELECT를 방지하지만 캐시 일관성 문제로 신중하게 사용해야 한다.

6. OSIV 설정 관련 전략

ON(기본값): Controller까지 영속성 컨텍스트를 열어둠

OFF: Service 계층에서만 영속성 컨텍스트를 사용

즉 OSIV OFF는 Lazy Loading을 Service 계층 안에서만 허용함으로써, 반드시 Fetch Join/EntityGraph 등을 사용하도록 강제하여 N + 1 문제를 구조적으로 차단하는 전략이다.

<br>

### 영속 상태의 종류

엔티티는 영속성 컨텍스트 안에서 다음 네 가지 상태 중 하나를 가진다.

**비영속(new/transient)**: 아직 영속성 컨텍스트에 저장되지 않은 상태(`new` 키워드로 단순 생성된 객체)

**영속(managed)**: 영속성 컨텍스트에 의해 관리되는 상태(`em.persist(entity)`)

**준영속(detached)**: 한때 영속이었으나, 현재는 관리되지 않는 상태(`em.detach(entity)` or `em.clear()`)

**삭제(removed)**: 삭제 예약 상태(`em.remove(entity)`)