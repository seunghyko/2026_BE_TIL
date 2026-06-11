# [TIL] 백엔드 6회차 세션: DTO 계층 구조 완성 및 REST API 설계 (CRUD)

## 1. Entity와 DTO 비교・분석

백엔드 아키텍처에서 데이터는 브라우저를 시작으로 `DispatcherServlet → Controller → Service → Repository → DB`라는 긴 터널을 타고 이동합니다. 이때 각 역할에 따라 데이터의 모양을 다르게 담아야 하는 이유가 있습니다.

### 1) 개념 정의

* **Entity (도메인 객체):** 데이터베이스(DB) 테이블과 1:1로 매핑되는 가장 핵심적인 객체입니다. DB 테이블의 한 줄(row)이 자바 객체 하나로 표현되며, 데이터의 본질적인 구조를 담당합니다.


* **DTO (Data Transfer Object, 데이터 운반 그릇):** 말 그대로 계층(Layer) 간, 혹은 네트워크 사이를 안전하게 건너다니며 데이터를 배달하는 순수한 데이터 전송용 그릇입니다. 비즈니스 로직을 포함하지 않고 오직 데이터만 담고 있습니다.



### 2) Entity를 외부에 그대로 노출(응답/요청)했을 때의 3가지 치명적 문제점 

1. **보안 및 민감 정보 유출 (문제 1):** 회원 엔티티(`Member`)나 할 일 엔티티(`Todo`)에 `password`, `secretMemo(비밀 메모)` 같은 필드가 포함되어 있다고 가정해 봅시다. Entity를 그대로 JSON으로 응답하면 프론트엔드 화면이나 네트워크 패킷에 사용자의 비밀번호까지 그대로 노출되는 보안 사고가 발생합니다.


2. **요청/응답 데이터의 불일치 및 오염 (문제 2):** 데이터를 등록할 때(Request)와 조회할 때(Response) 필요한 정보는 매번 다릅니다.


* **등록 시:** 아직 DB에 저장되기 전이라 고유 번호(`id`)가 없습니다.


* **조회 시:** 고유 번호(`id`)를 포함해서 내려주어야 합니다.


* 만약 Entity 하나만 공유해서 쓰면, 데이터를 등록하는 요청 창구에 `id` 필드가 버젓이 열려 있게 됩니다. 이를 악용해 클라이언트가 강제로 임의의 `id`를 주입하여 요청을 보내는 시스템 오염 사고가 일어날 수 있습니다. 용도별로 그릇을 따로 두어야 깔끔하고 안전합니다.




3. **DB와 API의 강한 결합(유지보수 지옥):** Entity는 DB 테이블 모양과 완전히 한 몸입니다. 만약 서비스가 커져서 DB의 컬럼명을 하나 바꾸면, Entity의 필드명이 바뀌고, 결과적으로 프론트엔드가 받던 JSON의 키(Key)값까지 모조리 깨져버립니다. DB의 사소한 변경이 API 스펙 전체를 뒤흔드는 대참사가 발생합니다.



### 3) Entity vs DTO 비교표

| 비교 항목 | Entity (엔티티) | DTO (데이터 전송 객체) |
| :--- | :--- | :--- |
| **주된 역할** | DB 테이블 매핑 및 비즈니스 핵심 데이터 관리 | 계층 간, 네트워크 간 데이터 안전 운반 |
| **비즈니스 로직** | 포함될 수 있음 (도메인 주도 설계 시) | **포함되지 않음** (순수한 데이터 그릇) |
| **구조가 바뀌는 이유** | 데이터베이스(DB)의 테이블 구조가 변경될 때 | API 스펙이나 프론트엔드의 요구사항이 변경될 때 |
| **외부 노출 여부** | **외부에 직접 노출 금지** (Repository 내부에서만 서식) | **외부와 직접 통신** (요청을 받고 응답을 보냄) |

---

## 2. DTO에 붙는 롬복(Lombok) 및 스프링 어노테이션

DTO 클래스를 작성할 때 붙이던 어노테이션들이 내부적으로 어떤 나비효과를 불러일으키는지 이해해야한다.

### 1) @Getter (필드 읽기 / 직렬화의 핵심) 

