# 🦁 멋쟁이 사자 1회차 - Spring 입문 정리

> 범위: ~p.88 (Annotation 직전까지)
> 키워드: Spring 등장 배경 / 구조 / IoC / DI / Bean & Container

---

## 1. 왜 Spring이 생겼을까?

### 🕰️ Spring 이전 (EJB 시대)의 문제점
Spring이 등장하기 전, Java 진영에서는 **EJB(Enterprise JavaBeans)** 가 주류였어. 그런데 이게 너무 무겁고 불편했음.

| 문제점 | 설명 |
|---|---|
| **무겁다 (Heavy)** | EJB 컨테이너 자체가 무거워서 실행/테스트가 느림 |
| **복잡하다** | 비즈니스 로직보다 EJB 규약을 맞추는 코드가 더 많음 |
| **테스트 어려움** | EJB 컨테이너 없이는 단위 테스트 자체가 거의 불가능 |
| **특정 기술에 종속** | EJB 인터페이스를 강제로 구현해야 해서 갈아타기 힘듦 |
| **강한 결합도** | 객체끼리 `new`로 직접 생성해서 강하게 묶여있음 |

### 🌱 그래서 Spring은?
- **POJO(Plain Old Java Object)** 기반 → 평범한 자바 객체로 개발 가능
- **가볍다** → 컨테이너가 가벼워 빠른 실행/테스트
- **낮은 결합도** → IoC/DI로 객체 간 의존성을 느슨하게
- **테스트 친화적** → 객체를 갈아끼우기 쉬워서 단위 테스트가 쉬움

> 💡 한 줄 요약: **"무거운 EJB를 대체하기 위해, 가볍고 객체지향다운 개발을 하려고 만들어진 프레임워크"**

---

## 2. Spring의 구조 (Architecture)

Spring은 모듈형 구조라서 필요한 것만 골라 쓸 수 있어.

```
┌─────────────────────────────────────────────┐
│         Spring Framework 구조               │
├─────────────────────────────────────────────┤
│  Test                                       │
├─────────────────────────────────────────────┤
│  Web (Web MVC, WebSocket, ...)              │
├─────────────────────────────────────────────┤
│  Data Access / Integration                  │
│  (JDBC, ORM, Transactions ...)              │
├─────────────────────────────────────────────┤
│  AOP  │  Aspects  │  Instrumentation        │
├─────────────────────────────────────────────┤
│  Core Container                             │
│  (Beans, Core, Context, SpEL)  ← 핵심!     │
└─────────────────────────────────────────────┘
```

### 핵심은 **Core Container**
- **Beans, Core, Context, SpEL** 이 네 개가 Spring의 심장
- 이 안에 우리가 배운 **IoC / DI / Bean / ApplicationContext** 가 다 들어있음

---

## 3. IoC (Inversion of Control, 제어의 역전)

### 🤔 기존 방식: 내가 직접 객체를 만든다
```java
public class OrderService {
    // 내가 직접 new!
    private MemberRepository repo = new MemberRepository();
}
```
→ `OrderService`가 `MemberRepository`를 **직접 생성하고 제어**함
→ `MemberRepository`를 다른 걸로 바꾸려면 코드를 직접 고쳐야 함 (강한 결합)

### ✅ IoC 방식: 객체 생성/관리를 Spring에게 맡긴다
```java
public class OrderService {
    private MemberRepository repo;
    
    // 내가 만들지 않아! 외부(Spring)가 넣어줌
    public OrderService(MemberRepository repo) {
        this.repo = repo;
    }
}
```
→ 객체를 **누가, 언제 만들지에 대한 제어권**이 개발자가 아닌 **Spring 컨테이너**에게 있음
→ 이걸 "**제어의 역전(Inversion of Control)**" 이라고 부름

### 📌 핵심
> "객체의 생성과 생명주기를 개발자가 아니라 **프레임워크(Spring)** 가 관리한다"

---

## 4. DI (Dependency Injection, 의존성 주입)

IoC를 **구현하는 구체적인 방법**이 바로 DI야.

### 🔗 Dependency(의존성)란?
> A 클래스가 B 클래스를 사용하면, **A는 B에 의존한다**고 말함.

```java
class OrderService {
    MemberRepository repo;  // OrderService는 MemberRepository에 "의존"
}
```

### 💉 Injection(주입)이란?
> 그 의존 객체를 **외부에서 넣어주는 것**

