# 4회차 TIL

## IoC / DI

이번 수업에서는 IoC, DI, Spring MVC에 대해 배웠다.

IoC와 DI는 갑자기 나온 새로운 개념이라기보다는, 이전에 배운 객체지향 개념을 스프링에서 더 잘 활용하기 위한 방식이다. 특히 다형성을 제대로 사용하려면 객체를 직접 생성하는 방식에서 벗어날 필요가 있다.

## 직접 객체를 생성하는 코드의 문제점

예시로 `OrderService`에서 결제 객체를 직접 생성하는 코드가 나왔다.

```java
public class OrderService {

    private final PaymentProcessor payment;

    public OrderService() {
        this.payment = new TossPayProcessor();
    }

    public void placeOrder(Order order) {
        payment.pay(order.getAmount());
        System.out.println("주문 완료");
    }
}
```

이 코드의 문제점은 `OrderService`가 결제 객체를 직접 만들고 있다는 것이다.

결제 수단을 토스페이에서 카카오페이로 바꾸려면 아래처럼 `OrderService` 코드를 직접 수정해야 한다.

```java
this.payment = new KakaoPayProcessor();
```

이렇게 되면 결제 구현체가 바뀔 때마다 서비스 코드가 같이 바뀐다.

또 테스트하기도 어렵다. 단위 테스트에서는 실제 결제 API를 호출하면 안 되기 때문에 `FakePayProcessor` 같은 가짜 객체를 넣고 싶은데, 생성자에서 이미 `TossPayProcessor`를 직접 만들고 있으면 외부에서 다른 객체를 넣을 수 없다.

결국 문제는 크게 네 가지로 정리할 수 있다.

- 결제 수단을 바꾸려면 서비스 코드를 수정해야 한다.
- 테스트용 가짜 객체를 넣기 어렵다.
- 주문 처리 책임과 결제 객체 생성 책임이 한 클래스에 섞여 있다.
- 작은 변경이 다른 코드까지 퍼질 수 있다.

핵심은 “사용하는 책임”과 “만드는 책임”을 분리해야 한다는 것이다.

## IoC

IoC는 Inversion of Control의 줄임말이고, 제어의 역전이라는 뜻이다.

여기서 제어는 객체를 생성하고, 호출하고, 관리하는 흐름을 말한다.

일반적인 코드에서는 필요한 객체를 직접 만든다.

```java
this.payment = new TossPayProcessor();
```

하지만 IoC에서는 객체를 직접 만들지 않고 외부에서 넣어준다.

```java
public class OrderService {

    private final PaymentProcessor payment;

    public OrderService(PaymentProcessor payment) {
        this.payment = payment;
    }
}
```

이렇게 하면 `OrderService`는 `TossPayProcessor`인지 `KakaoPayProcessor`인지 직접 알 필요가 없다. 그냥 `PaymentProcessor` 타입을 사용하면 된다.

순수 자바에서는 `main()` 같은 곳이 외부 역할을 할 수 있고, 스프링에서는 스프링 컨테이너가 외부 역할을 한다.

즉 IoC는 객체 생성과 연결의 권한을 내 코드가 아니라 외부 컨테이너에 맡기는 원칙이다.

## DI

DI는 Dependency Injection의 줄임말이고, 의존성 주입이라는 뜻이다.

의존성은 어떤 객체가 동작하기 위해 필요한 다른 객체를 말한다.

예를 들어 `OrderService`는 주문을 처리하기 위해 다음과 같은 객체가 필요할 수 있다.

```java
class OrderService {
    PaymentProcessor payment;
    EmailSender emailSender;
    OrderRepository orderRepository;
}
```

여기서 `PaymentProcessor`, `EmailSender`, `OrderRepository`는 `OrderService`의 의존성이다.

DI는 이런 의존 객체를 직접 만들지 않고 외부에서 넣어주는 방식이다.

IoC가 큰 원칙이라면, DI는 IoC를 구현하는 대표적인 방법이다.

