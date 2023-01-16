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

# 5. 체크박스 - 단일1

단순 HTML 체크 박스

`resources/templates/form/addForm.html` 추가

```html
<hr class="my-4">

<!--        single checkbox-->
  <div>판매 여부</div>
  <div>
      <div class="form-check">
          <input type="checkbox" id="open" name="open" class="form-check-input" >
          <label for="open" class="form-check-label" > 판매 오픈</label>
      </div>
  </div>
```

상품이 등록되는 곳에 다음과 같이 로그를 남겨서 값이 잘 넘어오는지 확인해보자.

`FormItemController` 추가

`FormItemController`  클래스 레벨에 `@Slf4j` 태그를 추가해주고, `addItem` 메서드에  아래 로그를 찍는 코드를 추가해줍니다.

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    log.info("item.open={}", item.getOpen());
    return "redirect:/form/items/{itemId}";
}
```

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c5a70cb8-ccd2-4e3d-821d-078933eebb47/Untitled.png)

[http://localhost:8080/form/items/4?status=true](http://localhost:8080/form/items/4?status=true)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6a5b12c8-6c07-45df-a139-e438b49e9954/Untitled.png)

로그 메시지는 아래와 같이 표시되는 것을 확인할 수 있다.

2023-01-16 15:19:56.381 INFO 5062 --- [nio-8080-exec-5] h.i.web.form.FormItemController : **item.open=true**

f12 을 눌러서 페이지 정보를 보면 HTML Form 에는 `open = on` 이라는 값이 넘어가는 것을 알 수 있다.

스프링은 on 이라는 문자를 true 타입으로 변환해준다.

그런데 만약 판매 오픈 체크박스를 체크하지 않고 상품을 등록한다면 어떻게 될까요?

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f532ef78-fe7b-4e27-a175-d93d42f1d57e/Untitled.png)

로그 메시지는 아래와 같다.

2023-01-16 15:28:10.868 INFO 5062 --- [nio-8080-exec-4] h.i.web.form.FormItemController : **item.open=null**

위 그림처럼  클라이언트에서 서버로 아예 `open` 이라는 값을 보내지 않는다. 

만약 우리가 판매 여부가 `true` 인 값을 다시 `false` 로 바꾸려는 의도로 상품 수정을 한다면 위와 같은 동작은 문제가 됩니다!

스프링 MVC 는 특별한 기술을 사용하여 이런 문제를 해결합니다

히든 필드를 하나 만들어서 `_open` 처럼 기존 체크박스 이름 앞에 언더바 를 붙여 전송하면 체크를 해제했다고 인식할 수 있습니다.

히든 필드는 항상 전송됩니다. 따라서 위 예시처럼 체크를 해제한 경우 `open` 은 전송되지 않고 `_open` 만 전송되는며, 이 때 스프링 MVC 는 체크를 해제했다고 판단합니다.

히든 필드를 사용하여 체크 해제를 하려면 아래 코드를 사용하면 됩니다.

체크 해제를 인식하기 위한 히든 필드

```html
<input type="hidden" name="_open" value="on"/>
```

우리의 예제에서는 아래처럼 작성하면 되겠지요!

```html
<!--        single checkbox-->
<div>판매 여부</div>
<div>
    <div class="form-check">
        <input type="checkbox" id="open" name="open" class="form-check-input" >
        <input type="hidden" name="_open" value="on"/>
        <label for="open" class="form-check-label" > 판매 오픈</label>
    </div>