### ❌ 강한 결합 (Tight Coupling)
```java
class OrderService {
    MemberRepository repo = new MysqlMemberRepository(); // DB 바꾸려면 코드 수정!
}
```

### ✅ 느슨한 결합 (Loose Coupling) + DI
```java
class OrderService {
    MemberRepository repo;  // 인터페이스 타입
    
    public OrderService(MemberRepository repo) { // 외부에서 주입받음
        this.repo = repo;
    }
}
```
→ Mysql이든 Oracle이든 H2든 **갈아끼우기만 하면 됨** → 코드 수정 X

### DI의 3가지 방식
| 방식 | 예시 | 특징 |
|---|---|---|
| **생성자 주입** | `public OrderService(Repo r)` | ⭐ 권장. 불변성 보장 |
| **Setter 주입** | `public void setRepo(Repo r)` | 선택적 의존성에 사용 |
| **필드 주입** | `@Autowired Repo repo;` | 간단하지만 비권장 (테스트 어려움) |

---

## 5. Bean과 Spring Container

### 🫘 Bean이란?
> **Spring 컨테이너가 관리하는 객체**
> 우리가 `new`로 만든 객체가 아니라, **Spring이 대신 만들어서 관리**해주는 객체

### 📦 Spring Container란?
> Bean을 **생성하고, 관리하고, 의존성을 주입해주는 공간**
> 대표적으로 `ApplicationContext`가 있음

### 동작 흐름
```
   [개발자]                        [Spring Container]
       │                                  │
       │  "이거 Bean으로 등록해줘"          │
       ├─────────────────────────────────▶│
       │  (@Component / @Bean / XML)      │  Bean 생성
       │                                  │  ↓
       │                                  │  의존성 주입(DI)
       │                                  │  ↓
       │   "OrderService 줘!"             │  Bean 보관
       │◀─────────────────────────────────┤
       │   완성된 객체 받음                │
```

### 컨테이너 종류
- **BeanFactory**: 가장 기본. Bean 관리 기본 기능 제공
- **ApplicationContext**: BeanFactory + 부가 기능 (메시지, 이벤트, AOP 등) ⭐ 실무에서 주로 사용

---

## 6. 전체 그림으로 한 번에 정리

```
                    ┌────────────────────────┐
                    │   Spring Container      │
                    │  (ApplicationContext)   │
                    │                         │
                    │   ┌──────┐  ┌──────┐   │
                    │   │ Bean │  │ Bean │   │
                    │   └──┬───┘  └──┬───┘   │
                    │      │ 의존성 주입│      │
                    │      ▼         ▼      │
                    │   ┌──────────────┐    │
                    │   │     Bean      │    │
                    │   └──────────────┘    │
                    └────────────────────────┘
                              │
                              │ 객체 요청
                              ▼
                       [ 개발자 코드 ]

  ★ IoC: 객체 제어권이 개발자 → 컨테이너로 넘어감
  ★ DI : 그 제어를 "주입" 방식으로 실현
  ★ Bean: 컨테이너가 관리하는 객체
```

---

## 7. 핵심 용어 한 줄 정리 (시험 직전 보기용 🔥)

| 용어 | 한 줄 정의 |
|---|---|
| **POJO** | 특정 기술에 종속되지 않은 평범한 자바 객체 |
| **IoC** | 객체 제어권이 개발자 → 프레임워크로 넘어가는 것 |
| **DI** | IoC를 실현하는 방법. 외부에서 의존 객체를 주입 |
| **Dependency** | A가 B를 사용할 때 A는 B에 의존함 |
| **Bean** | Spring 컨테이너가 관리하는 객체 |
| **Spring Container** | Bean을 생성/관리/주입하는 공간 |
| **ApplicationContext** | 실무에서 쓰는 대표 Spring 컨테이너 |
| **결합도** | 클래스끼리 얼마나 강하게 묶여있는지 (낮을수록 좋음) |

---

## 8. 다음 회차 예고
- **Annotation** (`@Component`, `@Autowired`, `@Bean`, `@Configuration` ...)
- XML 설정 → 어노테이션 기반 설정으로 진화한 이유

---

> 📚 **한 줄 회고**
> Spring은 결국 "**객체를 잘 관리해주는 거대한 상자(Container)**" 이고,
> 그 상자를 잘 쓰기 위한 원칙이 **IoC**, 그 원칙을 실현하는 방법이 **DI** 다!
