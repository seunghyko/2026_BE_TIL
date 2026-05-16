# 📘 Today I Learned

### 1. 실습2 STEP3에서 코드에러가 나는 이유
자바 컴파일러는 코드를 검사할 때 메모리 안에 들어있는 '실제 객체'가 무엇인지 따지지 않는다. 
즉, 오직 변수가 선언된 타입(Animal)만 보고 판단한다. 실습2 STEP3에서 Animal 타입으로 선언되어 있어 컴파일러가 Animal에 정의된 기능만 인식해버렸다 



### 2. Java의 컬렉션 프레임워크(List, Map)
# Java 핵심 자료구조: ArrayList & HashMap

---

## 1. ArrayList

### 개념
- `List` 인터페이스를 구현한 동적 배열기반의 컬렉션
- 순서를 보장하며 중복 요소 허용



### 기본 문법

```java
import java.util.ArrayList;
import java.util.List;

// 선언 및 생성 
List<String> list = new ArrayList<>();
```

### 주요 메서드

```java
List<String> fruits = new ArrayList<>();

// 추가
fruits.add("apple");             // 끝에 추가
fruits.add(0, "mango");          // 인덱스 위치에 삽입

// 조회
String first = fruits.get(0);   // 인덱스로 접근
int size = fruits.size();        // 요소 개수
boolean has = fruits.contains("apple");  // 포함 여부

// 수정
fruits.set(0, "grape");          // 인덱스 위치 값 교체

// 삭제
fruits.remove("apple");          // 값으로 삭제
fruits.remove(0);                // 인덱스로 삭제
fruits.clear();                  // 전체 삭제

// 검색
int idx = fruits.indexOf("apple");   // 첫 번째 인덱스 (-1이면 없음)

// 정렬
import java.util.Collections;
Collections.sort(fruits);            // 오름차순
Collections.sort(fruits, Collections.reverseOrder()); // 내림차순
```



---

## 2. HashMap

### 개념
- `Map` 인터페이스를 구현한 **해시 테이블** 기반의 컬렉션
- **Key-Value 쌍**으로 데이터를 저장하며, Key의 중복을 허용하지 않음
- 삽입 순서를 보장하지 않는다


### 기본 문법

```java
import java.util.HashMap;
import java.util.Map;

// 선언 및 생성 
Map<String, Integer> map = new HashMap<>();
```

### 주요 메서드

```java
Map<String, Integer> scores = new HashMap<>();

// 삽입 / 수정
scores.put("Alice", 90);         // Key가 없으면 삽입, 있으면 덮어씀
scores.putIfAbsent("Alice", 80); // Key가 없을 때만 삽입

// 조회
int s = scores.get("Alice");           // 없으면 NullPointerException
int s = scores.getOrDefault("Bob", 0); // 없으면 기본값 반환 (권장)
boolean has = scores.containsKey("Alice");   // Key 존재 여부
boolean has = scores.containsValue(90);      // Value 존재 여부

// 삭제
scores.remove("Alice");          // Key로 삭제
scores.clear();                  // 전체 삭제

// 크기
int size = scores.size();
```







