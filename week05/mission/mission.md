# WEEK 5 - 제이/한종서

## 미션

JPQL 방식을 이용하여 1주차 미션 때 짰던 쿼리들 리팩토링하기

### 본문

1. 리뷰 작성하는 쿼리

`ReviewRepository.java`

```java
package com.example.demo.domain.review.repository;

import com.example.demo.domain.review.entity.Review;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ReviewRepository extends JpaRepository<Review, Long> {
}
```

`ReviewService.java`

```java
package com.example.demo.domain.review.service;

import com.example.demo.domain.member.entity.Member;
import com.example.demo.domain.review.entity.Review;
import com.example.demo.domain.review.repository.ReviewRepository;
import com.example.demo.domain.store.entity.Store;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional
public class ReviewService {
    private final ReviewRepository reviewRepository;

    public void createReview(Member member, Store store, Float star, String content) {
        Review review = Review.builder()
                .member(member)
                .store(store)
                .star(star)
                .content(content)
                .build();

        reviewRepository.save(review);
    }
}
```

`JpaRepository`를 상속하여 `save()` 메서드를 사용, 리뷰 데이터를 DB에 저장한다.

`ReviewService` 클래스에 리뷰 생성 로직을 별도로 둬서 역할을 분리하였다.

2. 마이 페이지 화면 쿼리

`MemberRepository.java`

```java
package com.example.demo.domain.member.repository;

import com.example.demo.domain.member.entity.Member;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MemberRepository extends JpaRepository<Member, Long> {
}
```

`MyPageResponseDto.java`

```java
package com.example.demo.domain.member.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor
public class MyPageResponseDto {
    private String name;
    private String email;
    private String phoneNumberStatus;
    private Integer point;
}
```

`MemberService.java`

```java
package com.example.demo.domain.member.service;

import com.example.demo.domain.member.dto.MyPageResponseDto;
import com.example.demo.domain.member.entity.Member;
import com.example.demo.domain.member.repository.MemberRepository;
import jakarta.persistence.EntityNotFoundException;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberService {
    private final MemberRepository memberRepository;

    public MyPageResponseDto getMyPage(Long memberId) {
        Member member = memberRepository.findById(memberId)
                .orElseThrow(() -> new EntityNotFoundException("회원이 존재하지 않습니다."));

        String phoneNumberStatus =
                (member.getPhoneNumber() == null) ? "미인증" : member.getPhoneNumber();

        return new MyPageResponseDto(
                member.getName(),
                member.getEmail(),
                phoneNumberStatus,
                member.getPoint()
        );
    }
}
```

JPA의 기본 메서드인 `findById()`를 사용하여 회원을 조회한다.

회원의 이름, 이메일, 휴대폰 인증 여부, 포인트를 조회한다.

계층 간에 필요한 데이터만을 하나의 묶음으로 만들어 전달하고, 응답 형식을 통일하기 위해 DTO를 도입하였다.

3. 내가 진행중, 진행 완료한 미션 모아서 보는 쿼리(페이징 포함)

`MemberMissionRepository.java`

```java
package com.example.demo.domain.mission.repository;

import com.example.demo.domain.mission.dto.MemberMissionResponseDto;
import com.example.demo.domain.mission.entity.mapping.MemberMission;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.util.List;

public interface MemberMissionRepository extends JpaRepository<MemberMission, Long> {
    // 진행중 미션(isComplete = false)
    @Query("""
        SELECT new com.example.demo.domain.mission.dto.MemberMissionResponseDto(
            s.name, m.content, m.point, mm.isComplete
        )
        FROM MemberMission mm
        JOIN mm.mission m
        JOIN m.store s
        WHERE mm.member.id = :memberId
            AND mm.isComplete = false
    """)
    List<MemberMissionResponseDto> findOngoingMissions(
            @Param("memberId") Long memberId,
            Pageable pageable
    );

    // 진행 완료한 미션(isComplete = true)
    @Query("""
        SELECT new com.example.demo.domain.mission.dto.MemberMissionResponseDto(
            s.name, m.content, m.point, mm.isComplete
        )
        FROM MemberMission mm
        JOIN mm.mission m
        JOIN m.store s
        WHERE mm.member.id = :memberId
            AND mm.isComplete = true
    """)
    List<MemberMissionResponseDto> findCompletedMissions(
            @Param("memberId") Long memberId,
            Pageable pageable
    );
}
```

