# [TIL] JAVA OOP (상속/다형성/인터페이스/추상화)

## 🗓 학습 날짜
- 2026년 05월 07일

## 📚 공부한 내용

### 1. 상속 (Inheritance)
기존 클래스(부모)를 재사용하여 새로운 클래스(자식)를 작성하는 문법입니다.
- **핵심 규칙**: 자바는 단일 상속만 허용하며, 자식은 부모보다 항상 멤버가 많거나 같습니다.
- **관계**: `Is-a` 관계가 성립할 때 주로 사용합니다 (예: 학생은 사람이다).


```java
// 부모 클래스
class Person {
    private String name;
    private int age;

    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

// 자식 클래스 (Person 상속)
class Student extends Person {
    private String major; // 학생 전용 기능 확장

    public String getMajor() { return major; }
    public void setMajor(String major) { this.major = major; }
}

```

### 2. 오버라이딩(Overriding) vs 오버로딩(Overloading)

부모에게 물려받은 메서드를 자식 입장에서 다르게 동작하게 하고 싶을 때 사용하는 것이 **오버라이딩**입니다.

* **오버라이딩 (Overriding)**: 부모 메서드를 자식이 재정의하는 것. 메서드 이름, 매개변수, 반환 타입이 모두 같아야 합니다. `@Override` 어노테이션을 붙이면 컴파일러가 실수를 잡아주어 안전합니다.


* **오버로딩 (Overloading)**: 같은 이름의 메서드를 매개변수(개수, 타입, 순서)만 다르게 하여 여러 개 만드는 것.



```java
class Animal {
    void eat() { System.out.println("먹는다"); }
}

class Dog extends Animal {
    // 오버라이딩: 부모의 기능을 덮어씀
    @Override
    void eat() { System.out.println("강아지가 사료를 먹는다"); }
    
    // 오버로딩: 같은 이름, 다른 매개변수
    void eat(String food) { System.out.println(food + "를 먹는다"); }
}

```

### 3. 다형성 (Polymorphism)

같은 메서드를 호출하더라도, 실제 참조하는 객체가 무엇이냐에 따라 실행 결과가 달라지는 것을 의미합니다.

* 다형성이 성립하려면 **상속**과 **오버라이딩**이 전제되어야 합니다. => 부모 이름표로 묶기 위해


* 부모 타입의 배열 등을 사용하면 여러 자식 객체들을 한 번에 관리할 수 있어 코드가 매우 간결해집니다.



```java
Animal[] animals = { new Rabbit(), new Seal(), new Snake() };
Zookeeper keeper = new Zookeeper();

// 다형성을 활용하여 한 번의 반복문으로 각 동물에 맞는 먹이를 줄 수 있음
for (Animal animal : animals) {
    keeper.feedAnimal(animal);
}

```

### 4. 추상화와 추상 클래스 (Abstract Class)

여러 클래스들의 공통점을 뽑아내어 구조를 짤 때 사용합니다.

* `abstract` 키워드를 사용하며, 직접 객체(`new`)로 생성할 수 없습니다. 


* **추상 메서드**: 메서드 바디(`{}`)가 없는 메서드로, 상속받은 자식 클래스에서 **반드시 오버라이딩**하여 구현해야 합니다 (구현하지 않으면 컴파일 에러 발생). => 오버라이딩을 강제화



```java
abstract class Animal {
    // 자식이 무조건 구현해야 하는 추상 메서드
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

### 5. 인터페이스 (Interface)

인터페이스는 다중 구현(다중 상속)을 지원하는 순수 추상 클래스의 형태입니다.

* 모든 메서드는 기본적으로 `public abstract`입니다 (생략 권장).

* 자격증(Can-do)

* `extends` 대신 `implements` 키워드를 사용합니다.


* **디폴트 메서드 (Default Method)**: 인터페이스 내에서 `default` 키워드를 사용해 기본 구현을 제공할 수 있습니다. 기존 구현 클래스들을 수정하지 않고도 새로운 기능을 확장할 수 있습니다. => 기본이 abstract인데 인터페이스 안에서도 완성된 메서드를 만들 수 있게 해줌.



```java
interface Animal {
    void feed();
    void clean();
}

