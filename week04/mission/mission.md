# WEEK 4 - 제이/한종서

## 미션

![Image](/week04/mission/mission_1.png)

```java
@Getter
@EntityListeners(AuditingEntityListener.class)
@MappedSuperclass
public abstract class BaseEntity {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @CreatedDate
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    @Column(name = "updated_at")
    private LocalDateTime updatedAt;
}
```

BaseEntity에 공통 속성인 id, created_at, updated_at을 넣고 위와 같이 Member 관련 엔티티를 만들었다.

이후 DB에 테이블이 잘 생성되나 중간 점검으로 프로젝트를 실행시켜보았는데...

![Image](/week04/mission/mission_2.png)

오류가 발생했다.

무언가 복잡해보이는데 확실한 것은 `demo`라는 DB를 못 찾아서 발생한 문제인 것 같다.

그전에, 일단 MySQL이 실행중인 게 맞는지 확인해보았는데,

![Image](/week04/mission/mission_3.png)

MySQL 프로세스가 실행되고 있지 않았다.

프로세스를 시작시키고, demo DB를 생성하는 쿼리를 짰다.

![Image](/week04/mission/mission_4.png)

인텔리제이에서 SQL 콘솔을 띄우고 위와 같은 쿼리를 날려 demo DB를 생성해주었다.

![Image](/week04/mission/mission_5.png)

이후 프로젝트를 실행시키니 `Hibernate`가 create table DDL을 날린 것을 확인할 수 있었다.

![Image](/week04/mission/mission_6.png)

나머지 엔티티들도 만들어서 프로젝트를 실행시켜 DDL을 날려주었다.

![Image](/week04/mission/mission_7.png)

이후 DataGrip에서 생성된 DB를 확인할 수 있었다.

- 테이블 예시

![Image](/week04/mission/mission_8.png)

[미션 레포지토리](https://github.com/dosp74/demo/tree/feature/chapter4)