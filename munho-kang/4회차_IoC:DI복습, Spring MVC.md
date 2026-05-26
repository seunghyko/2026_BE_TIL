### 전체 흐름

```
[ 클라이언트 (브라우저 / 앱) ]
           │  HTTP 요청/응답
           ▼
┌─────────────────────────┐
│      Controller         │  ← 요청을 받아서 누구한테 시킬지 결정
└─────────────────────────┘
           │
           ▼
┌─────────────────────────┐
│       Service           │  ← 실제 비즈니스 로직 수행
└─────────────────────────┘
           │
           ▼
┌─────────────────────────┐
│      Repository         │  ← DB에 데이터를 저장/조회
└─────────────────────────┘
           │  SQL / JPA
           ▼
┌─────────────────────────┐
│       Database          │
└─────────────────────────┘
```

> DTO는 계층 사이에서 데이터를 안전하게 전달하는 **택배 상자** 역할

### 각 계층의 한 줄 책임

```
Controller  →  "어떤 URL로 왔어? 누구한테 처리 맡길게. 결과 돌려줄게."
Service     →  "비즈니스 규칙에 따라 처리할게. DB는 Repository한테 맡길게."
Repository  →  "DB에 저장하고, 조회하고, 수정하고, 삭제할게."
DTO         →  "계층 간에 데이터 주고받을 때 나만 써. Entity 직접 노출 금지."
```

> **핵심 원칙**: 각 계층은 **바로 아래 계층**하고만 대화한다. Controller가 Repository를 직접 호출하는 건 규칙 위반.



## DTO (Data Transfer Object)

### DTO의 핵심 정의

Entity를 API 응답으로 그대로 내보내면 `password`, `ssn` 등 민감 정보가 노출된다.
DTO는 이를 막는 첫 번째 방어선이다.

```
Entity      →  DB와 매핑, 민감 정보 포함, 내부에서만 사용
DTO         →  필요한 데이터만 골라서 계층 간 전달용으로 사용

클라이언트 ──[RequestDto]──▶ Controller ──▶ Service ──▶ Repository ──▶ DB
클라이언트 ◀──[ResponseDto]── Controller ◀── Service ◀── Repository ◀── DB
```

### RequestDto vs ResponseDto

| 구분 | 역할 | 흐름 방향 |
|------|------|-----------|
| **RequestDto** | 클라이언트 → 서버로 오는 데이터 포장 | 입력 |
| **ResponseDto** | 서버 → 클라이언트로 나가는 데이터 포장 | 출력 |

### 핵심 원칙

```
✅ Entity는 절대 Controller 밖으로 나가면 안 된다
✅ 클라이언트 입력은 항상 RequestDto로 받는다
✅ 클라이언트 응답은 항상 ResponseDto로 내보낸다
✅ 변환 책임은 보통 Service 계층이 갖는다
✅ DTO는 로직 없이 데이터만 담는 순수한 객체여야 한다
```

---

## Repository

### Repository의 핵심 정의

> **Repository**란, **데이터베이스에 대한 CRUD 작업을 추상화한 계층**이다.
> Service는 "어떻게 DB에 저장하는지" 알 필요 없이, Repository의 메서드만 호출하면 된다.

### JPA / Spring Data JPA 계층 구조

```
[ Spring Data JPA ]   ← 메서드 이름으로 쿼리 자동 생성
        │
        ▼
    [ JPA ]           ← Java 표준 ORM 명세
        │
        ▼
  [ Hibernate ]       ← JPA 실제 구현체, SQL 자동 생성
        │
        ▼
  [ Database ]
```

> **ORM**: Java 객체와 DB 테이블을 자동으로 매핑해주는 기술.
> `User` 객체를 저장하면 Hibernate가 알아서 `INSERT SQL`을 만들어 실행한다.


### 핵심 원칙

```
✅ Repository는 인터페이스로 선언, 구현체는 Spring이 자동 생성
✅ 단순 조회는 메서드 이름 규칙으로 해결
✅ 복잡한 쿼리만 @Query 사용
✅ 단건 조회 결과는 Optional로 받아 NPE 방어
✅ Repository는 DB 작업만, 비즈니스 로직은 절대 넣지 않는다
```

---

## Service

### Service의 핵심 정의

> **Service**란, **애플리케이션의 핵심 비즈니스 규칙을 처리하는 계층**이다.
> Controller로부터 요청을 받아 Repository를 활용해 처리하고, 결과를 돌려준다.

### 비즈니스 로직이란?

```
회원가입 요청이 들어왔을 때...

❌ 비즈니스 로직이 아닌 것
  - HTTP 요청을 받는다         → Controller의 역할
  - DB에 INSERT 한다           → Repository의 역할

✅ 비즈니스 로직인 것
  - 이미 가입된 이메일인지 확인한다
  - 비밀번호를 암호화한다
  - 가입 환영 이메일을 발송한다
  - 14세 미만이면 가입을 거부한다
```