interface Playable {
    void play();
}

// 다중 구현 가능
class Rabbit implements Animal, Playable {
    @Override 
    public void feed() { System.out.println("토끼에게 풀을 준다"); }
    
    @Override 
    public void clean() { System.out.println("토끼 우리를 건초로 청소한다"); }
    
    @Override 
    public void play() { System.out.println("토끼가 공을 가지고 논다"); }
}

```

---

##  (TIL 추가 과제)

### Q1. 실습2 STEP3에서 코드 에러가 나는 이유는 무엇인가?

#### 🤔 나의 고민과 접근 방식
`Main`에서 `Animal[]` 배열을 반복문으로 돌리며 `Zookeeper`의 메서드를 호출하려고 했다. 하지만 매개변수로 전달된 객체는 `Animal` 타입으로 인식되기 때문에, `Animal` 설계도에 없는 `play()` 메서드를 호출하려고 하면 컴파일 에러가 발생한다. 
따라서 `Animal` 객체가 `play()` 기능을 사용하게 하려면, 잠시 `Playable` 타입으로 형변환(Casting)을 해줘야 한다고 생각했다.

#### 💡 해결 과정 및 깨달은 점
1. **올바른 형변환 문법:**
   - `animal.(Playable)play()` ❌ (문법 오류)
   - `((Playable) animal).play();` ⭕ (괄호를 이용해 형변환을 우선순위로 처리해야 함)



### Q2. Java의 컬렉션 프레임워크 (List, Map)
#### 1. Map 인터페이스
**"사전(Dictionary)과 같은 자료구조"**
- `Key`와 `Value`가 한 쌍으로 저장되는 형태입니다.
- 단어를 찾을 때 사전을 펼치듯, `Key`를 주면 즉시 `Value`를 찾을 수 있어 데이터 검색에 매우 강력합니다.
- **특징:** `Key`는 중복될 수 없지만, `Value`는 중복이 가능합니다.

| 종류 | 특징 | 핵심 요약 |
| :--- | :--- | :--- |
| **HashMap** | 순서 보장 X, 정렬 X | **압도적인 속도.** 실무에서 Map을 쓴다고 하면 90% 이상은 이것을 사용함. |
| **TreeMap** | Key를 기준으로 **자동 정렬** (가나다, 숫자 순) | 정렬이 필요할 때 유용하지만, 데이터를 넣고 뺄 때마다 정렬하므로 속도가 상대적으로 느림. |
| **LinkedHashMap** | 데이터가 입력된 **순서를 기억함** | 들어온 순서대로 데이터를 꺼내야 할 때 유용함. |

---

#### 2. List 인터페이스
**"순서표를 뽑고 줄 서 있는 데이터들의 모임"**
- 데이터가 들어온 순서(Index)를 철저하게 유지합니다.
- 중복된 데이터를 허용합니다. (예: 대기번호 1번도 '고ㅁ현', 3번도 '고ㅁ현' 가능)

| 종류 | 특징 | 핵심 요약 |
| :--- | :--- | :--- |
| **ArrayList** | 배열처럼 Index 번호표를 가지고 있음 | **데이터 조회(읽기)가 매우 빠름.** 하지만 중간에 누군가 새치기(삽입)하거나 빠지면(삭제), 뒤로 다 밀거나 당겨야 해서 수정 작업엔 매우 느림. (가장 흔하게 쓰임) |
| **LinkedList** | 기차 칸처럼 데이터들이 끈으로 연결되어 있음 | **데이터 중간 삽입/삭제가 매우 빠름.** (연결된 끈만 끊고 이어주면 끝). 하지만 100번째 데이터를 찾으려면 1번부터 끈을 타고 넘어가야 해서 조회 속도는 느림. |
---
