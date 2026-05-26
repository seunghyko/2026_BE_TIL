# TIL - Spring MVC / IoC / DI

## 📅 날짜

2026-05-24

---

# 오늘 배운 내용

오늘은 IoC(제어의 역전)과 DI(의존성 주입), 그리고 MVC 패턴에 대해 학습했다. 또 추가적으로 Controller와 Service, Repository, DTO에 대해 따로 공부하였다.

---

# 1. IoC (제어의 역전)

## 📌 개념

기존에는 객체를 개발자가 직접 생성하고 관리했지만, Spring에서는 객체의 생성과 관리 권한을 Spring 컨테이너가 담당한다.

즉, 객체의 생성(Create), 호출(Call), 소멸(Destroy)에 대한 제어권이 개발자에게서 Spring으로 넘어가는 것을 IoC라고 한다.

## 📌 특징

- 객체 생성 관리 자동화
- 코드 간 결합도 감소
- 유지보수성 증가
- 객체 재사용성 향상

## 📌 기존 방식

```java
MemberService service = new MemberService();
```

개발자가 직접 객체를 생성한다.

## 📌 Spring 방식

```java
@Autowired
private MemberService service;
```

Spring이 객체를 생성하고 주입해준다.

---

# 2. DI (Dependency Injection)

## 📌 개념

DI는 필요한 객체(의존성)를 직접 생성하지 않고 외부(Spring 컨테이너)에서 주입받는 방식이다.

즉, 객체 간 의존성을 Spring이 대신 연결해준다.

## 📌 장점

- 객체 간 결합도 감소
- 테스트 용이
- 유지보수 편리
- 코드 확장성 증가

## 📌 예시

```java
@Service
public class MemberService {
}
```

```java
@Controller
public class MemberController {

    @Autowired
    private MemberService memberService;
}
```

Controller가 Service 객체를 직접 생성하지 않고 Spring이 자동으로 주입한다.

---

# 3. MVC 패턴 ⭐

오늘 학습에서 가장 중요했던 개념은 MVC 패턴이다.

MVC 패턴은 역할을 분리하여 유지보수성과 확장성을 높이는 웹 개발 구조이다.

---

## 📌 MVC 구조

### 1) Model

- 데이터와 비즈니스 로직 담당
- DB 처리 및 데이터 관리
- Service, Repository 등이 포함됨

### 2) View

- 사용자에게 보여지는 화면 담당
- HTML, JSP, Thymeleaf 등이 사용됨

### 3) Controller

- 사용자의 요청을 처리
- Model과 View를 연결
- 흐름 제어 담당

---

# 📌 MVC 흐름

```text
사용자 요청
   ↓
Controller
   ↓
Service
   ↓
Repository(DB)
   ↓
결과 반환
   ↓
View 출력
```

---

# 📌 MVC 패턴이 중요한 이유

## ✅ 역할 분리

각 기능이 분리되어 코드 관리가 쉬워진다.

## ✅ 유지보수 용이

화면 수정(View)과 로직 수정(Model)을 독립적으로 진행할 수 있다.

## ✅ 협업 효율 증가

프론트엔드와 백엔드 작업을 나누기 쉽다.

## ✅ 확장성 증가

기능 추가 및 수정이 편리하다.

---

# 4. Spring MVC 예제 코드

## 📌 Controller

```java
@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";
    }
}
```

## 📌 View

```html
<h1>Hello Spring MVC</h1>
```

---

# 5. Spring 계층 구조 추가 학습

Spring MVC에서는 역할을 명확하게 나누기 위해 계층 구조를 사용한다.

대표적으로 다음과 같은 구조를 사용한다.

```text
Controller → Service → Repository → DB
```

그리고 데이터를 전달하기 위해 DTO를 함께 사용한다.

---

# 📌 Controller

## ✅ 역할

- 사용자의 요청(Request)을 받는 역할
- URL 요청 처리
- Service 호출
- 결과를 View로 전달

즉, 사용자의 요청과 응답을 담당하는 계층이다.

## ✅ 특징

- 비즈니스 로직을 직접 처리하지 않음
- 요청 흐름 제어 담당
- Service에 작업을 위임

## ✅ 예시

```java
@Controller
public class MemberController {

    @Autowired
    private MemberService memberService;

    @GetMapping("/member")
    public String memberPage() {
        memberService.findMember();
        return "member";
    }
}
```

---

# 📌 Service

## ✅ 역할

- 실제 비즈니스 로직 처리
- Controller와 Repository 사이 연결
- 데이터 가공 및 처리 수행

즉, 프로그램의 핵심 기능을 담당한다.

## ✅ 특징

- 로직 중심 계층
- 여러 Repository를 함께 사용할 수 있음
- 유지보수성과 재사용성이 높아짐

## ✅ 예시

```java
@Service
public class MemberService {

    @Autowired
    private MemberRepository memberRepository;

    public void findMember() {
        memberRepository.findById();
    }
}
```

---

# 📌 Repository

## ✅ 역할

- 데이터베이스 접근 담당
- SQL 실행 및 데이터 저장/조회
- DB와 직접 연결되는 계층

즉, 데이터 관리 전용 계층이다.

## ✅ 특징

- CRUD 작업 수행
- 데이터 영속성 관리
- DB 처리 코드 분리 가능

## ✅ 예시

```java
@Repository
public class MemberRepository {

    public void findById() {
        System.out.println("DB 조회");
    }
}
```

---

# 📌 DTO (Data Transfer Object)

## ✅ 역할

- 계층 간 데이터 전달 객체
- 데이터를 담아서 이동시키는 용도

즉, 데이터를 안전하게 전달하기 위한 객체이다.

## ✅ DTO를 사용하는 이유

- 필요한 데이터만 전달 가능
- 엔티티 직접 노출 방지
- 보안성 증가
- 계층 간 결합도 감소

## ✅ 예시

```java
public class MemberDTO {

    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

---

# 📌 전체 흐름 정리

```text
사용자 요청
   ↓
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

- Controller : 요청 처리
- Service : 비즈니스 로직 처리
- Repository : DB 처리
- DTO : 데이터 전달

---

# 💡 느낀 점

Spring은 객체를 직접 관리하지 않고 Spring 컨테이너가 대신 관리한다는 점이 인상 깊었다. 또한 MVC 패턴을 통해 역할을 분리하면 코드가 훨씬 깔끔해지고 유지보수가 쉬워진다는 점을 이해할 수 있었다.

특히 Controller, Service, Repository의 역할을 분리하는 구조가 실제 웹 개발에서 매우 중요하다는 것을 배웠다.

앞으로는 Spring MVC 구조를 기반으로 프로젝트를 직접 만들어보며 익숙해질 필요가 있다고 느꼈다.

또한 DTO를 사용하면 필요한 데이터만 전달할 수 있어 보안성과 효율성을 높일 수 있다는 점도 배웠다.
