# Validation 

## BindingResult 사용하기
```
스프링이 제공하는 검증 오류를 보관하는 객체 검증 오류가 발생하면 여기에 보관된다. 표현 영역에서 BindingResult 를
사용하면 @ModelAttribute 로 모델 값에 데이터를 매핑할 때 오류가 발생하더라도 오류 페이지를 호출하는 것이 아니라

오류 정보를 BindingResult 에 담아서 컨트롤러를 정상적으로 호출한다.

BindingResult 를 컨트롤러에서 사용할 때는 매핑되는 모델 즉 @ModelAttribute 의 뒤에 파라미터를 위치시켜야 한다!
(@ModelAttribute Item item, BindingResult bindingResult, ...)

- 글로벌 오류 생성 예시 

if (item.getPrice() != null && item.getQuantity() != null) {
 int resultPrice = item.getPrice() * item.getQuantity();
 if (resultPrice < 10000) {
 bindingResult.addError(new ObjectError("item", "가격 * 수량의 합은
10,000원 이상이어야 합니다. 현재 값 = " + resultPrice));
 }}

- 필드 오류 생성 예시 

if (!StringUtils.hasText(item.getItemName())) {
 bindingResult.addError(new FieldError("item", "itemName", "상품 이름은 필수입니다."));}
```
```
* 타임리프의 오류 처리 예시

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
```
