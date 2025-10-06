# WEEK 4 - 제이/한종서

## 핵심 키워드

### 계층형 구조 vs 도메인형 구조

`계층형 구조`와 `도메인형 구조`는 **프로젝트 패키지를 어떻게 나눌 것인가**에 대한 철학 차이로, 프로젝트 시작 단계에서 자주 등장하는 주제이다.

1. 계층형 구조(Layered Architecture)

일반적으로, Controller -> Service -> Repository 같은 Layer별로 패키지를 나누는 방식이다.

예시

```plaintext
com.example.demo
 ├── controller
 │    └── MemberController.java
 ├── service
 │    └── MemberService.java
 ├── repository
 │    └── MemberRepository.java
 ├── entity
 │    └── Member.java
```

특징

- 계층(Layer) 중심: 같은 역할을 하는 것들끼리 묶는다.
- 구조가 직관적이다.
- `Controller는 요청 처리만`, `Service는 비즈니스 로직만`처럼 Layer 책임이 명확하다.

단점

- 규모가 커지면 도메인 간 응집도가 떨어지고, 한 패키지(ex. service) 안에 파일이 수십, 수백 개가 모여서 관리가 힘들어질 수 있다.
- 서로 다른 도메인(ex. Service끼리) 간의 경계가 불분명해질 수 있다.

<br>

2. 도메인형 구조(Domain-Driven Architecture)

도메인(업무 단위, 비즈니스 단위)별로 패키지를 나누는 방식이다.

예시

```plaintext
com.example.demo
 ├── member
 │    ├── controller
 │    │    └── MemberController.java
 │    ├── service
 │    │    └── MemberService.java
 │    ├── repository
 │    │    └── MemberRepository.java
 │    └── entity
 │         └── Member.java
 ├── food
 │    ├── controller
 │    ├── service
 │    ├── repository
 │    └── entity
```

특징

- 도메인 중심: 회원, 음식, 리뷰 같은, 기능별로 파일들을 묶는다.
- 관련된 코드가 한 패키지 안에 모여 있기 때문에 응집도가 높다.
- 서비스가 커져도 모듈화를 통해 유지보수하기 쉽다.

단점

- 계층 간 일관성이 조금 약해질 수 있다. 예를 들면, Controller가 여러 도메인을 다루는 경우가 있을 수 있다.

<br>

### JPA

JPA(Java Persistence API)는 자바에서 데이터베이스를 객체지향적으로 다룰 수 있게 해주는 표준 API이다.

ORM(Object-Relational Mapping) 기술의 일종으로, 자바 클래스와 데이터베이스 테이블을 매핑해주어, SQL을 직접 쓰지 않고도 객체를 생성/조회/수정/삭제할 수 있다.

유의해야 할 점으로는 JPA 자체는 구현체가 아니고, 규격이라는 점이다.

JPA를 실제로 구현한 대표적인 프레임워크로는 `Hibernate`, `EclipseLink`, `OpenJPA` 등이 있다.

#### 장점

- SQL 대신 자바 코드로 DB 작업을 할 수 있다. 이는 생산성을 증가시킨다.
- 객체 중심으로 개발할 수 있게 하여 객체지향 패러다임을 유지할 수 있다.
- DB 독립적이기 때문에 DBMS가 바뀌어도 코드 수정을 최소화할 수 있다.
- 캐싱, 지연 로딩(Lazy Loading) 같은 성능 최적화 기능도 제공한다.

#### 주요 개념

- `Entity`: 테이블과 매핑되는 자바 클래스
- `EntityManager`: 엔티티의 생성, 조회, 수정, 삭제를 담당하는 핵심 객체
- `JPQL(Java Persistence Query Language)`: 객체지향 쿼리 언어로, SQL과 유사하지만 엔티티 객체 단위로 조회

#### 한 줄 요약

JPA는 **자바 클래스 = DB 테이블**로 매핑해주는 ORM 표준 기술이며, `Hibernate` 같은 구현체가 실제 동작을 담당한다.

<br>

#### Spring Data JPA

중요한 점은 JPA 자체는 실행 가능한 코드가 아니라는 점이다. 인터페이스와 규약만을 제공하며, 이러한 JPA 명세를 실제로 구현해서 사용해야 한다.

이때, `Spring Data JPA`를 쓰면 편하다.

`Spring Data JPA`는 스프링에서 제공하는, JPA 사용을 더 편리하게 해주는 라이브러리이다.

JPA + Hibernate 같은 구현체를 쉽게 쓸 수 있도록 도와준다.

#### 특징

- `CrudRepository`, `JpaRepository` 같은 인터페이스만 상속받아도 기본 CRUD 기능이 자동으로 구현된다.
- 메서드 이름만으로 쿼리를 사용할 수 있다. (ex. `findByUsername`, `findByEmailAndStatus`)
- 필요하면 직접 `@Query` 어노테이션을 사용해 JPQL이나 네이티브 쿼리를 작성할 수도 있다.

#### 요약

`JPA` 표준(명세)

`Hibernate` JPA의 대표 구현체

`Spring Data JPA` JPA + 구현체(Hibernate 등)를 더 쉽게 쓸 수 있게 하는 스프링 모듈

<br>

### N + 1 문제

1개의 쿼리를 날렸을 때 N개의 추가 쿼리가 발생하는 현상이다.

주로 즉시 로딩(Eager Loading) 전략과 지연 로딩(Lazy Loading) 전략이 복합적으로 사용될 때 발생한다.