## 의존성 주입 방법

의존성 주입 방법은 크게 세 가지가 있다.

### 1. 생성자 주입

생성자 주입은 객체를 만들 때 생성자를 통해 필요한 의존성을 넣어주는 방식이다.

```java
@Service
public class OrderService {

    private final PaymentProcessor payment;
    private final OrderRepository repo;

    public OrderService(PaymentProcessor payment, OrderRepository repo) {
        this.payment = payment;
        this.repo = repo;
    }
}
```

생성자 주입은 가장 권장되는 방식이다.

이유는 필요한 의존성이 생성자에 명확하게 드러나고, `final` 키워드를 사용할 수 있으며, 테스트할 때 mock 객체를 넣기 쉽기 때문이다.

### 2. Setter 주입

Setter 주입은 setter 메서드를 통해 의존성을 넣는 방식이다.

```java
@Autowired
public void setPayment(PaymentProcessor payment) {
    this.payment = payment;
}
```

Setter 주입은 선택적인 의존성을 표현할 때 사용할 수 있다.

다만 객체가 생성된 직후부터 setter가 호출되기 전까지는 필드가 `null`일 수 있다. 그래서 필수 의존성에는 생성자 주입이 더 적절하다.

### 3. 필드 주입

필드 주입은 필드에 바로 `@Autowired`를 붙이는 방식이다.

```java
@Autowired
private PaymentProcessor payment;
```

코드는 짧지만 의존성이 잘 드러나지 않고, 테스트하기 어렵다. 그래서 권장되는 방식은 아니다.

결론적으로 필수 의존성은 생성자 주입을 사용하는 것이 좋다.

## Spring IoC 컨테이너

스프링에서 IoC 컨테이너 역할을 하는 대표적인 객체는 `ApplicationContext`이다.

스프링 컨테이너는 설정 정보와 어노테이션을 읽고 객체를 생성한다. 그리고 필요한 객체들을 서로 연결해서 사용할 수 있는 상태로 만든다.

흐름은 대략 다음과 같다.

```text
설정 / 어노테이션 확인
-> 객체 생성
-> 의존성 주입
-> 애플리케이션에서 사용
```

## Bean

스프링이 만들고 관리하는 객체를 Bean이라고 한다.

직접 `new`로 만든 객체는 일반 객체이고, 스프링 컨테이너가 생성하고 관리하는 객체는 Bean이다.

Bean으로 등록할 때 사용하는 어노테이션은 다음과 같다.

- `@Component`: 일반적인 스프링 컴포넌트
- `@Service`: 비즈니스 로직을 담당하는 클래스
- `@Repository`: 데이터 접근을 담당하는 클래스
- `@Controller`: 웹 요청을 처리하는 클래스
- `@RestController`: REST API 요청을 처리하는 클래스

인터페이스와 추상 클래스는 직접 객체를 만들 수 없기 때문에 보통 구현 클래스에 어노테이션을 붙인다.

## 어노테이션

어노테이션은 클래스나 메서드 위에 붙이는 메모 또는 라벨 같은 것이다.

예시는 다음과 같다.

```java
@Service
public class OrderService {
}

@Autowired
private PaymentProcessor payment;

@Override
public void pay(int amount) {
}
```

어노테이션은 단순한 주석이 아니라 스프링이 읽고 동작을 결정하는 정보로 사용된다.

예를 들어 `@Service`가 붙은 클래스는 스프링 Bean으로 등록될 수 있고, `@GetMapping`이 붙은 메서드는 특정 HTTP 요청과 연결된다.

## Spring MVC

Spring MVC는 웹 요청을 역할별로 나누어 처리하는 구조이다.

MVC는 Model, View, Controller의 줄임말이다.

- Model: 데이터 또는 처리 결과
- View: 사용자에게 보여줄 화면
- Controller: 요청을 받고 처리 흐름을 결정하는 역할