* **원리:** 자바 객체의 필드 값을 읽어오는 `getXxx()` 메서드를 자동으로 생성해 줍니다.
* **Response DTO에 필수인 이유:** 자바 백엔드가 Response DTO 객체를 프론트엔드로 보낼 때, 스프링 내부의 **Jackson 라이브러리**가 자바 객체를 JSON 문자열로 변환하는 과정(**직렬화**)을 거칩니다. 이때 Jackson은 필드에 직접 접근하는 것이 아니라, **객체의 Getter 메서드를 호출하여 값을 추출**합니다. 만약 Response DTO에 `@Getter`를 빼먹으면 스프링이 값을 꺼낼 방법이 없어서 프론트엔드는 텅 빈 중괄호 `{ }`만 받게 됩니다.



### 2) @Setter (필드 쓰기 / 역직렬화의 도구) 

* **원리:** 필드에 값을 저장하는 `setXxx()` 메서드를 자동으로 생성해 줍니다.
* **Request DTO에서 쓰이는 이유:** 클라이언트가 보내온 JSON 텍스트 데이터를 자바 객체로 변환할 때(**역직렬화**), 빈 그릇에 데이터를 채워 넣기 위해 사용됩니다.



### 3) @NoArgsConstructor (기본 생성자) 

* **원리:** 파라미터(매개변수)가 아예 없는 기본 생성자 `public ClassName() {}`를 자동으로 만들어 줍니다.
* **필수 이유:** 스프링이 JSON 요청을 받아 DTO 객체로 바꿀 때, 내부적으로 리플렉션 기술을 사용해 **"일단 알맹이가 빈 객체(기본 생성자 이용)를 먼저 하나 개설한 뒤"** 값을 주입합니다. 기본 생성자가 존재하지 않으면 객체 자체를 개설하지 못해 에러가 발생합니다.

### 4) @RequestBody (컨트롤러의 입구) 

* **원리:** HTTP 요청 본문(Body)에 담긴 JSON 문자열을 자바의 객체(Request DTO)로 자동 파싱하여 변수에 넣어주는 스프링 어노테이션입니다.


* **주의점:** 자바는 타입에 엄격하므로, JSON의 Key 이름과 DTO의 필드명이 글자 하나, 대소문자 하나까지 완벽히 매칭되어야 합니다. 만약 프론트엔드에서 `{"todoTitle": "운동"}`으로 보냈는데 DTO 필드가 `String title;`로 되어 있다면 매칭에 실패하여 필드에 `null`이 박히는 참사가 일어납니다.



---

## 3. 데이터 변환 구조와 `static` 키워드

과거에는 Controller나 Service 계층에서 DTO에서 값을 하나하나 꺼내 (`req.getItemName()`) Entity 객체에 수동으로 박아 넣었습니다. 하지만 이 방식은 코드가 지저분해지고 책임 분리가 되지 않습니다. 따라서 **변환 로직 자체를 DTO 내부 메서드로 격리**해야 합니다.

### 1) Request DTO → Entity 변환 (`toEntity()` 메서드) 

* 이미 클라이언트의 데이터가 Request DTO 객체 안에 가득 채워져서 생성된 상태입니다.


* 따라서 **일반 인스턴스 메서드**로 생성합니다. 메서드 내부에서 `this.itemName`처럼 `this`(나 자신)의 값을 활용하여 새 Entity 객체를 찍어내어 반환합니다.



### 2) Entity → Response DTO 변환 (`from()` 메서드와 `static`) 

* static(정적) 메서드란? (카페 비유) 


* **일반 메서드:** "카페 문을 열고 들어가서 안에서 주문하는 것"과 같습니다. 즉, `new ResponseDTO()`를 통해 **객체가 이미 먼저 메모리에 존재해야만** 호출할 수 있습니다.
* **static 메서드:** "카페에 들어가지 않고, 밖에서 건물 간판만 보고 바로 드라이브스루로 주문하는 것"과 같습니다. 객체를 생성(`new`)하지 않아도, **클래스 이름 자체(`TodoResponse.from(todo)`)로 직접 호출**할 수 있는 특별한 정적 메서드입니다.


