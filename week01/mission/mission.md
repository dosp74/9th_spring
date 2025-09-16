# WEEK 1 - 제이/한종서

## 미션

### 설계한 DB 사진

![Image](/week00/mission/mission_1.png)

![Image](/week00/mission/mission_2.png)

### 본문

1. 리뷰 작성하는 쿼리

```sql
INSERT INTO review (id, user_id, store_id, star, content, created_at, updated_at)
VALUES (1, 25, 50, 5.0, '음 너무 맛있어요 포인트도 얻고 맛있는 맛집도 알게 된 것 같아 너무나도 행복한 식사였답니다. 다음에 또 올게요!!', NOW(), NULL);
```

2. 마이 페이지 화면 쿼리

```sql
SELECT name, email, CASE
    WHEN phone_number IS NULL THEN '미인증'
    ELSE phone_number
                    END AS phone_number_status, point
FROM user
WHERE id = ?; -- 로그인한 사용자 식별(설계 안 된 부분)
```

`CASE ... END` 결과 컬럼에 이름을 붙이지 않으면, DBMS가 자동으로 붙이거나(가독성 떨어짐) 에러가 날 수도 있기 때문에 `AS`로 별칭을 지어주는 것이 좋다.

3. 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

```sql
SELECT mission.point, user_mission.is_complete, store.name, mission.content
FROM mission
JOIN user_mission ON mission.id = user_mission.mission_id
JOIN store ON mission.store_id = store.id
WHERE user_mission.user_id = ? -- 로그인한 사용자 식별(설계 안 된 부분)
LIMIT 10 OFFSET 0;
```

4. 홈 화면 쿼리(현재 선택된 지역에서 도전이 가능한 미션 목록, 페이징 포함)

```sql
SELECT l.name, s.name, m.content, m.point, m.deadline
FROM mission AS m
JOIN store AS s ON m.store_id = s.id
JOIN local AS l ON s.local_id = l.id
WHERE l.id = ? -- 현재 선택된 지역
    AND m.deadline >= CURRENT_DATE
LIMIT 10 OFFSET 0;
```

`CURRENT_DATE`의 반환 값은 `YYYY-MM-DD`

이 경우 `m.deadline >= CURRENT_DATE`는 deadline의 날짜가 오늘이거나 오늘 이후이면 통과한다.