</div>
```

그렇다면 실행을 해봅시다.

### 채크박스 체크시

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ab59dcf0-2355-4ff9-9204-19829e401552/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/da18a92d-e339-46ae-9f7e-e4a0dcfec7bb/Untitled.png)

체크박스를 체크하고 상품 등록을 하면 `open = on`  & `_open = on` 을 보내줍니다!

로그메시지

2023-01-16 15:36:23.494 INFO 5150 --- [nio-8080-exec-7] h.i.web.form.FormItemController : **item.open=true**

### 체크 해제시

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ec6daced-750b-41ea-8e06-4836ee033384/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d6a51615-547d-416f-bcd1-220325b68fee/Untitled.png)

체크박스를 체크하지 않고 보내면 `_open` 이라는 새로운 필드가 만들어져서 오직 `_open=on` 만이 넘어가는 것을 알 수 있습니다.

로그메시지

2023-01-16 15:34:02.951 INFO 5150 --- [nio-8080-exec-2] h.i.web.form.FormItemController : **item.open=false**

그런데 이렇게 체크박스를 만들 때마다 히든 필드를 만들어야 하는 것은 너무 번거롭습니다. 그렇다면 어떻게 편하게 해결할 수 있을까요? 그 방법은 아래에서 바로 설명됩니다.


# 6. 체크박스 - 단일2

개발할 때 마다 위처럼 히든 필드를 추가하는 것은 상당히 번거롭습니다. 타임리프가 제공하는 폼 기능을 사용하면 이런 부분을 자동으로 처리할 수 있다.

`resources/templates/form/addForm.html` 변경 (체크박스 코드 `th:field="*{open}"` )

```html
<!--        single checkbox-->
<div>판매 여부</div>
<div>
    <div class="form-check">
        <input type="checkbox" id="open" th:field="*{open}" class="form-check-input" >
        <label for="open" class="form-check-label" > 판매 오픈</label>
    </div>
</div>
```

[http://localhost:8080/form/items/add](http://localhost:8080/form/items/add) 실행 후 페이지 소스 보기

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d512f605-ed51-46ab-b6e5-e5855e0f9986/Untitled.png)

44번 줄을 보면 자동으로  <input type=”hidden” name=”_open” value=”on” /> 이라는 필드를 추가해 주는 것을 확인할 수 있다!!!

즉, 이전처럼 직접 히든 필드를 만들지 않아도 히든 필드 부분은 `th:field=”*{open}”` 이라는 타임리프 코드가 알아서 만들어 주는 것이다!!!

그렇다면 상품 상세에도 적용해봅시다.

`item.html`

```html
<hr class="my-4">
<!--    single checkbox-->
<div>판매여부</div>
<div>
    <div class="form-check">
        <input type="checkbox" id="open" th:field="${item.open}" class="form-check-input" disabled>
        <labe for="open" class="form-check-label">판매 오픈</labe>
    </div>
</div>
```

주의할 점은 현재 `item.html` 에는 `th:object` 를 사용하지 않았기 때문에 `th:field` 부분에 `${item.open}` 으로 적어주어야 합니다!!

여기서는 `disabled` 를 사용해서 상품 상세에서는 체크 박스가 선택되지 않도록 했다.

실행 후 상품 등록 시 판매 여부 체크박스를 체크하여 등록했다. 등록 후에는 상품 상세 화면으로 이동하게 된다.

이 때 페이지 소스 보기를 하면 아래와 같다.

```html
<hr class="my-4">
<!--    single checkbox-->
<div>판매여부</div>
<div>
    <div class="form-check">
        <input type="checkbox" id="open" class="form-check-input" disabled name="open" value="true" checked="checked">
        <labe for="open" class="form-check-label">판매 오픈</labe>
    </div>
</div>
```

소스를 보면 `checked="checked"` 라는 속성이 추가된 것을 확인할 수 있습니다.

이런 부분은 개발자가 직접 처리하려면 상당히 번거롭지만 `th:field` 를 사용한다면 값이 `true` 인 경우 체크를 자동으로 처리해줍니다!!!

상품 수정에도 똑같이 적용해줍니다.

`editForm.html`  에 추가

```html
<hr class="my-4">
<!--        single checkbox-->
<div>판매 여부</div>
<div>
    <div class="form-check">
        <input type="checkbox" id="open" th:field="*{open}" class="form-check-input">
        <label for="open" class="form-check-label">판매 오픈 </label>
    </div>
