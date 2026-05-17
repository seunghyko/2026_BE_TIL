# TIL: Java OOP - 클래스와 캡슐화


## 절차 지향 vs 객체 지향

- **절차 지향**: 코드가 위에서 아래로 순서대로 실행되는 방식 (실행 순서 중심)
- **객체 지향**: 관련된 데이터와 기능을 하나로 묶어서 객체로 관리하는 방식 (객체 중심)
    - **상태(State)**: 지금 어떤 상태인가 → 객체가 가진 데이터
    - **기능(Methods)**: 무엇을 할 수 있는가 → 객체가 할 수 있는 행동
        - **오버로딩**: 같은 이름의 메서드를 파라미터만 다르게 여러 개 만들 수 있다
            ```java
            void setVolume(int volume) { }
            void setVolume(int volume, boolean mute) { }
            ```
    - **생성자**: 객체를 만들 때 초기값을 설정할 수 있다
        ```java
        // MusicPlayer.java
        class MusicPlayer {
            private int volume;

            MusicPlayer(int volume) {
                this.volume = volume;
            }
        }

        // MusicPlayerMain.java
        MusicPlayer player = new MusicPlayer(10);
        ```

> **Q. 함수 vs 메서드 차이가 뭐야?**
> - **함수**: 독립적으로 존재
> - **메서드**: 클래스 안에 속해있는 함수
> - Java에서는 모든 코드가 클래스 안에 있기 때문에 **메서드만 존재**

---

## 절차 지향의 문제점

- 기능이 늘어날수록 수정할 곳이 계속 늘어난다
- 음악 플레이어가 N개면 변수도 N배로 늘어난다

---

## 절차 지향 → 객체 지향 리팩토링

**절차 지향**

```java
int volume = 0;
boolean isOn = false;

isOn = true;
volume += 1;
volume -= 1;
isOn = false;
```

**객체 지향**

```java
class MusicPlayer {
    int volume = 0;
    boolean isOn = false;

    void on() { isOn = true; }
    void off() { isOn = false; }
    void volumeUp() { volume++; }
    void volumeDown() { volume--; }
}
```

---

## OOP의 4대 특성

- **캡슐화**: 데이터를 직접 건드리지 못하게 하고, 정해진 방법으로만 사용하도록
- **상속**: 기존 클래스의 기능을 물려받아서 재사용하는 것
- **추상화**: 복잡한 내부는 숨기고, 필요한 기능만 쉽게 사용할 수 있게 하는 것
- **다형성**: 같은 이름의 기능을 상황에 따라 다르게 동작하게 하는 것

---

## 캡슐화 (Encapsulation)

> 변수는 private으로 숨기고, 메서드를 통해서만 변경한다

**MusicPlayer.java**

```java
class MusicPlayer {
    private int volume = 0;

    void volumeUp() {
        volume++;
    }
}
```

**MusicPlayerMain.java**

```java
MusicPlayer player = new MusicPlayer();
player.volumeUp();      // ✅
player.volume = 99999;  // ❌
```

> **Q. private으로 선언하면 적용 범위가 어떻게 돼?**
> - **클래스 안 변수** (`private int volume`) → 클래스 안 어디서든 접근 가능, 외부에서만 막힘
> - **메서드 안 변수** (`int volume`) → 그 메서드 안에서만 존재

---

## Getter / Setter

**Getter**: `private` 변수의 값을 외부에서 읽을 때 사용

```java
// MusicPlayer.java
int getVolume() {
    return this.volume;
}

// MusicPlayerMain.java
int v = player.getVolume();  // ✅
```

**Setter**: `private` 변수의 값을 외부에서 변경할 때 사용

```java
// MusicPlayer.java
void setVolume(int volume) {
    this.volume = volume;
}

// MusicPlayerMain.java
player.setVolume(10);  // ✅
```

> **Q. `this`가 뭐야?**
> 클래스 안 변수와 메서드에서 전달받은 값의 이름이 같을 때 구분하기 위해 사용
> - `this.volume` → 클래스 안 변수
> - `volume` → 전달받은 값

---

## 접근 제어자

캡슐화를 하려면 `private`을 써야 하는데, 그럼 어디까지 막을 수 있는 걸까?

| 접근 제어자 | 접근 범위 | 비고 |
|---|---|---|
| `private` | 같은 클래스 안에서만 | |
| `default` | 같은 패키지 안에서 | 아무것도 안 쓰면 자동 적용 |
| `protected` | 같은 패키지 + 하위 클래스 | |
| `public` | 어디서든 | |