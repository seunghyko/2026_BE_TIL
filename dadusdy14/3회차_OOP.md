# 3회차 TIL - OOP


## 상속 (Inheritance)

- 기존의 클래스를 재사용하여 새로운 클래스를 작성하는 것
  ```java
  class 자식 extends 부모 { }
  ```

**왜 상속을 쓰는가?**

`Student`, `Teacher` 클래스가 둘 다 `name`, `age`, getter/setter를 가지고 있다면 → 똑같은 코드가 두 군데에 존재  
→ 공통 부분을 `Person`(부모)에 한 번만 작성하고, `Student`와 `Teacher`가 상속받으면 중복 없이 사용 가능

**Is-a vs Has-a — 그럼 무조건 extends 쓰면 되는 건가?**

- **관계가 Is-a인 경우에만** 상속(extend)을 써야 함
- `Is-a` ("~은 ~이다")
  - 학생은 사람이다 → `Student extends Person`
- `Has-a` ("~은 ~을 가지고 있다")
  - 자동차는 엔진을 가지고 있다 → `Car` 안에 `Engine` 필드로 넣는 게 맞음

**상속의 핵심 규칙**

- Java는 **단일 상속**만 허용 
- 부모를 수정하면 자식 전체에 자동 반영
- 모든 클래스의 조상은 `Object`

---

## 오버라이딩 (Overriding)

- 부모한테 물려받은 메서드를 자식 입장에서 다르게 동작하게 하고 싶으면? → **오버라이딩**
- 부모 클래스의 메서드를 자식 클래스에서 재정의하는 것

```java
class Animal {
    void eat() { System.out.println("먹는다"); }
}

class Dog extends Animal {
    @Override
    void eat() { System.out.println("강아지가 사료를 먹는다"); }
}

class Cat extends Animal {
    @Override
    void eat() { System.out.println("고양이가 생선을 먹는다"); }
}
```

**오버라이딩 성립 조건** — 아래 3가지가 모두 같아야 함

- 메서드 이름
- 매개변수 (개수, 타입, 순서)
- 반환 타입

→ 이름·매개변수·반환타입이 같고 **내용만** 새로 작성하는 것

**`@Override` 어노테이션**

- 필수는 아니지만 붙이는 것을 권장
- **실수 방지**: 메서드 이름을 오타냈을 때 컴파일 에러로 바로 잡아줌
- **가독성**: 이 메서드가 오버라이딩임을 명시적으로 표시

---

## 오버라이딩 vs 오버로딩

- **오버라이딩** : 부모 메서드를 자식이 재정의
- **오버로딩** : 이름은 같고 매개변수가 다른 메서드를 추가

```java
class Animal { void eat() { } }

class Dog extends Animal {
    @Override
    void eat() { System.out.println("사료를 먹는다"); }  // 오버라이딩

    void eat(String food) { System.out.println(food + "를 먹는다"); }  // 오버로딩
    void eat(int count) { System.out.println(count + "번 먹는다"); }   // 오버로딩
}
```

**기억법**

- 오버라이딩(overriding) → 덮어쓰다 → 부모 것을 내가 덮어씀
- 오버로딩(overloading) → 짐을 싣다 → 같은 이름에 여러 버전을 실음

---

## 다형성 (Polymorphism)

**왜 등장했는가?**

- 동물 종류마다 메서드를 따로 만들면 동물이 늘어날수록 코드도 계속 늘어남
- `Animal`을 상속 + `feed()` 오버라이딩을 하면 메서드 하나로 모든 동물을 처리할 수 있음
- 즉, **같은 메서드 호출인데 어떤 객체냐에 따라 실행 결과가 달라지는 것**

**실제 코드 흐름**

```java
// 1. 부모 클래스
class Animal {
    void feed() { }
}

// 2. 자식 클래스들 - 각자 오버라이딩
class Rabbit extends Animal {
    @Override
    void feed() { System.out.println("토끼에게 풀을 준다"); }
}
class Seal extends Animal {
    @Override
    void feed() { System.out.println("물개에게 생선을 준다"); }
}

// 3. Animal 타입 하나로 모두 처리
class Zookeeper {
    void feedAnimal(Animal animal) { animal.feed(); }
}

// 4. 실행
Zookeeper keeper = new Zookeeper();
keeper.feedAnimal(new Rabbit());  // 토끼에게 풀을 준다
keeper.feedAnimal(new Seal());    // 물개에게 생선을 준다
```

**다형성의 장점**

- 새로운 동물이 추가돼도 `Zookeeper` 코드는 수정 불필요 → 확장에 유리

---

## 추상화 (Abstraction)

**왜 등장했는가?**

- `Animal` 클래스는 자식들에게 공통 구조를 제공하는 부모 역할인데, 정작 `Animal` 자체를 직접 만들어서 쓸 일은 없음
- `Animal animal = new Animal()` 이렇게 생성하는 걸 막고 싶음 → **추상 클래스**