</div>
```

상품 수정도 `th:object` , `th:field` 를 모두 적용합니다.

체크 박스를 수정해도 반영되도록  아래처럼 코드를 수정합니다.

`ItemRepository` - `update()` 메서드 수정

```java
public void update(Long itemId, Item updateParam) {
    Item findItem = findById(itemId);
    findItem.setItemName(updateParam.getItemName());
    findItem.setPrice(updateParam.getPrice());
    findItem.setQuantity(updateParam.getQuantity());

    findItem.setOpen(updateParam.getOpen());
    findItem.setRegions(updateParam.getRegions());
    findItem.setItemType(updateParam.getItemType());
    findItem.setDeliveryCode(updateParam.getDeliveryCode());
}
```

open 이외에 나머지 필드도 업데이트 되도록 미리 넣어둡니다.



# 7. 체크박스 - 멀티

체크 박스를 멀티로 사용해서, 하나 이상을 체크할 수 있도록 해봅니다.

- 등록 지역
    - 서울, 부산, 제주
    - 체크 박스로 다중 선택할 수 있다

그런데 문제가 있다. 등록 폼, 상세화면, 수정 폼에서 모두 서울, 부산, 제주라는 체크 박스를 반복해서 보여주어야 한다. 이렇게 하려면 각각의 컨트롤러에서 `model.addAttribute(...)` 을 사용해서 체크 박스를 구성하는 데이터를 반복해서 넣어주어야 한다. 

여러 페이지에 이런식으로 계속해서 각 url 매핑 메서드에 넣는 것은 너무 쓸데 없는 반복이다.

스프링 MVC 에서는 이 반복을 하지 않도록 하기 위해 `@ModelAttribute` 기술을 사용할 수 있다.

`FormItemController` - 추가

```java
@ModelAttribute("regions")
public Map<String, String> regions() {
    Map<String, String> regions = new LinkedHashMap<>();
    regions.put("SEOUL", "서울");
    regions.put("BUSAN", "부산");
    regions.put("JEJU", "제주");
    return regions;
}
```

`@ModelAttribute`의 특별한 사용법

 `@ModelAttribute` 는 이렇게 컨트롤러에 있는 별도의 메서드에 적용할 수 있다. 이렇게 하면 해당 컨트롤러를 요청할 때 `regions` 에서 반환한 값이 자동으로 모델( `model` )에 담기게 된다. 물론 이렇게 사용하지 않고, 각각의 컨트롤러 메서드에서 모델에 직접 데이터를 담아서 처리해도 된다.

순서가 보장되는 `Map`  인 `LinkedHashMap` 을 사용하였다. 물론 성능 최적화를 하는 것은 고민해봐야 합니다. 해당 지역이 바뀌지 않는다면 `static` 영역에 `Map` 을 만들어 놓는 것이 더 좋을 것입니다.

`addForm.html` - 추가

```html
<!--        multi checkbox-->
<div>
    <div>등록 지역</div>
    <div th:each="region: ${regions}" class="form-check form-check-inline">
        <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
        <label th:for="${#ids.prev('regions')}" th:text="${region.value}" class="form-check-label"> 서울 </label>
    </div>
</div>
```

`th:for="${#ids.prev('regions')}”`

멀티 체크박스는 같은 이름의 여러 체크박스를 만들 수 있다. 그런데 문제는 이렇게 반복해서 HTML 태그를 생성할 때, 생성된 HTML 태그 속성에서 `name` 은 같아도 되지만, `id` 는 모두 달라야 한다. 따라서 타임리프는 체크박스를 `each` 루프 안에서 반복해서 만들 때 임의로 1 , 2 , 3 숫자를 뒤에 붙여준다.

[http://localhost:8080/form/items/add](http://localhost:8080/form/items/add) 실행 결과, 페이지 소스 보기

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f611cb22-afd4-439a-bd34-6a23d88d0270/Untitled.png)

```html
<!--        multi checkbox-->
<div>
    <div>등록 지역</div>
    <div class="form-check form-check-inline">
        <input type="checkbox" value="SEOUL" class="form-check-input" id="regions1" name="regions"><input type="hidden" name="_regions" value="on"/>
        <label for="regions1" class="form-check-label">서울</label>
    </div>
    <div class="form-check form-check-inline">
        <input type="checkbox" value="BUSAN" class="form-check-input" id="regions2" name="regions"><input type="hidden" name="_regions" value="on"/>
        <label for="regions2" class="form-check-label">부산</label>
    </div>
    <div class="form-check form-check-inline">
        <input type="checkbox" value="JEJU" class="form-check-input" id="regions3" name="regions"><input type="hidden" name="_regions" value="on"/>
        <label for="regions3" class="form-check-label">제주</label>
    </div>
</div>
```