* **왜 여기서 static을 써야 할까?** <br>
엔티티를 응답 DTO로 바꿀 때, 우리는 아직 응답 DTO 객체를 만들기 전입니다. 손에는 오직 DB에서 꺼내온 엔티티(`Todo`)만 쥐어져 있습니다. 응답 DTO 객체가 존재하지도 않는데 일반 메서드를 쓸 수는 없다. 따라서 객체 없이도 클래스 틀만 보고 곧바로 엔티티를 주입받아 DTO 객체를 조립해 낼 수 있도록 **`static` 기반의 정적 팩토리 메서드**로 설계해야 합니다.



---

## 4. API 명세서 구조 분석 (식당 점원 비유) 

API(Application Programming Interface)를 가장 쉽게 이해하는 방법은 '식당의 점원'으로 생각하는 것입니다. 손님(클라이언트)은 요리가 진행되는 주방(서버 내부)에 직접 들어갈 수 없으며, 반드시 점원(API)에게 메뉴판(API 명세서)을 보고 주문(요청)해야 완성된 음식(응답)을 돌려받을 수 있습니다.

### 1) API 명세서의 5대 핵심 기입 요소 

1. **카테고리:** 이 API가 다루는 본질적인 '자원(Resource)'의 묶음 분류입니다. 보통 URL의 첫 번째 복수 명사 이름과 일치합니다 (예: 상품 관리 -> `/api/items`, 출결 관리 -> `/api/attendances`).


2. **Param (파라미터):** API를 호출할 때 필터링이나 식별을 위해 주입하는 변수명입니다 (예: `id`, `date`).


3. **Request Body (요청 본문):** POST(등록)나 PUT(수정)처럼 새롭게 저장하거나 변경할 대량의 실제 데이터를 JSON 데이터로 묶어 전달하는 본문입니다. *※ GET(조회)과 DELETE(삭제)는 대상 식별자가 이미 URL 경로에 드러나 있으므로 보통 Request Body가 비어있습니다.* 


4. **Response (응답 구조):** 서버가 처리를 마친 후 돌려줄 JSON 스펙입니다.


* **Key:** JSON 텍스트로 치환되어 나갈 영문 필드명.


* **설명:** 해당 필드가 무엇을 의미하는지 적어둔 한글 명세.


* **타입:** String, number, Array 등 데이터 형태 명시.




5. **옵션 상세 (ENUM 및 Nullable)** 


* **ENUM (열거형):** 오타나 실수를 원천 차단하기 위해 들어올 수 있는 규격 값을 미리 엄격하게 지정해 둔 집합입니다 (예: 출결 상태를 `ATTEND`, `ABSENT` 외에 `APPLE` 같은 오타가 들어오지 못하게 서버 단계에서 잠그는 것).


* **Nullable:** 필수 값인지, 아니면 비어있어도(`null`) 허용되는 값인지 명시하여 프론트엔드가 화면을 그릴 때 참고하도록 합니다.




6. **Status (HTTP 상태 코드):** `2xx`(성공), `4xx`(클라이언트 요청 오류), `5xx`(서버 내부 에러)를 상황별로 미리 정의하여 프론트엔드가 매끄럽게 에러 분기 처리를 하도록 안내합니다.



---

## 5. 상품 관리(Item) CRUD API 실전 전체 소스 코드 

안쪽 데이터(Entity)부터 시작해서 바깥쪽 문(Controller) 순서로 흐름이 이어지도록 유기적으로 설계된 전체 스택 코드입니다.

### 1) [Entity] `domain/Item.java` 

```java
package com.likelion.itemapi.domain;

import lombok.Getter;
import lombok.Setter;

@Getter @Setter
public class Item {
    private Long id;            // 저장소(DB)가 자동으로 채워줄 고유 식별자 
    private String itemName;    // 상품명 
    private Integer price;      // 가격
    private Integer quantity;   // 수량 

    public Item() {} // 스프링과 다양한 라이브러리를 위한 기본 생성자

    // [중요] 상품 등록 시점에는 id를 알 수 없으므로, id 없이 객체를 조립하는 생성자를 뚫어둡니다. 
    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}

```

### 2) [Repository] `repository/ItemRepository.java` 

