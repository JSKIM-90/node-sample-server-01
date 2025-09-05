# CLAUDE.md - 프로젝트 컨텍스트

## 프로젝트 개요
Node.js Express 서버 with JWT 인증

## Q&A

### Bearer 토큰이란?
"Bearer"는 HTTP Authorization 헤더에서 사용되는 인증 스킴(scheme) 타입입니다.

**형식:**
```
Authorization: Bearer <token>
```

**왜 Bearer를 사용하는가?**
1. **표준 규격**: RFC 6750 OAuth 2.0 표준에서 정의된 토큰 타입
2. **토큰 타입 구분**: 여러 인증 방식을 구별하기 위함
   - `Basic` - Base64로 인코딩된 username:password
   - `Bearer` - 토큰 기반 인증
   - `Digest` - 해시 기반 인증
   
3. **의미**: "Bearer"는 "소지자"라는 뜻으로, 이 토큰을 가진(bear) 사람이 리소스에 접근할 권한이 있다는 의미

**예시:**
```javascript
// 서버에서 토큰 파싱
const authHeader = req.headers["authorization"];
// "Bearer eyJhbGciOiJIUzI1NiIs..." 형태로 들어옴
const token = authHeader && authHeader.split(" ")[1]; 
// "Bearer " 부분을 제거하고 실제 토큰만 추출
```

클라이언트는 반드시 "Bearer " 접두사를 붙여서 보내야 서버가 올바른 인증 방식으로 인식합니다.

### 표준 규약으로서의 Authorization 헤더
네, 정확합니다! 웹 생태계의 모든 구성요소들(브라우저, 서버, 프레임워크 등)이 공통으로 합의한 표준입니다.

**합의된 내용:**
- `Authorization` 헤더 = 인증 정보를 전달하는 표준 위치
- `Bearer` = 토큰 기반 인증임을 알려주는 표준 식별자
- 이 규약은 RFC 문서로 명문화되어 전 세계가 동일하게 구현

**실제 구현:**
- **브라우저**: fetch API, axios 등이 Authorization 헤더 자동 처리
- **서버**: Express, Spring, Django 등 모든 프레임워크가 이 헤더를 파싱
- **프록시/게이트웨이**: nginx, Apache도 이 헤더를 인식하고 전달

이것이 "웹 표준"의 힘 - 모두가 같은 규칙으로 소통할 수 있게 만듦

네, 맞습니다. "문자열의 마법"이 아니라 명확한 "규칙과 표준"입니다. Bearer라는 단어 자체에 특별한 기능이 있는 게 아니라, 모두가 합의한 규칙이기 때문에 작동하는 것입니다.

## API 요청 예제

### 1. 회원가입 (POST /api/register)
```bash
curl -X POST http://localhost:3000/api/register \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "test1234",
    "email": "test@example.com"
  }'
```

**응답:**
```json
{
  "message": "User registered successfully",
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 2,
    "username": "testuser",
    "email": "test@example.com"
  }
}
```

### 2. 로그인 (POST /api/login)
```bash
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{
    "username": "testuser",
    "password": "test1234"
  }'
```

**응답:**
```json
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 2,
    "username": "testuser",
    "email": "test@example.com"
  }
}
```

### 3. 토큰 갱신 (POST /api/refresh-token) - 인증 필요
```bash
curl -X POST http://localhost:3000/api/refresh-token \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

**응답:**
```json
{
  "message": "Token refreshed successfully",
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

### 4. 프로필 조회 (GET /api/profile) - 인증 필요
```bash
curl -X GET http://localhost:3000/api/profile \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

**응답:**
```json
{
  "id": 2,
  "username": "testuser",
  "email": "test@example.com"
}
```

### 5. 프로필 수정 (PUT /api/profile) - 인증 필요
```bash
curl -X PUT http://localhost:3000/api/profile \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newemail@example.com",
    "password": "newpassword123"
  }'
```

**응답:**
```json
{
  "message": "Profile updated successfully",
  "user": {
    "id": 2,
    "username": "testuser",
    "email": "newemail@example.com"
  }
}
```

### 6. 데이터 생성 (POST /api/data) - 인증 필요
```bash
curl -X POST http://localhost:3000/api/data \
  -H "Authorization: Bearer YOUR_TOKEN_HERE" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "My First Post",
    "content": "This is the content of my post"
  }'
```

**응답:**
```json
{
  "message": "Data created successfully",
  "data": {
    "id": 1704067200000,
    "title": "My First Post",
    "content": "This is the content of my post",
    "userId": 2,
    "createdAt": "2024-01-01T00:00:00.000Z"
  }
}
```

### 7. 데이터 삭제 (DELETE /api/data/:id) - 인증 필요
```bash
curl -X DELETE http://localhost:3000/api/data/1704067200000 \
  -H "Authorization: Bearer YOUR_TOKEN_HERE"
```

**응답:**
```json
{
  "message": "Data with ID 1704067200000 deleted successfully",
  "deletedId": "1704067200000"
}
```

### 기존 엔드포인트 (인증 불필요)

#### 빈 객체 반환 (GET /empty)
```bash
curl http://localhost:3000/empty
```
**응답:** `{}`

#### 빈 응답 (GET /nothing)
```bash
curl http://localhost:3000/nothing
```
**응답:** (빈 문자열)

#### 특정 이름 조회 (GET /names/:id)
```bash
curl http://localhost:3000/names/5
```
**응답:**
```json
{
  "id": "5",
  "name": "John 5"
}
```

#### 이름 목록 조회 (GET /names)
```bash
curl http://localhost:3000/names
```
**응답:**
```json
[
  {"id": 1, "name": "John"},
  {"id": 2, "name": "Jane"},
  {"id": 3, "name": "Jim"}
]
```

## 참고사항
- 토큰은 기본적으로 1시간 후 만료 (`.env`의 TOKEN_EXPIRY로 변경 가능)
- YOUR_TOKEN_HERE 부분을 실제 받은 토큰으로 교체해서 사용
- Windows에서는 curl 대신 PowerShell의 Invoke-RestMethod 사용 가능

## JWT 토큰 vs JWT_SECRET 차이점

### JWT_SECRET (서버 비밀 키)
- `.env` 파일에 저장: `JWT_SECRET=RANDOM_TOKEN`
- 서버만 알고 있는 비밀 키
- 토큰을 생성(sign)하고 검증(verify)할 때 사용
- 절대 클라이언트에게 노출되면 안 됨

### JWT 토큰 (인증 토큰)
- 로그인/회원가입 시 서버가 생성해서 반환
- 형태: `eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MiwidXNlcm5hbWU...`
- 클라이언트가 Authorization 헤더에 포함시켜 보내는 값
- `Bearer` 뒤에 붙이는 것이 바로 이 토큰

### 올바른 사용법
1. 먼저 로그인 요청:
```bash
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","password":"test1234"}'
```

2. 응답에서 토큰 받기:
```json
{
  "token": "eyJhbGciOiJIUzI1NiIs...(실제 긴 토큰)"
}
```

3. 받은 토큰을 Authorization 헤더에 사용:
```bash
curl -X GET http://localhost:3000/api/profile \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs...(실제 받은 토큰)"
```

**주의**: "RANDOM_TOKEN"을 Bearer 뒤에 직접 넣으면 안 됩니다!


https://docs.tosspayments.com/blog/everything-about-basic-bearer-auth