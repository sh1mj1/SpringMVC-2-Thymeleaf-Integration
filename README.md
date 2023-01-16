# SpringMVC-2-Thymeleaf-Integration


# 1. 프로젝트 설정

스프링 MVC 1편에서 마지막에 완성했던 상품 관리 프로젝트를 떠올려봅시다.

지금부터 이 프로젝트에 스프링이 지원하는 다양한 기능을 붙여가면서 스프링 MVC 을 깊이있게 학습해봅시다.

MVC1 편에서 개발한 상품 관리 프로젝트를 약간 다듬어서 form-start 라는 프로젝트로 시작합니다.

**프로젝트 설정 순서**

1. form-start 의 폴더 이름을 form 로 변경합니다.

2. **프로젝트 임포트**

File Open 해당 프로젝트의 build.gradle 을 선택하자. 그 다음에 선택창이 뜨는데, Open as Project를 선택하자.

3. ItemServiceApplication.main() 을 실행해서 프로젝트가 정상 수행되는지 확인합니다.

테스트 링크

[http://localhost:8080](http://localhost:8080/) 

[http://localhost:8080/form/items](http://localhost:8080/form/items)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9d2a468b-6db7-45ca-af0b-48743bda14d0/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aff61c94-3157-41cd-9ad4-b499efaa044d/Untitled.png)


# 2. 타임리프 스프링 통합

타임리프는 크게 2가지 메뉴얼을 제공합니다.

기본 메뉴얼: https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html

스프링 통합 메뉴얼: https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html

타임리프는 스프링 없이도 동작하지만, 스프링과 통합을 위한 다양한 기능을 편리하게 제공합니다.. 그리고 이런 부분은 스프링으로 백엔드를 개발하는 개발자 입장에서 타임리프를 선택하는 하나의 이유가 됩니다.

### **스프링 통합으로 추가되는 기능들**

- 스프링의 SpringEL 문법 통합
- `${@myBean.doSomething()}` 처럼 스프링 빈 호출 지원
- 편리한 폼 관리를 위한 추가 속성
    - `th:object` (기능 강화, 폼 커맨드 객체 선택)
    `th:field` , `th:errors` , `th:errorclass`
- 폼 컴포넌트 기능
    - checkbox, radio button, List 등을 편리하게 사용할 수 있는 기능 지원
- 스프링의 메시지, 국제화 기능의 편리한 통합
- 스프링의 검증, 오류 처리 통합
- 스프링의 변환 서비스 통합(ConversionService)

### **설정 방법**

타임리프 템플릿 엔진을 스프링 빈에 등록하고, 타임리프용 뷰 리졸버를 스프링 빈으로 등록하는 방법

- https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html#the-springstandard-
dialect
- https://www.thymeleaf.org/doc/tutorials/3.0/thymeleafspring.html#views-and-view-
resolvers

스프링 부트는 이런 부분을 모두 자동화 해줍니다!!

`build.gradle` 에 다음 한줄을 넣어주면 Gradle 은 타임리프와 관련된 라이브러리를 다운로드 받고, 스프링 부트는 앞서 설명한 타임리프와 관련된 설정용 스프링 빈을 자동으로 등록해줍니다.

`build.gradle`

```groovy
implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
```

타임리프 관련 설정을 변경하고 싶으면 다음을 참고해서 `application.properties` 에 추가하면 된다.

### 스프링 부트가 제공하는 타임리프 설정, thymeleaf 검색 필요

- https://docs.spring.io/spring-boot/docs/current/reference/html/appendix-application-properties.html#common-application-properties-templating
- 개발을 하다가 필요한 일이 생기면 사용하면 됩니다.

# 3. 입력 폼 처리

지금부터 타임리프가 제공하는 입력 폼 기능을 적용해서 기존 프로젝트의 폼 코드를 타임리프가 지원하는 기능을 사용해서 효율적으로 개선해봅니다.

- `th:object` : 커맨드 객체를 지정한다.
- `*{...}` : 선택 변수 식이라고 한다. `th:object` 에서 선택한 객체에 접근한다.
- `th:field :` HTML 태그의 id , name , value 속성을 자동으로 처리해준다.

렌더링 전

```html
<input type="text" th:field="*{itemName}" />
```

렌더링 후

```html
<input type="text" id="itemName" name="itemName" th:value="*{itemName}" />
```

등록 폼

`th:object` 를 적용하려면 먼저 해당 오브젝트 정보를 넘겨주어야 한다. 등록 폼이기 때문에 데이터가 비어있는 빈 오브젝트를 만들어서 뷰에 전달하자.

변경된 `FormItemController` 

```java
package hello.itemservice.web.form;

import ...

@Controller
@RequestMapping("/form/items")
@RequiredArgsConstructor
public class FormItemController {

    private final ItemRepository itemRepository;

    @GetMapping
    public String items(Model model) {
        List<Item> items = itemRepository.findAll();
        model.addAttribute("items", items);
        return "form/items";
    }

    @GetMapping("/{itemId}")
    public String item(@PathVariable long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);
        return "form/item";
    }

    @GetMapping("/add")
    public String addForm(Model model) {
        model.addAttribute("item", new Item());
        return "form/addForm";
    }

    @PostMapping("/add")
    public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {
        Item savedItem = itemRepository.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/form/items/{itemId}";
    }

    @GetMapping("/{itemId}/edit")
    public String editForm(@PathVariable Long itemId, Model model) {
        Item item = itemRepository.findById(itemId);
        model.addAttribute("item", item);
        return "form/editForm";
    }

    @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @ModelAttribute Item item) {
        itemRepository.update(itemId, item);
        return "redirect:/form/items/{itemId}";
    }

}
```

이제 본격적으로 타임리프 등록 폼을 변경하자.

변경된 `form/addForm.html`  코드

```html
<!DOCTYPE HTML>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="utf-8">
    <link th:href="@{/css/bootstrap.min.css}"
          href="../css/bootstrap.min.css" rel="stylesheet">
    <style>
        .container {
            max-width: 560px;
        }
    </style>
</head>
<body>

<div class="container">

    <div class="py-5 text-center">
        <h2>상품 등록 폼</h2>
    </div>

    <form action="item.html" th:action th:object="${item}" method="post">
        <div>
            <label for="itemName">상품명</label>
            <!--            <input type="text" id="itemName" name="itemName" class="form-control" placeholder="이름을 입력하세요">-->
            <input type="text" id="itemName" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">
        </div>

        <div>
            <label for="price">가격</label>
            <!--            자동으로 id 와 name 을 만들어줌.-->
            <input type="text" id="price" th:field="*{price}" class="form-control" placeholder="가격을 입력하세요">
        </div>
        <div>
            <label for="quantity">수량</label>
            <!--            자동으로 id 와 name 을 만들어줌.-->
            <input type="text" id="quantity" th:field="*{quantity}" class="form-control" placeholder="수량을 입력하세요">
        </div>

        <hr class="my-4">

        <div class="row">
            <div class="col">
                <button class="w-100 btn btn-primary btn-lg" type="submit">상품 등록</button>
            </div>
            <div class="col">
                <button class="w-100 btn btn-secondary btn-lg"
                        onclick="location.href='items.html'"
                        th:onclick="|location.href='@{/form/items}'|"
                        type="button">취소
                </button>
            </div>
        </div>

    </form>

</div> <!-- /container -->
</body>
</html>
```

- `th:object="${item}"` : `<form>` 에서 사용할 객체를 지정한다. 선택 변수 식( `*{...}` )을 적용할 수 있다.
- `th:field="*{itemName}"`
    - `*{itemName}` 는 선택 변수 식을 사용했는데, `${item.itemName}` 과 같다. 앞서 `th:object` 로`item`을 선택했기 때문에 선택 변수 식을 적용할 수 있다.
    - `th:field` 는 `id` , `name` , `value` 속성을 모두 자동으로 만들어준다.
        - `id` : `th:field` 에서 지정한 변수 이름과 같다. `id="itemName"`
        - `name` : `th:field` 에서 지정한 변수 이름과 같다. `name="itemName"`
        - `value` : `th:field` 에서 지정한 변수의 값을 사용한다. `value=""`

참고로 해당 예제에서 `id` 속성을 제거해도 `th:field` 가 자동으로 만들어준다.

- 렌더링 전

```html
<input type="text" id="itemName" th:field="*{itemName}" class="form-control" placeholder="이름을 입력하세요">
```

- 렌더링 후

```html
<input type="text" id="itemName" name="itemName" class="form-control" placeholder="이름을 입력하세요"  value="">
```

[http://localhost:8080/form/items/add](http://localhost:8080/form/items/add) 실행, 페이지 소스 보기

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a17647d6-f397-4241-b002-51f3edb9fdab/Untitled.png)

실행 후 페이지 소스보기를 하면 `name` 과 `value`, `id` 까지 자동으로 만들어지는 것을 확인할 수 있습니다.

### **수정 폼**

`FormItemController` 유지

```java
@GetMapping("/{itemId}/edit")
public String editForm(@PathVariable Long itemId, Model model) {
		Item item = itemRepository.findById(itemId);
		model.addAttribute("item", item);
		return "form/editForm";
}
```

`form/editForm.html` 변경 코드 부분

```html
<form action="item.html" th:action th:object="${item}" method="post">

	<div>
		<label for="id">상품 ID</label>
		<input type="text" id="id" th:field="*{id}" class="form-control" readonly>
	</div>

	<div>
		<label for="itemName">상품명</label>
		<input type="text" id="itemName" th:field="*{itemName}" class="form-control">
	</div>

	<div>
		<label for="price">가격</label>
		<input type="text" id="price" th:field="*{price}" class="form-control">
	</div>

	<div>
		<label for="quantity">수량</label>
		<input type="text" id="quantity" th:field="*{quantity}" class="form-control">
	</div>
```

수정 폼은 앞서 설명한 내용과 같다. 수정 폼의 경우 `id` , `name` , `value` 를 모두 신경써야 했는데, 많은 부분이 `th:field` 덕분에 자동으로 처리되는 것을 확인할 수 있다.

- 렌더링 전

```html
<input type="text" id="itemName" th:field="*{itemName}" class="form-control">
```

- 렌더링 후

```html
<input type="text" id="itemName" class="form-control" name="itemName" value="itemA">
```

[http://localhost:8080/form/items/1/edit](http://localhost:8080/form/items/1/edit) 실행, 페이지 소스보기

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5e7f0de4-313b-4db1-bda3-ede43ef17651/Untitled.png)

실행 후 페이지 소스보기를 하면 `name` 과 `value`, `id` 까지 자동으로 만들어지는 것을 확인할 수 있습니다.

### **정리**

`th:object` , `th:field` 덕분에 폼을 개발할 때 약간의 편리함을 얻었다.

쉽고 단순해서 크게 어려움이 없었을 것이다.

사실 이것의 진짜 위력은 뒤에 설명할 검증(Validation)에서 나타난다. 이후 검증 부분에서 폼 처리와 관련된 부분을 더 깊이있게 알아보자.

# 4. 요구사항 추가

타임리프를 사용해서 폼에서 체크박스, 라디오 버튼, 셀렉트 박스를 편리하게 사용하는 방법을 학습해보자.

기존 상품 서비스에 다음 요구사항이 추가되었다.

- 판매 여부
    - 판매 오픈 여부
    - 체크박스로 선택할 수 있다.
- 등록 지역
    - 서울, 부산, 제주
    - 체크박스로 다중 선택할 수 있다.
- 상품 종류
    - 도서, 식품, 기타
    - 라디오 버튼으로 하나만 선택할 수 있다.
- 배송 방식
    - 빠른 배송
    - 일반 배송
    - 느린 배송
    - 셀렉트 박스로 하나만 선택할 수 있다.

### 예시 이미지

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e3f24019-27b3-482d-b533-62ae285a5a14/Untitled.png)

`ItemType` - 상품 종류

```java
package hello.itemservice.domain.item;

public enum ItemType {

    Book("도서"), FOOD("음식"), ETC("기타");

    private final String description;

    ItemType(String description) {
        this.description = description;
    }
}
```

상품 종류는 `ENUM` 을 사용한다. 설명을 위해 `description` 필드를 추가했다.

`DeliveryCode` - 배송 방식

```java
package hello.itemservice.domain.item;

import lombok.AllArgsConstructor;
import lombok.Data;

/**
 * FAST: 빠른 배송
 * NORMAL
 * SLOW
 */
@Data
@AllArgsConstructor
public class DeliveryCode {
    private String code;
    private String displayName;
}
```

배송 방식은 `DeliveryCode` 라는 클래스를 사용한다. `code` 는 FAST 처럼 시스템에서 전달하는 값이고, `displayName` 은 빠른 배송 같은 고객에게 보여주는 값이다.

`Item` - 상품

```java
package hello.itemservice.domain.item;

import lombok.Data;

import java.util.List;

@Data
public class Item {

    private Long id;
    private String itemName;
    private Integer price;
    private Integer quantity;

    private String deliveryCode; // 배송 방식
    private Boolean open; // 판매 여부
    private List<String> regions;   // 등록 지역
    private ItemType itemType; // 상품 종류

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

ENUM , 클래스, String 같은 다양한 상황을 준비했다. 각각의 상황에 어떻게 폼의 데이터를 받을 수 있는지 하나씩 알아보자.

`ItemRepository` 클래스는 이전 스프링 MVC 1편에서의 것을 그대로 사용합니다.