```java
package com.likelion.itemapi.repository;

import com.likelion.itemapi.domain.Item;
import org.springframework.stereotype.Repository;
import java.util.*;

@Repository
public class ItemRepository {
    // 실제 데이터베이스가 연동되기 전이므로, 메모리 상의 HashMap을 임시 DB로 활용합니다.
    private final Map<Long, Item> store = new HashMap<>();
    private long sequence = 0L; // id를 1씩 증가시켜 발급해 줄 시퀀스 변수 

    // Create (저장)
    public Item save(Item item) {
        item.setId(++sequence); // 시퀀스 증가 후 엔티티에 세팅 
        store.put(item.getId(), item);
        return item;
    }

    // Read (전체 조회)
    public List<Item> findAll() {
        return new ArrayList<>(store.values());
    }

    // Read (단건 조회)
    public Item findById(Long id) {
        return store.get(id);
    }

    // Update (수정)
    public void update(Long id, Item updateParam) {
        Item foundItem = store.get(id);
        if (foundItem != null) {
            foundItem.setItemName(updateParam.getItemName());
            foundItem.setPrice(updateParam.getPrice());
            foundItem.setQuantity(updateParam.getQuantity());
        }
    }

    // Delete (삭제)
    public void delete(Long id) {
        store.remove(id);
    }
}

```

### 3) [Request DTO] `dto/ItemRequest.java` 

```java
package com.likelion.itemapi.dto;

import com.likelion.itemapi.domain.Item;
import lombok.Getter;
import lombok.Setter;
import lombok.NoArgsConstructor;

@Getter 
@Setter               // JSON 데이터 수신(역직렬화) 시 필드 매핑을 위해 셋터 개방 
@NoArgsConstructor    // 역직렬화의 필수 조건인 기본 생성자 자동 주입 
public class ItemRequest {
    private String itemName;
    private Integer price;
    private Integer quantity;

    // [책임 분리] 들어온 그릇(DTO) 스스로가 엔티티 객체로 변환되어 나가는 인스턴스 메서드 
    public Item toEntity() {
        return new Item(itemName, price, quantity); // 생성자 시점에 id가 없는 상태로 빌드 
    }
}

```

### 4) [Response DTO] `dto/ItemResponse.java` 

```java
package com.likelion.itemapi.dto;

import com.likelion.itemapi.domain.Item;
import lombok.Getter;

@Getter // [주의] 이 Getter가 없으면 Jackson 직렬화가 실패하여 { } 공백으로 출력됩니다! 
public class ItemResponse {
    private final Long id;
    private final String itemName;
    private final Integer price;
    private final Integer quantity;

    // 내부 조립용 생성자
    public ItemResponse(Item item) {
        this.id = item.getId();
        this.itemName = item.getItemName();
        this.price = item.getPrice();
        this.quantity = item.getQuantity();
    }

    // [정적 팩토리 메서드] 객체 생성 없이 클래스 레벨에서 엔티티를 DTO 그릇으로 전환해 주는 변환기 
    public static ItemResponse from(Item item) {
        return new ItemResponse(item);
    }
}

```

### 5) [Service] `service/ItemService.java` 

```java
package com.likelion.itemapi.service;

import com.likelion.itemapi.domain.Item;
import com.likelion.itemapi.dto.ItemRequest;
import com.likelion.itemapi.dto.ItemResponse;
import com.likelion.itemapi.repository.ItemRepository;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
@RequiredArgsConstructor // final이 붙은 리포지토리를 생성자 주입(DI) 방식으로 자동 수신
public class ItemService {
    private final ItemRepository itemRepository;

    // 상품 등록 
    public ItemResponse create(ItemRequest request) {
        Item item = request.toEntity();        // 1. DTO를 엔티티로 해체 
        Item savedItem = itemRepository.save(item); // 2. 저장소에 대행 전송 
        return ItemResponse.from(savedItem);  // 3. 엔티티를 응답그릇에 담아 변환 
    }

    // 전체 조회
    public List<ItemResponse> findAll() {
        return itemRepository.findAll().stream()
                .map(ItemResponse::from) // 깔끔하게 자바 스트림을 이용하여 DTO 리스트로 일괄 캐스팅
                .toList();
    }

    // 단건 조회 
    public ItemResponse findOne(Long id) {
        Item item = itemRepository.findById(id);
        return ItemResponse.from(item);
    }

    // 상품 수정 
    public ItemResponse update(Long id, ItemRequest request) {
        itemRepository.update(id, request.toEntity());
        return ItemResponse.from(itemRepository.findById(id));
    }

    // 상품 삭제 
    public void delete(Long id) {
        itemRepository.delete(id);
    }
}

```