`MemberMissionResponseDto.java`

```java
package com.example.demo.domain.mission.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;

@Getter
@AllArgsConstructor
public class MemberMissionResponseDto {
    private String storeName;
    private String content;
    private Integer point;
    private Boolean isComplete;
}
```

`MemberMissionService.java`

```java
package com.example.demo.domain.mission.service;

import com.example.demo.domain.mission.dto.MemberMissionResponseDto;
import com.example.demo.domain.mission.repository.MemberMissionRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.PageRequest;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.List;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MemberMissionService {
    private final MemberMissionRepository memberMissionRepository;

    public List<MemberMissionResponseDto> getOngoingMissions(Long memberId, int page, int size) {
        return memberMissionRepository.findOngoingMissions(memberId, PageRequest.of(page, size));
    }

    public List<MemberMissionResponseDto> getCompletedMissions(Long memberId, int page, int size) {
        return memberMissionRepository.findCompletedMissions(memberId, PageRequest.of(page, size));
    }
}
```

로그인한 사용자가 진행 중이거나 진행 완료한 미션 목록을 조회한다.

`@Query` 어노테이션을 사용하여 복잡한 조건에 대한 쿼리문을 자바 코드로 작성하였다.

진행 중, 진행 완료에 대한 JPQL 쿼리를 각각 짜고 이 또한 별도의 응답 DTO를 두었다.

4. 홈 화면 쿼리(현재 선택된 지역에서 도전이 가능한 미션 목록, 페이징 포함)

`MissionRepository.java`

```java
package com.example.demo.domain.mission.repository;

import com.example.demo.domain.mission.dto.MissionHomeResponseDto;
import com.example.demo.domain.mission.entity.Mission;
import org.springframework.data.domain.Pageable;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import java.time.LocalDate;
import java.util.List;

public interface MissionRepository extends JpaRepository<Mission, Long> {
    @Query("""
        SELECT new com.example.demo.domain.mission.dto.MissionHomeResponseDto(
            l.name, s.name, m.content, m.point, m.deadline
        )
        FROM Mission m
        JOIN m.store s
        JOIN s.local l
        WHERE l.id = :localId
            AND m.deadline >= :today
    """)
    List<MissionHomeResponseDto> findAvailableMissionsByLocal(
            @Param("localId") Long localId,
            @Param("today") LocalDate today,
            Pageable pageable
    );
}
```

`MissionHomeResponseDto.java`

```java
package com.example.demo.domain.mission.dto;

import lombok.AllArgsConstructor;
import lombok.Getter;

import java.time.LocalDate;

@Getter
@AllArgsConstructor
public class MissionHomeResponseDto {
    private String localName;
    private String storeName;
    private String content;
    private Integer point;
    private LocalDate deadline;
}
```

`MissionService.java`

```java
package com.example.demo.domain.mission.service;

import com.example.demo.domain.mission.dto.MissionHomeResponseDto;
import com.example.demo.domain.mission.repository.MissionRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.data.domain.PageRequest;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.time.LocalDate;
import java.util.List;

@Service
@RequiredArgsConstructor
@Transactional(readOnly = true)
public class MissionService {
    private final MissionRepository missionRepository;

    public List<MissionHomeResponseDto> getAvailableMissions(Long localId, int page, int size) {
        return missionRepository.findAvailableMissionsByLocal(
                localId,
                LocalDate.now(),
                PageRequest.of(page, size)
        );
    }
}
```

JPQL로 지역과 마감일을 조건으로 미션을 조회한다. 이 또한 별도의 응답 DTO를 두었다.

[미션 레포지토리](https://github.com/dosp74/demo/tree/feature/chapter5)