> **Dirty Checking (변경 감지)**: `@Transactional` 안에서 조회한 Entity를 수정하면
> `save()`를 호출하지 않아도 트랜잭션 종료 시 JPA가 변경을 감지해 자동으로 UPDATE SQL을 실행한다.

### 핵심 원칙

```
✅ 모든 비즈니스 규칙은 Service에만 존재한다
✅ 데이터 변경 메서드엔 @Transactional, 조회엔 readOnly = true
✅ 의존성 주입은 생성자 주입(@RequiredArgsConstructor)을 사용한다
✅ Entity 수정은 Dirty Checking을 활용, 불필요한 save() 호출을 피한다
✅ 예외는 커스텀 예외 클래스로 명확하게 표현한다
```

---

## Controller 

### Controller의 핵심 정의

> **Controller**란, **HTTP 요청을 받아서 적절한 Service를 호출하고,
> 그 결과를 HTTP 응답으로 돌려주는 계층**이다.
> 비즈니스 로직은 절대 없고, 교통정리만 한다.

### Controller의 책임 범위

```
✅ Controller가 해야 할 것
  - URL 매핑 (어떤 경로로 왔는지)
  - HTTP Method 구분 (GET / POST / PUT / DELETE)
  - 요청 데이터 추출 (@PathVariable, @RequestBody 등)
  - 입력값 유효성 검증 트리거 (@Valid)
  - Service 호출 후 결과를 ResponseEntity로 포장
  - 적절한 HTTP 상태 코드 반환

❌ Controller가 하면 안 되는 것
  - 비즈니스 규칙 판단
  - 직접 DB 접근 (Repository 직접 호출 금지)
  - 복잡한 데이터 가공 로직
```

### 요청 데이터 추출 어노테이션

```java
// @PathVariable — URL 경로에서 값 추출
// GET /api/users/42
@GetMapping("/{id}")
public UserResponseDto getUser(@PathVariable Long id) { ... }

// @RequestBody — HTTP Body의 JSON을 객체로 변환
@PostMapping
public UserResponseDto createUser(@RequestBody UserRequestDto requestDto) { ... }

// @RequestParam — URL 쿼리 파라미터 추출
// GET /api/users?keyword=kim&page=1
@GetMapping
public List<UserResponseDto> searchUsers(
        @RequestParam String keyword,
        @RequestParam(defaultValue = "0") int page) { ... }

// @Valid — Bean Validation 자동 트리거
@PostMapping
public UserResponseDto createUser(@Valid @RequestBody UserRequestDto requestDto) { ... }
```

### HTTP 상태 코드 규칙

| 작업 | Method | 성공 상태 코드 | 의미 |
|------|--------|--------------|------|
| 조회 | GET | 200 OK | 정상 반환 |
| 생성 | POST | 201 Created | 리소스 생성됨 |
| 수정 | PUT/PATCH | 200 OK | 수정 후 결과 반환 |
| 삭제 | DELETE | 204 No Content | 삭제됨, 반환값 없음 |



### 핵심 원칙

```
✅ Controller는 교통정리만 — 로직은 Service에게
✅ @Valid로 입력 검증, ResponseEntity로 상태 코드 제어
✅ @RestControllerAdvice로 예외를 한 곳에서 일관되게 처리
✅ REST 규칙에 맞는 HTTP 상태 코드를 반환한다
✅ 각 계층은 바로 아래 계층과만 대화한다
```

---

## 🎓 전체 최종 요약

```
┌──────────────────────────────────────────────────────────┐
│                     클라이언트 요청                        │
└─────────────────────────┬────────────────────────────────┘
                          │ HTTP
                          ▼
┌──────────────────────────────────────────────────────────┐
│  Controller   @RestController / @RequestMapping           │
│  역할: URL 매핑, 요청 추출, @Valid, ResponseEntity 반환    │
└─────────────────────────┬────────────────────────────────┘
                          │ DTO
                          ▼
┌──────────────────────────────────────────────────────────┐
│  Service      @Service / @Transactional                   │
│  역할: 비즈니스 규칙, 검증, 암호화, 예외 처리              │
└─────────────────────────┬────────────────────────────────┘
                          │ Entity
                          ▼
┌──────────────────────────────────────────────────────────┐
│  Repository   JpaRepository<Entity, ID>                   │
│  역할: save / findById / 쿼리 메서드 / @Query             │
└─────────────────────────┬────────────────────────────────┘
                          │ SQL
                          ▼
                      Database
```

### 한 문장 정리

```
Controller  →  "요청 받아서 넘겨주고, 결과 포장해서 돌려준다"
DTO         →  "필요한 것만 담아 안전하게 전달한다"
Service     →  "우리 서비스의 규칙대로 처리한다"
Repository  →  "DB에 묻고 답한다"
```