### 6) [Controller] `controller/ItemApiController.java`

```java
package com.likelion.itemapi.controller;

import com.likelion.itemapi.dto.ItemRequest;
import com.likelion.itemapi.dto.ItemResponse;
import com.likelion.itemapi.service.ItemService;
import lombok.RequiredArgsConstructor;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController // 리턴값을 자동으로 JSON 직렬화하여 클라이언트에 쏴주는 핵심 입구
@RequestMapping("/api/items") // 이 컨트롤러가 공통으로 가질 상위 URL 패스 명세 
@RequiredArgsConstructor
public class ItemApiController {
    private final ItemService itemService;

    // 1. 상품 등록 (POST /api/items) 
    @PostMapping
    public ItemResponse createItem(@RequestBody ItemRequest request) { // JSON 본문을 자바 객체로 강제 역직렬화 
        return itemService.create(request);
    }

    // 2. 전체 조회 (GET /api/items) 
    @GetMapping
    public List<ItemResponse> getItems() {
        return itemService.findAll();
    }

    // 3. 단건 조회 (GET /api/items/{id}) 
    @GetMapping("/{id}")
    public ItemResponse getItem(@PathVariable Long id) { // URL 경로에 박힌 식별자를 파라미터로 추출
        return itemService.findOne(id);
    }

    // 4. 상품 수정 (PUT /api/items/{id}) 
    @PutMapping("/{id}")
    public ItemResponse updateItem(@PathVariable Long id, @RequestBody ItemRequest request) {
        return itemService.update(id, request);
    }

    // 5. 상품 삭제 (DELETE /api/items/{id}) 
    @DeleteMapping("/{id}")
    public void deleteItem(@PathVariable Long id) {
        itemService.delete(id);
    }
}

```

---

## 6. 3줄 요약

### 1) **DTO 분리** : 데이터 유출 등의 보안 사고를 방지하고 DB 테이블 변경이 API 스펙을 흔들지 않도록, 외부에 Entity를 직접 노출하는 대신 전송 전용 그릇인 DTO를 반드시 분리하여 사용해야 한다.
### 2) **REST API 규격화** : 프론트엔드와의 매끄러운 통신 및 분기 처리를 위해 HTTP 메서드(POST・GET・PUT・DELETE), 상태코드(2xx・4xx・5xx), 바디 데이터, ENUM 등을 명확히 정의한 API 명세서를 설계해야 한다.
### 3) **CRUD 구현 흐름 및 책임 격리** : 상품 관리 기능은 안쪽 데이터(Entity・Repository)부터 바깥쪽 입구(Controller) 순으로 구현하며, DTO 내부에 변환 로직(`toEntity()`, `static from()`)을 격리하여 계층 간 결합도를 낮춰야 한다.

## 7. 막히거나 이해가 안 가는 것
반복해서 내용을 보다보니 아~ 싶은 것들은 많이 생기나, 실제 코드를 보거나 구현을 시도할 때는 뭐부터 해야할지 감이 잘 안 잡힌다. 

## 8. Rest API 설계
| 기능 | Method | URL | 요청 바디 | 응답 | 상태코드 |
|---|---|---|---|---|---|
| 목록 조회 | GET | /api/items | 없음 | ItemResponse 리스트 | 200 OK |
| 단건 조회 | GET | /api/items/{id} | 없음 | ItemResponse | 200 OK |
| 등록 | POST | /api/items | ItemRequest | ItemResponse | 201 Created |
| 수정 | PUT | /api/items/{id} | ItemRequest | ItemResponse | 200 OK |
| 삭제 | DELETE | /api/items/{id} | 없음 | 없음 | 204 No Content |