- 즉시 로딩(Eager Loading): 연관 관계가 있는 엔티티를 조회할 때, 연관된 엔티티도 함께 조회하는 전략이다. 이는 `@OneToOne`과 `@ManyToOne`의 기본값이다.
- 지연 로딩(Lazy Loading): 연관 관계가 있는 엔티티를 조회할 때, 연관된 엔티티는 프록시 객체로 남겨두고, 실제 사용될 때(호출될 때) 조회하는 전략이다. 이는 `@OneToMany`와 `@ManyToMany`의 기본값이다.

구체적으로, N + 1 문제는 다음과 같은 상황에서 발생한다.

1. 지연 로딩을 사용하는 `@OneToMany` 관계에서 모든 연관된 엔티티를 순회하며 접근할 때
 - 1개의 쿼리: `Team` 엔티티 목록을 가져오는 쿼리
 - N개의 쿼리: 조회된 `N`개의 `Team` 엔티티 각각에 대해 연관된 `Member` 엔티티를 조회하는 쿼리

총 1 + N개의 쿼리가 실행된다.

2. 즉시 로딩을 사용하는 경우
 - `@ManyToOne`이나 `@OneToOne` 관계에서 즉시 로딩을 사용하면 의도하지 않게 연관된 엔티티를 모두 로드한다. 예를 들어, `Member` 엔티티를 100개 조회했는데, `Team` 엔티티가 즉시 로딩으로 설정되어 있다면 `Member`를 조회하는 1번의 쿼리 외에 `Team`을 조회하는 쿼리가 100번 더 나가는 문제가 발생할 수 있다.

#### 해결 방안

1. 지연 로딩(Lazy Loading) 사용

**모든 연관 관계를 지연 로딩으로 설정하는 것**이 가장 기본적인 해결책(?)이다.

예를 들어, `@ManyToOne`의 기본 로딩 방식인 즉시 로딩을 `fetch = FetchType.LAZY`로 변경하는 것이 좋다. `@ManyToOne(fetch = FetchType.LAZY)`

하지만, 필요할 때 쿼리를 날려도 반복문 같은 곳에서 연관 데이터를 여러 번 접근하면 여전히 N + 1 문제가 발생한다.

2. **Fetch Join 사용**

JPQL(Java Persistence Query Language)에서 `JOIN FETCH`를 사용하여 연관된 엔티티를 한 번의 쿼리로 함께 가져오는 방법이다.

예시

```java
// N + 1 문제가 발생하는 코드
List<Team> teams = teamRepository.findAll();

// Fetch Join을 사용하여 N + 1 문제 해결
@Query("SELECT t FROM Team t JOIN FETCH t.members")
List<Team> findAllWithMembers();
```

3. 엔티티 그래프(Entity Graph) 사용

`@EntityGraph` 어노테이션을 사용하여 미리 로딩할 연관 엔티티를 명시하는 방법이다.

4. Batch Size 설정

`Hibernate`의 `hibernate.default_batch_fetch_size` 속성을 설정하여, 지정된 크기만큼의 엔티티들을 한꺼번에 조회하는 쿼리를 생성하도록 할 수 있다.

<br>

### 기본 키 생성 전략

JPA는 엔티티의 PK(Primary Key) 값을 어떻게 생성할지에 대해 몇 가지 전략을 제공한다.

1. 직접 할당

개발자가 직접 ID 값을 세팅하는 방식이다.

`@Id`만 선언하면 된다.

```java
@Id
private String name;
```

이는 PK를 직접 넣어주는 방식으로, 단순하며 제어권이 100% 개발자에게 있다.

그렇기 때문에 실수할 가능성이 존재하며, 자동 증가 같은 기능 또한 없다.

2. 자동 생성(`@GeneratedValue`)

자동으로 PK 값을 만들어주는 전략이다.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

자동으로 PK 값을 만들어주는 전략은 여러 가지가 있다.

2-1. AUTO

JPA 구현체(Hibernate)가 DB 방언(dialect)에 맞춰 자동으로 선택한다.

예를 들어, MySQL은 IDENTITY, Oracle은 SEQUENCE. 이 방식은 설정할 때는 편하지만 DB를 바꾸면 전략이 바뀔 수 있다는 점을 기억하면 된다.

2-2. IDENTITY

DB의 `AUTO_INCREMENT` 기능을 사용한다. (MySQL, MariaDB 등)

DB가 PK를 생성하고, JPA는 INSERT 후 DB가 생성한 값을 가져온다.

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;
```

DB가 알아서 기본 키를 생성하는 방식이다.

DB INSERT 전에 PK를 활용할 수 없다는 특징이 있다.

2-3. SEQUENCE

DB의 시퀀스 오브젝트를 사용한다. (Oracle, PostgreSQL 등)

`@SequenceGenerator`와 함께 사용한다.

직접 만드는 경우

```sql
CREATE SEQUENCE member_seq START WITH 1 INCREMENT BY 1;
```

JPA에서의 사용법

```java
@Id
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq")
@SequenceGenerator(
        name = "member_seq",
        sequenceName = "member_sequence",
        allocationSize = 1
)
private Long id;
```

IDENTITY 방식과 다르게 DB INSERT 전에 PK를 활용할 수 있어 캐시 최적화에 유리하다는 점이 있다.

하지만 MySQL처럼 시퀀스가 없는 DB에서는 사용할 수 없다.

2-4. TABLE

별도의 키 생성 전용 테이블을 만들어 PK를 관리한다.

DB 독립적으로 사용이 가능하지만 성능이 가장 떨어진다.