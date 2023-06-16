# Validation 

## BindingResult 사용하기
```
스프링이 제공하는 검증 오류를 보관하는 객체 검증 오류가 발생하면 여기에 보관된다. 표현 영역에서 BindingResult 를
사용하면 @ModelAttribute 로 모델 값에 데이터를 매핑할 때 오류가 발생하더라도 오류 페이지를 호출하는 것이 아니라

오류 정보를 BindingResult 에 담아서 컨트롤러를 정상적으로 호출한다.

BindingResult 를 컨트롤러에서 사용할 때는 매핑되는 모델 즉 @ModelAttribute 의 뒤에 파라미터를 위치시켜야 한다!
(@ModelAttribute Item item, BindingResult bindingResult, ...)
```
### 타임리프를 사용해서 오류 처리하기 
```
- 글로벌 오류

<div th:if="${#fields.hasGlobalErrors()}">
 <p class="field-error" th:each="err : ${#fields.globalErrors()}" th:text="$
{err}">전체 오류 메시지</p>
</div>

th:if="${#fields.hasGlobalErrors()}" #fields 로 BindingResult 가 제공하는 검증 오류에 접근할 수 있다.

- 필드 오류

<input type="text" id="itemName" th:field="*{itemName}"
 th:errorclass="field-error" class="form-control" placeholder="이름을 입력하세요">
<div class="field-error" th:errors="*{itemName}">
 상품명 오류
</div>

th:errors= 해당 필드에 오류가 있으면 태그를 출력한다.
th:errorclass=  th:field 에서 지정한 필드 오류가 있으면 class 정보를 추가한다.
```
### 사용자가 입력한 값을 화면에서 다시 보여주기

사용자 입력 오류가 발생하더라도 기존에 입력한 데이터를 다시 보여주면서 오류가 발생한 곳을 알려주면 간편하게 입력 값을 수정할 수 있다.

```
* 필드 에러 오류 저장 예시

bindingResult.addError(new FieldError("item", "quantity", item.getQuantity(), false, null, null,
"수량은 최대 9,999 까지 허용합니다."));

public FieldError(String objectName, String field, @Nullable Object 
rejectedValue, boolean bindingFailure, @Nullable String[] codes, @Nullable
Object[] arguments, @Nullable String defaultMessage)

objectNmae: 오류가 발생한 객체 이름 (@ModelAttribute 로 넘어오는 파라미터 이름)
field: 파라미터에서 오류가 발생한 필드 이름
rejectedValue: 사용자가 입력한 값(거절된 값을 다시 넣어준다.)
bindingFailure: 타입 오류 같은 바인딩 실패인지, 검증 실패인지 구분하는 값 (바인딩 실패면 true)
codes: 메시지 코드
arguments: 메시지에서 사용하는 인자
defaultMessage: 기본 오류 메시지


오류가 발생하면 bindingResult 에 오류 값을 담아서 컨트롤러를 호출한다. 이때 타임리프의
th:field="*{}" 는 오류가 발생하면 FieldError 에서 보관한 값을 사용해서 값을 출력한다. 
```

## 오류 코드와 메시지 처리하기
```
* appliation.properties

spring.messages.basename=messages,errors 오류 메시지 처리를 위해 국제화에 더해서 errors 를 추가한다. 
(messages.properties 로 오류 메시지를 처리할 수도 있음) 

errors.properties 에서 오류 메시지를 처리할 수 있다!
(참고로 errors_en.properties 파일을 생성하면 오류 메시지도 국제화 처리할 수 있다.)
```
```
* errors.properties 예시 

required.item.itemName = 상품 이름은 필수입니다.
range.item.price = 가격은 {0}  ~ {1} 까지 허용합니다.
max.item.quantity=수량은 최대 {0} 까지 허용합니다.
totalPriceMin=가격 * 수량의 합은 {0}원 이상이어야 합니다. 현재 값 = {1}
```
### BindingResult 가 제공하는 rejectValue(), reject() 사용하기 

BindingResult 는 검증하는 객체인 target 다음에 위치하며 검증해야할 target 객체를 인지하고 있다. rejectValue(), reject() 를 사용하면 FieldError, ObjectError 를 직접 생성하지 않고, 깔끔하게 검증 오류를 다룰 수 있다.

```
bindingResult.rejectValue("price", "range", new Object[]{1000, 100000}, null); // 필드 오류 만들기
bindingResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null); // 글로벌 오류 만들기

target 객체를 인지하고 있기 때문에 따로 값을 넣어주지 않고 최소한의 정보만으로 오류 값을 넘겨줄 수 있다.


* rejectValue 구성 
void rejectValue(@Nullable String field, String errorCode,
@Nullable Object[] errorArgs, @Nullable String defaultMessage);

field: 오류 필드명
errorCode: 오류 코드 (메시지에 등록된 오류 코드가 아닌 messageesolver 를 위한 오류 코드)
errorArgs: 오류 메시지에서 {0} 을 치환하기 위한 값
defaultMessage: 오류 메시지를 찾을 수 없을 때 사용하는 기본 값 
```
## 오류 코드와 메시지 처리 2 
```
required.item.itemName: 상품 이름은 필수입니다.
required: 필수 값입니다.

오류 메시지를 생성할 때 코드와 타깃 필드 값을 사용해서 구체적으로 만들 수도 있지만 단순 코드만으로도
생성할 수 있다. 
```
### MessageCodesResolver 와 오류 코드 관리 

