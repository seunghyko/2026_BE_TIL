# [TIL] JAVA OOP (클래스와 캡슐화)

**멋쟁이사자처럼 제주대학교 14기 백엔드 2회차 세션 복습**
*주제: 절차 지향 프로그래밍에서 객체 지향 프로그래밍으로의 전환, 그리고 캡슐화*

---

## 1. 절차 지향 vs 객체 지향 프로그래밍

### 절차 지향 프로그래밍 (Procedural Programming)
* 실행 순서를 중요하게 생각하는 방식이다.
* 프로그램의 흐름을 순서대로 따르고 처리하며, **"어떻게"** 동작하는지가 중심이 된다.

### 객체 지향 프로그래밍 (Object-Oriented Programming, OOP)
* 객체를 중요하게 생각하는 방식이다.
* 실제 세계의 사물이나 사건을 보고 객체들 간의 상호작용을 중심으로 프로그래밍하며, **"무엇을"** 할 것인가가 중심이 된다.

---

## 2. 음악 플레이어로 보는 패러다임의 전환

**요구사항:** 음악 플레이어를 켜고 끄기, 볼륨 증가/감소, 상태 확인 기능 구현

### 🚨 절차 지향 방식의 한계
처음에는 변수와 실행 흐름 위주로 코드를 작성한다.

```java
public class MusicPlayerMain1 {
    public static void main(String[] args) {
        int volume = 0;
        boolean isOn = false;

        // 음악 플레이어 켜기
        isOn = true;
        System.out.println("음악 플레이어를 시작합니다.");

        // 볼륨 증가
        volume += 1;
        System.out.println("음악 플레이어 볼륨: " + volume);

        // 볼륨 감소
        volume -= 1;
        System.out.println("음악 플레이어 볼륨: " + volume);
    }
}
```

**문제점:**
* 음소거, 볼륨 최대치 제한 등 기능이 추가될 때마다 변수 상태를 직접 건드리는 코드를 계속 추가해야 한다.
* 음악 플레이어가 2개, 3개로 늘어나면 `volume1`, `volume2`, `isOn1`, `isOn2` 처럼 변수가 무한정 늘어나 관리가 불가능해진다.

### ✨ 객체 지향 방식으로의 변경
상태(변수)와 기능(메서드)을 하나의 `MusicPlayer` 클래스로 묶어, 객체 스스로 자신의 상태를 관리하도록 만든다.

```java
// 객체(MusicPlayer)의 명세서(클래스) 작성
public class MusicPlayer {
    // 상태 (State)
    int volume = 0;
    boolean isOn = false;

    // 기능 (Methods)
    void on() {
        isOn = true;
        System.out.println("음악 플레이어를 시작합니다.");
    }

    void off() {
        isOn = false;
        System.out.println("음악 플레이어를 종료합니다.");
    }

    void volumeUp() {
        volume++;
        System.out.println("음악 플레이어 볼륨: " + volume);
    }
}
```

```java
// 메인 실행부 (사용하는 곳)
public class MusicPlayerFin {
    public static void main(String[] args) {
        MusicPlayer player = new MusicPlayer();
        
        player.on();
        player.volumeUp(); // 객체에게 "볼륨 올려"라고 명령(메시지 전달)
        player.off();
    }
}
```

---

## 3. 캡슐화 (Encapsulation)

위처럼 객체 지향으로 바꿨지만 아직 문제가 있다. 외부에서 실수나 악의적인 목적으로 값을 이상하게 바꾸는게 가능하다. (예: `player.volume = -1000000;`)
이를 막기 위해 **데이터(변수)는 직접 건드리지 못하게 숨기고, 정해진 방법(메서드)으로만 사용하도록 보호**하는 것이 바로 **캡슐화**이다.

### 접근 제어자 `private`과 Getter/Setter
* `private`: 같은 클래스 내부에서만 접근할 수 있게 만드는 접근 제어자이다.
* 외부에서는 직접 변수에 접근할 수 없으므로, 대신 값을 변경하거나 가져올 수 있는 **메서드(Setter/Getter)** 를 제공한다. 단, **Setter**를 남발하기보단 의미가 명확한 메서드(예:`volumeUp()`, `volumeDown()`)를 통해서만 변경하도록 유도하고, 단순히 값을 덮어쓰는 **Setter**는 꼭 필요할 때만 만들어 사용하는 것이 객체를 더욱 안전하게 보호하는 방법이다.

```java
public class MusicPlayer {
    // 변수는 private으로 철저히 숨김
    private int volume = 0;
    private boolean isOn = false;

    // ... 기존 메서드들 생략 ...

    // 외부에서 볼륨을 설정할 수 있도록 제공하는 정해진 메서드 (Setter)
    void setVolume(int volume) {
        this.volume = volume;
        System.out.println("음악 플레이어 볼륨 설정: " + this.volume);
    }
}
```

### `this` 키워드란?
위 코드에서 매개변수로 받은 `volume`과 객체가 원래 가진 `volume`의 이름이 같다. 이때 객체 자신의 변수(클래스 안 변수)를 가리키기 위해 `this.volume`과 같이 사용한다.

---

## 4. 오버로딩과 생성자

### 오버로딩 (Overloading)
"같은 이름, 다른 느낌". 같은 이름의 메서드이지만 전달받는 파라미터(매개변수)의 타입이나 개수를 다르게 하여 여러 개 정의하는 문법이다.

```java
public class MusicPlayer {
    // 1. 일반 볼륨 설정
    void setVolume(int volume) {
        System.out.println("일반 볼륨 설정: " + volume);
    }

    // 2. 음소거 여부와 함께 볼륨 설정 (오버로딩)
    void setVolume(int volume, boolean mute) {
        if (mute) {
            System.out.println("음소거 상태입니다.");
        } else {
            System.out.println("볼륨 설정: " + volume);
        }
    }
}
```

### 생성자 (Constructor)
객체가 처음 생성될 때 (`new MusicPlayer()`) 초기 값을 설정해 주는 역할을 한다.

```java
class MusicPlayer {
    int volume;

    // 생성자: 반환 타입이 없고 클래스 이름과 동일하다.
    MusicPlayer(int volume) {
        this.volume = volume;
    }
}

// 사용 예시: 객체를 생성함과 동시에 볼륨을 10으로 초기화!
// MusicPlayer player = new MusicPlayer(10); 
```

---

**💡 오늘의 핵심 요약:**
절차 지향 코드는 유지보수가 어려워 객체 지향으로 전환해야 하며, 객체는 상태(field)와 기능(method)을 가집니다. 외부에서 상태를 마음대로 조작할 수 없도록 `private`으로 캡슐화하고, 기능(메서드)을 통해서만 상호작용하게 설계해야 훌륭한 객체 지향 프로그램이 될 수 있다.
