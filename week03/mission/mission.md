# WEEK 3 - 제이/한종서

## 미션

1주차에서 설계한 데이터베이스를 기준으로 수행하였습니다.

### 홈 화면

- API Endpoint

`GET /api/home`

- Path Variable

`X`

- Query String

`GET /api/home?localName=안암동`

- Request Header

`Authorization: Bearer {accessToken}`

- Request Body

`X`

### 마이 페이지 리뷰 작성

- API Endpoint

`POST /api/stores/{store-id}/reviews`

- Path Variable

`{store-id}`

- Query String

`X`

- Request Header

Authorization: Bearer {accessToken}

Content-Type: multipart/form-data

- Request Body

```yaml
star:
  type: float
  nullable: X
content:
  type: string
  nullable: X
photo:
  type: image
  nullable: O
```

`예시`

```plaintext
------boundary
Content-Disposition: form-data; name="star"

5.0
------boundary
Content-Disposition: form-data; name="content"

음 너무 맛있어요 포인트도 얻고 맛있는 맛집도 알게 된 것 같아 너무나도 행복한 식사였답니다. 다음에 또 올게요!!
------boundary
Content-Disposition: form-data; name="photos"; filename="img.jpg"
Content-Type: image/jpeg

<바이너리>
------boundary--
```

서버 측에서는 MultipartFile[] photos 배열로 1장(photos[0]) 또는 여러 장을 받아 이미지를 저장합니다.

### 미션 목록 조회(진행중, 진행 완료)

- API Endpoint

`GET /api/missions`

- Path Variable

`X`

- Query String

`GET /api/missions?status=completed`

- Request Header

`Authorization: Bearer {accessToken}`

- Request Body

`X`

### 미션 성공 누르기

- API Endpoint

`POST /api/missions/{mission-id}/complete-request`

- Path Variable

`{mission-id}`

- Query String

`X`

- Request Header

`Authorization: Bearer {accessToken}`

- Request Body

`X`

### 회원가입하기(소셜 로그인 고려 X)

- API Endpoint

`POST /auth/users`

- Path Variable

`X`

- Query String

`X`

- Request Header

`Content-Type: application/json`

- Request Body

```yaml
user_term:
  type: array
  items:
    type: string
    enum: [service, privacy, location, marketing]
  nullable: X
name:
  type: string
  nullable: X
gender:
  type: string
  enum: [male, female, none]
  nullable: X
birth:
  type: string
  nullable: X
address:
  type: string
  nullable: X
detail_address:
  type: string
  nullable: O
user_prefer_food:
  type: array
  items:
    type: string
    enum: [한식, 일식, 중식, 양식, 치킨, 분식, 고기/구이, 도시락, 야식(족발,보쌈), 패스트푸드, 디저트, 아시안푸드]
  nullable: X
```

`예시`

```json
{
  "user_term": ["service", "privacy", "marketing"],
  "name": "홍길동",
  "gender": "male",
  "birth": "2005-01-25",
  "address": "서울특별시 노원구 월계동",
  "detail_address": "101동 501호",
  "user_prefer_food": ["한식", "일식", "중식"]
}
```

소셜 로그인이 아니라면 ID, Password 입력도 필요합니다.