reject(), rejectValue() 메서드는 내부에서 MessageCodesResolver 를 사용해서 메시지 코드를 생성한다. 
```
* 생성 예시

rejectValue("itemName", "required")

required.item.itemName
required.itemName
required.java.lang.String
required

reject("totalPriceMin")

totalPriceMin.item
totalPriceMin

타임리프 화면을 랜더링할 때 th:errors 가 실행되고 오류 메시지가 있으면 코드를 순서대로 돌아가면서
메시지를 찾는다.
```
메시지 코드 리졸버는 구체적인 것을 먼저 만들고 덜 구체적인 것을 가장 나중에 만들기 때문에 생성한 메시지를 재활용할 수 있다. 

## 오류 코드와 메시지 처리 3
```
검증 오류 코드는 2 가지로 나눌 수 있다.

개발자가 직접 설정한 오류 코드 -> rejectValue 를 직접 호출하는 경우
스프링이 직접 검증 오류에 추가한 경우(주로 타입 미스 매치)

스프링은 타입 오류가 발생하면 typeMistMatch 라는 오류 코드를 사용한다.

typeMisMatch.item.price
typeMistMatch.price
typeMistMatch.java.lang.Integer
typeMismatch

typeMisMatch.java.lang.Integer = 숫자를 입력해주세요.
typeMistMatch = 타입 오류 입니다. 
```

#### + Safe Navigation Operator 사용하기
```
타임리프로 값을 보여줄 때 모델 값을 호출했는데 값이 비어 있다면 NPE 가 발생한다. 타임리프로 넘겨진 모델의 필드 값을
호출하기 전 앞에 ? 를 붙이면 NPE 대신 null 값을 반환한다.

th:if="${모델?.containsKey('필드 값')
이렇게 하면 조건에 따라 값이 없으면 null 이 반환되고  th:if 에서 null 은 실패로 처리되므로 값을 출력하지 않는다.

https://docs.spring.io/spring-framework/docs/current/reference/html/
core.html#expressions-operator-safe-navigation 참고
```

# Bean Validation 

Bean Validation 이 제공하는 검증 애노테이션을 사용하면 검증 논리를 구현하는 대신 애노테이션 하나로 쉽게 검증할 수 있다.

```
하이버네이트 Validator 링크 (하이버네이트지만 ORM 과는 관련 없음)

공식 사이트: http://hibernate.org/validator/
공식 메뉴얼: https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/
검증 애노테이션 모음: https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single
/#validator-defineconstraints-spec
```
## Bean Validation 사용하기
```
* 검증 순서

@ModelAttribute 각각의 필드에 타입 변환을 시도하고 성공하면 Validator 를 적용한다 실패하면
typeMisMatch 로 FieldError 를 추가한다.

즉 바인딩에 성공한 필드값만 Bean Validation 이 적용된다.


파라미터 사용 예시

public String addItem(@Validated @ModelAttribute Item item, BindingResult bindingResult,
RedirectAttributes redirectAttributes)
```
### Bean Validation 이 제공하는 에러 코드 사용하기 

앞서 에러 코드로 errors.properties 메시지를 보여준 것처럼 Bean Validation 도 에러 코드로 메시지를 화면에 보여줄 수 있다.

```
* 에러 코드 예시

@NotBlank
NotBlank.item.itemName
NotBlank.itemName
NotBlank.java.lang.String
NotBlank

애노테이션이 에러 코드가되고 타깃과 필드 값을 가진다.
```
```
* Bean Validation 메시지 등록과 스캔 순서 

NotBlank={0} 공백 X
Range={0}, {2} ~ {1} 허용
Max={0}, 최대 {1}

빈 밸리데이션에서 {0} 은 필드명을 의미한다. {1},{2} 는 각 애노테이션마다 다를 수 있다.

1. 빈 밸리데이션은 코드를 바탕으로 생성된 메시지를 먼저 메시지 소스에서 찾는다.
2. 애노테이션 message 속성을 사용하면 쉽게 메시지를 생성할 수 있다. @NotBlank(message = "공백! {0}")
3. 라이브러리가 제공하는 기본 값을 사용한다.
```
```
* Bean Validation 오브젝트 오류 처리하기

bidningResult.reject("totalPriceMin", new Object[]{10000, resultPrice}, null);
빈 밸리데이션을 사용하더라도 글로벌 오류는 reject 메서드를 사용하는 것이 편리하다! 
```
## Bean Validation 의 한계 

데이터를 등록할 때와 수정할 때의 요구 사항이 다를 수 있고 그에 따른 검증 구현이 달라질 수 있다. 등록시에는 최대 수량에 제한이 있지만 수정할 때는 최대 수량 제한이 없다던지 등록할 때는 id 에 값이 없어도 되지만 수정할 때는 id 값이 필수로 사용될 수 있다. 

### @Validated 가 제공하는 Groups 사용하기 

보통 Groups 는 잘 사용되지 않는다. 실무에서는 주로 등록용 폼 객체와 수정용 폼 객체를 분리해서 사용한다. 

### Form 전송 객체를 분리하기

Groups 가 사용되지 않는 다른 이유로는 등록시 폼에서 전달하는 데이터가 저장할 도메인 객체와 딱 맞지 않는 것도 있다. 회원을 등록한다고 했을 때 회원과 관련된 데이터만 폼에서 전달하는 것이 아니라 약관 정보나 부가 데이터도 같이 넘어오게 된다. 

그렇기 떄문에 전송 객체를 각각 생성하고 @ModelAttribute 로 값을 받을 때 전송 객체별로 Validation 을 진행해야 한다. 참고로 등록용 수정용 뷰 템플릿이 비슷하다고 해도 모델을 하나로 합쳐서 사용하는 것은 추천하지 않는다. 예제는 교재를 참고한다. 









