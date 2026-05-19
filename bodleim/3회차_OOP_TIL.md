# 📘 Today I Learned

## 1. 실습2 STEP3에서 발생한 컴파일 오류 원인

자바에서는 변수를 통해 메서드나 필드에 접근할 때, **실제 객체가 무엇인지보다 변수의 타입을 먼저 기준으로 검사한다.**

예를 들어 `Animal animal = new Dog();`처럼 작성하면, 메모리에는 `Dog` 객체가 만들어져 있어도 컴파일러는 `animal` 변수를 `Animal` 타입으로 본다.  
따라서 `Dog` 클래스에만 있는 메서드를 바로 호출하면 컴파일 단계에서 오류가 발생할 수 있다.

즉, 실습2 STEP3의 핵심은 **객체 자체는 하위 클래스일 수 있지만, 참조 변수의 타입이 Animal이면 Animal에 정의된 기능만 사용할 수 있다**는 점이다.

---

## 2. Java 컬렉션 프레임워크 정리

# Java에서 자주 쓰는 컬렉션: ArrayList와 HashMap

---

## 1. ArrayList

### 개념

`ArrayList`는 여러 데이터를 순서대로 저장할 때 사용하는 컬렉션이다.  
배열과 비슷하지만 크기가 고정되어 있지 않아, 데이터를 추가하거나 삭제하기가 더 편하다.

- 저장한 순서가 유지된다
- 같은 값을 여러 번 넣을 수 있다
- 인덱스를 이용해 원하는 위치의 값을 가져올 수 있다

### 기본 문법

```java
import java.util.ArrayList;
import java.util.List;

List<String> subjects = new ArrayList<>();
```

### 사용 예시

```java
List<String> subjects = new ArrayList<>();

// 과목 추가
subjects.add("Java");
subjects.add("Spring");
subjects.add("Database");
subjects.add(1, "Git");

// 조회
String firstSubject = subjects.get(0);
int subjectCount = subjects.size();
boolean hasSpring = subjects.contains("Spring");

// 수정
subjects.set(2, "MySQL");

// 삭제
subjects.remove("Git");
subjects.remove(0);

// 검색
int springIndex = subjects.indexOf("Spring");

// 전체 삭제
subjects.clear();
```

### 정렬 예시

```java
import java.util.Collections;

List<Integer> numbers = new ArrayList<>();

numbers.add(30);
numbers.add(10);
numbers.add(20);

Collections.sort(numbers); // 오름차순 정렬
Collections.sort(numbers, Collections.reverseOrder()); // 내림차순 정렬
```

---

## 2. HashMap

### 개념

`HashMap`은 데이터를 **이름표와 값**처럼 묶어서 저장할 때 사용하는 컬렉션이다.  
리스트처럼 순서로 찾는 것이 아니라, `Key`를 이용해 원하는 `Value`를 찾는다.

- 데이터를 `Key-Value` 형태로 저장한다
- Key는 중복될 수 없다
- 같은 Key에 다시 값을 넣으면 기존 값이 바뀐다
- 기본적으로 저장 순서는 보장되지 않는다

### 기본 문법

```java
import java.util.HashMap;
import java.util.Map;

Map<String, String> phoneBook = new HashMap<>();
```

### 사용 예시

```java
Map<String, String> phoneBook = new HashMap<>();

// 연락처 추가 / 수정
phoneBook.put("민수", "010-1111-2222");
phoneBook.put("지연", "010-3333-4444");
phoneBook.put("민수", "010-9999-8888"); // 같은 Key라 기존 값 변경

phoneBook.putIfAbsent("현우", "010-5555-6666");

// 조회
String minsuNumber = phoneBook.get("민수");
String unknownNumber = phoneBook.getOrDefault("수빈", "등록된 번호 없음");

boolean hasMinsu = phoneBook.containsKey("민수");
boolean hasNumber = phoneBook.containsValue("010-3333-4444");

// 삭제
phoneBook.remove("지연");

// 크기 확인
int totalContacts = phoneBook.size();

// 전체 삭제
phoneBook.clear();
```

---

## 정리

`ArrayList`는 여러 값을 **순서대로 관리**할 때 적합하고,  
`HashMap`은 특정 이름이나 키를 기준으로 값을 **빠르게 찾고 관리**할 때 적합하다.