수업에서는 Controller를 주방장, View를 접시, Model을 재료와 요리에 비유했다.

Spring MVC를 사용하면 HTTP 요청을 직접 하나하나 처리하지 않아도 된다.

스프링 없이 `/hello` 요청을 처리하려면 `ServerSocket`을 열고 요청 문자열을 직접 읽은 다음, 응답도 직접 작성해야 한다.

```java
ServerSocket server = new ServerSocket(8080);
Socket client = server.accept();

String request = in.readLine();

if (request.contains("/hello")) {
    out.println("HTTP/1.1 200 OK");
    out.println();
    out.println("Hello!");
}
```

하지만 스프링에서는 아래처럼 작성할 수 있다.

```java
@GetMapping("/hello")
public String hello() {
    return "Hello!";
}
```

스프링이 요청 경로와 실행할 메서드를 연결해주기 때문이다.

## DispatcherServlet

DispatcherServlet은 Spring MVC에서 모든 요청을 가장 먼저 받는 객체이다.

브라우저에서 `/users`, `/posts` 같은 요청이 들어오면 DispatcherServlet이 먼저 요청을 받는다.

DispatcherServlet은 프론트 컨트롤러 역할을 한다. 즉, 요청을 직접 처리하기보다는 어떤 컨트롤러로 보낼지 결정하는 입구 역할을 한다.

요청 흐름은 대략 다음과 같다.

```text
Client 요청
-> DispatcherServlet
-> HandlerMapping
-> Controller
-> Service
-> Repository
-> DB
```

## HandlerMapping

DispatcherServlet은 요청을 받은 뒤 어느 컨트롤러로 보내야 하는지 알아야 한다.

이때 사용하는 것이 HandlerMapping이다.

HandlerMapping은 요청 URL과 컨트롤러 메서드를 매핑해준다.

예를 들어 다음 코드가 있다고 하면,

```java
@RestController
public class PostController {

    @GetMapping("/posts")
    public String getPosts() {
        return "게시글 목록";
    }
}
```

`GET /posts` 요청이 들어왔을 때 `PostController`의 `getPosts()` 메서드를 실행해야 한다는 정보를 HandlerMapping이 관리한다.

즉 `@GetMapping("/posts")`를 보고 요청 경로와 메서드를 연결하는 역할을 한다.

## Controller, Service, Repository, DTO

Controller가 요청을 받지만 모든 일을 Controller에서 처리하지는 않는다.

보통 백엔드 코드는 역할을 나누어서 작성한다.

```text
Client 요청
-> Controller
-> Service
-> Repository
-> DB
```

Controller는 HTTP 요청과 응답을 담당한다.

Service는 실제 비즈니스 로직을 담당한다.

Repository는 DB 접근을 담당한다.

DTO는 Data Transfer Object의 줄임말이다.

계층 간 데이터를 주고받을 때 사용하는 객체이다. 예를 들어 Controller가 요청 데이터를 받을 때나, Service에서 처리한 결과를 응답으로 돌려줄 때 DTO를 사용할 수 있다.

Entity를 그대로 외부에 노출하지 않고, 필요한 데이터만 담아서 전달하기 위해 사용한다.

예시는 다음과 같다.

```java
public class PostResponseDto {
    private Long id;
    private String title;
    private String content;
}
```

이렇게 역할을 나누면 코드의 책임이 분리되고, 변경이 생겼을 때 수정 범위를 줄일 수 있다.

## HTTP / HTTPS

HTTP는 HyperText Transfer Protocol의 줄임말이다.

서로 다른 시스템이 데이터를 주고받기 위한 기본적인 통신 규칙이다.

HTTP와 HTTPS의 차이는 데이터 암호화 여부이다.

- HTTP: 데이터를 그대로 전달한다.
- HTTPS: 데이터를 암호화해서 전달한다.

요청을 보내고 응답을 받는 방식 자체는 비슷하지만, HTTPS는 데이터를 암호화하기 때문에 더 안전하다.