**추상 클래스 (Abstract Class)**

- 공통 기능과 구조를 정의하는 클래스
- 직접 생성해서 사용 불가
- 상속을 통해 자식 클래스가 기능을 구현

```java
abstract class Animal {
    abstract void feed();
}

// Animal animal = new Animal();  // 컴파일 에러!
```

**추상 메서드 (Abstract Method)**

- 메서드 앞에 `abstract` 키워드를 붙임
- 실제 코드가 없음
- 상속받은 자식 클래스가 **반드시** 오버라이딩해서 구현해야 함
- 구현하지 않으면 컴파일 에러 발생

```java
abstract class Animal {
    abstract void feed();
    abstract void clean();
}

class Rabbit extends Animal {
    @Override
    void feed() { System.out.println("토끼에게 풀을 준다"); }

    @Override
    void clean() { System.out.println("토끼 우리를 건초로 청소한다"); }
}
```

---

## 인터페이스 (Interface)

**추상 클래스 → 인터페이스**

- 추상 클래스에서 모든 메서드가 추상 메서드이면 → **순수 추상 클래스**
- 이걸 더 간결하게 표현한 게 인터페이스

```java
// 순수 추상 클래스
abstract class Animal {
    abstract void feed();
    abstract void clean();
}

// 인터페이스로 표현
interface Animal {
    void feed();   // public abstract 생략 가능
    void clean();
}
```

**인터페이스 특징**

- 메서드가 모두 `public abstract` → 생략 가능 (생략 권장)
- `extends` 대신 `implements` 사용
- **다중 구현** 지원 (인터페이스는 여러 개 동시에 구현 가능)

```java
// 상속
class Rabbit extends Animal { }

// 인터페이스 구현
class Rabbit implements Animal {
    @Override
    public void feed() { System.out.println("토끼에게 풀을 준다"); }

    @Override
    public void clean() { System.out.println("토끼 우리를 건초로 청소한다"); }
}
```

**다중 구현**

```java
interface Animal { void feed(); void clean(); }
interface Playable { void play(); }

class Rabbit implements Animal, Playable {
    @Override
    public void feed() { ... }
    @Override
    public void clean() { ... }
    @Override
    public void play() { System.out.println("토끼가 공을 가지고 논다"); }
}
```

---

## 디폴트 메서드 (Default Method)

**왜 등장했는가?**

- 이미 사용 중인 인터페이스에 새로운 메서드를 추가하면 → 그 인터페이스를 구현한 모든 클래스에 오버라이딩을 추가해야 함
- 이 문제를 해결하기 위해 **인터페이스 안에서 기본 구현을 제공하는 메서드** 등장

```java
interface Animal {
    void feed();
    void clean();

    default void sleep() {
        System.out.println("동물이 잠을 잔다");  // 기본 구현 제공
    }
}
```

- `sleep()`을 추가해도 기존 `Rabbit`, `Seal`, `Snake` 클래스 수정 불필요
- 필요하면 자식 클래스에서 오버라이딩해서 재정의 가능

**디폴트 메서드 충돌**

- 여러 인터페이스에 같은 이름의 디폴트 메서드가 있으면 충돌 발생 → 직접 오버라이딩해서 해결

```java
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }

class Child implements A, B {
    @Override
    public void hello() { System.out.println("직접 정의"); }  // 직접 해결
}
```

---
---
---

## STEP3 코드 에러가 나는 이유

```java
for (Animal animal : animals) {
    keeper.feedAnimal(animal);
    keeper.cleanCage(animal);
    keeper.play((Playable) animal);  // 형변환 필요
}
```

1. animals 배열은 Animal 타입 
2. feedAnimal(animal), clean(animal)은 Animal에 있는 메서드 → 문제없음
3. play()는 Playable에 있는 메서드 → Animal 타입으로는 호출 불가
그래서 (Playable) animal로 형변환
4. 형변환은 해당 객체가 실제로 Playable을 implements한 경우에만 가능

---

## 컬렉션 프레임워크 (Collection Framework)

- 배열의 단점(크기 고정)을 해결

**List - ArrayList**

- 순서가 있는 목록
- 크기가 자동으로 늘어나고 줄어듦

```java
ArrayList<String> list = new ArrayList<>();

list.add("사과");        // 추가
list.get(0);             // 인덱스로 꺼내기
list.size();             // 크기
list.remove(0);          // 삭제
list.contains("사과");   // 포함 여부 확인
```

**Map - HashMap**

- 키-값 쌍으로 데이터를 저장 (파이썬의 딕셔너리와 동일)
- 키로 값을 꺼냄

```java
HashMap<String, String> map = new HashMap<>();

map.put("A", "010-1234-5678");  // 추가
map.get("A");                    // 키로 값 꺼내기
map.containsKey("A");           // 키 존재 여부
map.remove("A");                // 삭제
map.size();                          // 크기
```