위처럼 `region1`, `region2`, `region3` 으로 id 가 만들어집니다.

그렇다면 우리가 상품등록을 할 때 체크를 했을 때 정상적으로 동작하는지 로그를 찍어봅시다.

`FormItemController.addItem()` 에 코드 추가

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    log.info("item.open={}", item.getOpen());
    log.info("item.regions={}", item.getRegions());
    return "redirect:/form/items/{itemId}";
}
```

### 지역 서울, 부산, 제주 선택 시

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b30446f-5e68-41eb-820b-5980921aa0e0/Untitled.png)

로그 메시지

h.i.web.form.FormItemController : **item.open=false**

h.i.web.form.FormItemController : **item.regions=[SEOUL, BUSAN, JEJU]**

### 지역 선택을 하지 않았을 때

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4bfca08c-a8e7-4c07-96c5-7aaf56319aa6/Untitled.png)

h.i.web.form.FormItemController          : **item.open=false**
h.i.web.form.FormItemController          : **item.regions=[]**

`_regions` 는 앞서 설명한 기능입니다. 웹 브라우저에서 체크를 하나도 하지 않았을 때, 클라이언트가 서버에 아무런 데이터를 보내지 않는 것을 방지합니다. 참고로 `_regions` 조차 보내지 않으면 결과는 `null` 이 됩니다.

`_regions` 가 체크박스 숫자만큼 생성될 필요는 없지만, 타임리프가 생성되는 옵션 수 만큼 생성해서 그런 것이니 무시해도 됩니다.

`item.html` - 추가

```html
<!--        multi checkbox-->
<div>
    <div>등록 지역</div>
    <div th:each="region: ${regions}" class="form-check form-check-inline">
        <input type="checkbox" th:field="${item.regions}" th:value="${region.key}" class="form-check-input" disabled>
        <label th:for="${#ids.prev('regions')}" th:text="${region.value}" class="form-check-label"> 서울 </label>
    </div>
</div>
```

주의: `item.html` 에는 `th:object` 를 사용하지 않았기 때문에 `th:field` 부분에 `${item.regions}` 으로 적어주어야 한다. `disabled` 를 사용해서 상품 상세에서는 체크 박스가 선택되지 않도록 했다

### 타임리프의 체크 확인 `checked="checked"`

멀티 체크 박스에서 등록 지역을 선택해서 저장하면, 조회시에 `checked` 속성이 추가된 것을 확인할 수 있다. 타임리프는 `th:field` 에 지정한 값과 `th:value` 의 값을 비교해서 체크를 자동으로 처리해준다.

`editForm.html` - 추가 ( `addForm.html` 과 같다.)

```html
<!--        multi checkbox-->
<div>
    <div>등록 지역</div>
    <div th:each="region: ${regions}" class="form-check form-check-inline">
        <input type="checkbox" th:field="*{regions}" th:value="${region.key}" class="form-check-input">
        <label th:for="${#ids.prev('regions')}" th:text="${region.value}" class="form-check-label"> 서울 </label>
    </div>
</div>
```


# 8. 라디오 버튼

라디오 버튼은 여러 선택지 중에 하나를 선택할 때 사용할 수 있다. 이번시간에는 라디오 버튼을 자바 ENUM을 활용해서 개발해보자.

- 상품 종류
    - 도서, 식품, 기타
    - 라디오 버튼으로 하나만 선택할 수 있다.

`FormItemController` - 추가

```java
@ModelAttribute("itemTypes")
public ItemType[] itemTypes(){
    return ItemType.values();
}
```

`itemTypes` 를 등록 폼, 조회, 수정 폼에서 모두 사용하므로 `@ModelAttribute` 의 특별한 사용법을 적용합니다.

`ItemType.values()` 를 사용하면 해당 `ENUM`의 모든 정보를 배열로 반환한다. 예) `[BOOK, FOOD,ETC]`

상품 등록 폼에 기능을 추가해보자.

`addForm.html` - 추가

```html
<!--        radio button-->
<div>
    <div>상품 종류</div>
    <div th:each="type: ${itemTypes}" class="form-check form-check-inline">
        <input type="radio" th:field="*{itemType}" th:value="${type.name()}" class="form-check-input">
        <label th:for="${#ids.prev('itemType')}" th:text="${type.description}" class="form-check-label">BOOK</label>
    </div>
</div>
```

실행 결과, 폼 전송

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d900b00b-2c05-4e90-af0c-992f03de7582/Untitled.png)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b84ca86a-0f59-4d51-92fe-d512906d72b7/Untitled.png)

```html
itemType=BOOK // 도서 선택, 선택하지 않으면 아무 값도 넘어가지 않는다.
```

로그 추가

```java
@PostMapping("/add")
public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {
    Item savedItem = itemRepository.save(item);
    redirectAttributes.addAttribute("itemId", savedItem.getId());
    redirectAttributes.addAttribute("status", true);
    log.info("item.open={}", item.getOpen());
    log.info("item.regions={}", item.getRegions());
    log.info("item.itemType={}", item.getItemType());

    return "redirect:/form/items/{itemId}";
}
```

실행 로그

```html
item.itemType=BOOK: 값이 있을 때
item.itemType=null: 값이 없을 때
```

체크 박스는 수정시 체크를 해제하면 아무 값도 넘어가지 않기 때문에, 별도의 히든 필드로 이런 문제를 해결했다.

라디오 버튼은 이미 선택이 되어 있다면, 수정시에도 항상 하나를 선택하도록 되어 있으므로 체크 박스와 달리 별도의 히든 필드를 사용할 필요가 없다.

상품 상세와 수정에도 라디오 버튼을 넣어주자.

`item.html`

```html
<!--    radio button-->
<div>
    <div>상품 종류</div>
    <div th:each="type: ${itemTypes}" class="formcheck form-check-inline">
        <input type="radio" th:field="${item.itemType}" th:value="${type.name()}" class="form-check-input" disabled>
        <label th:for="${#ids.prev('itemType')}" th:text="${type.description}" class="form-check-label">BOOK</label>
    </div>
</div>
```

주의: `item.html` 에는 `th:object` 를 사용하지 않았기 때문에 `th:field` 부분에 `${item.itemType}` 으로 적어주어야 한다.

`disabled` 를 사용해서 상품 상세에서는 라디오 버튼이 선택되지 않도록 했다.

`editForm.html`

```html
<!--        radio button-->
<div>
    <div>상품 종류</div>
    <div th:each="type: ${itemTypes}" class="form-check form-check-inline">
        <input type="radio" th:field="*{itemType}" th:value="${type.name()}" class="form-check-input">
        <label th:for="${#ids.prev('itemType')}" th:text="${type.description}"
               class="form-check-label">BOOK</label>
    </div>
</div>
```

선택한 식품( `FOOD` )에 `checked="checked"` 가 적용된 것을 확인할 수 있다.

### 타임리프에서 ENUM 직접 사용하기

위애서 만든 `FormItemController` 의 `itemTypes()` 메서드에서처럼 모델에 ENUM을 담아서 전달하는 대신에 타임리프는 자바 객체에 직접 접근할 수 있다.

**타임리프에서 ENUM 직접 접근**

```html
<div th:each="type : ${T(hello.itemservice.domain.item.ItemType).values()}">
```

스프링EL 문법으로 `ENUM`을 직접 사용할 수 있다. `ENUM`에 `values()` 를 호출하면 해당 `ENUM`의 모든 정보가 배열로 반환된다.

그런데 이렇게 사용하면 `ENUM`의 패키지 위치가 변경되거나 할때 자바 컴파일러가 타임리프까지 컴파일 오류를 잡을 수 없으므로 추천하지는 않는다.


# 9. 셀렉트 박스

셀렉트 박스는 여러 선택지 중에 하나를 선택할 때 사용할 수 있다. 이번시간에는 셀렉트 박스를 자바 객체를 활용해서 개발해보자.

- 배송 방식
    - 빠른 배송
    - 일반 배송
    - 느린 배송
    - 셀렉트 박스로 하나만 선택할 수 있다.

`FormItemController` - 추가

```java
@ModelAttribute("deliveryCodes")
public List<DeliveryCode> deliveryCodes() {
    List<DeliveryCode> deliveryCodes = new ArrayList<>();
    deliveryCodes.add(new DeliveryCode("FAST", "빠른 배송"));
    deliveryCodes.add(new DeliveryCode("NORMAL", "일반 배송"));
    deliveryCodes.add(new DeliveryCode("SLOW", "느린 배송"));
    return deliveryCodes;
}
```

`DeliveryCode` 라는 자바 객체를 사용하는 방법으로 진행하겠다.

`DeliveryCode` 를 등록 폼, 조회, 수정 폼에서 모두 사용하므로 `@ModelAttribute` 의 특별한 사용법을 적용하자.

참고: `@ModelAttribute` 가 있는 `deliveryCodes()` 메서드는 컨트롤러가 호출 될 때 마다 사용되므로 `deliveryCodes` 객체도 계속 생성된다. 이런 부분은 미리 생성해두고 재사용하는 것이 더 효율적이다.

`addForm.html` - 추가

```html
<!-- SELECT -->
<div>
    <div>배송 방식</div>
    <select th:field="*{deliveryCode}" class="form-select">
        <option value="">==배송 방식 선택==</option>
        <option th:each="deliveryCode : ${deliveryCodes}" th:value="${deliveryCode.code}"
                th:text="${deliveryCode.displayName}">FAST
        </option>
    </select>
</div>
```

[http://localhost:8080/form/items/add](http://localhost:8080/form/items/add) 실행 , 페이지 소스 보기(타임리프로 생성된 html)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e8dcc6d6-40d8-4513-bbab-674cef428157/Untitled.png)

```html
<!-- SELECT -->
<div>
    <div>배송 방식</div>
    <select class="form-select" id="deliveryCode" name="deliveryCode">
        <option value="">==배송 방식 선택==</option>
        <option value="FAST">빠른 배송</option>
        <option value="NORMAL">일반 배송</option>
        <option value="SLOW">느린 배송</option>
    </select>
</div>

<hr class="my-4">
```

주의: `item.html` 에는 `th:object` 를 사용하지 않았기 때문에 `th:field` 부분에 `${item.deliveryCode}` 으로 적어주어야 한다.

`disabled` 를 사용해서 상품 상세에서는 셀렉트 박스가 선택되지 않도록 했다.

상품 상세와 수정에도 셀렉트 박스를 넣어주자.

`item.html`

```html
<!-- SELECT -->
<div>
    <div>배송 방식</div>
    <select th:field="${item.deliveryCode}" class="form-select" disabled>
        <option value="">==배송 방식 선택==</option>
        <option th:each="deliveryCode : ${deliveryCodes}" th:value="${deliveryCode.code}"
                th:text="${deliveryCode.displayName}">FAST
        </option>
    </select>
</div>
<hr class="my-4">
```

주의: `item.html` 에는 `th:object` 를 사용하지 않았기 때문에 `th:field` 부분에 `${item.deliveryCode}` 으로 적어주어야 한다. 

`disabled` 를 사용해서 상품 상세에서는 셀렉트 박스가 선택되지 않도록 했다.

`editForm.html`

```html
<!-- SELECT -->
<div>
    <div>배송 방식</div>
    <select th:field="*{deliveryCode}" class="form-select">
        <option value="">==배송 방식 선택==</option>
        <option th:each="deliveryCode : ${deliveryCodes}" th:value="${deliveryCode.code}"
                th:text="${deliveryCode.displayName}">FAST
        </option>
    </select>
</div>
<hr class="my-4">
```

[http://localhost:8080/form/items/1/edit](http://localhost:8080/form/items/1/edit) 실행 (페이지 소스 보기 → 타임리프로 생성된 html)

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e7c19329-551c-42d2-87e3-cb86cf78a033/Untitled.png)

```html
<!-- SELECT -->
<div>
    <div>배송 방식</div>
    <select class="form-select" id="deliveryCode" name="deliveryCode">
        <option value="">==배송 방식 선택==</option>
        <option value="FAST">빠른 배송</option>
        <option value="NORMAL">일반 배송</option>
        <option value="SLOW">느린 배송</option>
    </select>
</div>
<hr class="my-4">
```

`selected="selected”`

빠른 배송을 선택한 예시인데, 선택된 셀렉트 박스가 유지되는 것을 확인할 수 있다.