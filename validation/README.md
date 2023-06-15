# Validation 

스프링 프레임워크의 검증 관련 사항을 정리 

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

th:errors="field-error" 해당 필드에 오류가 있으면 태그를 출력한다.
th:errorclass="field-error"  th:field 에서 지정한 필드 오류가 있으면 class 정보를 추가한다.
```
### 사용자가 입력한 값을 화면에서 다시 보여주기




#### + Safe Navigation Operator 사용하기
```
타임리프로 값을 보여줄 때 모델 값을 호출했는데 값이 비어 있다면 NPE 가 발생한다. 타임리프로 넘겨진 모델의 필드 값을
호출하기 전 앞에 ? 를 붙이면 NPE 대신 null 값을 반환한다.

th:if="${모델?.containsKey('필드 값')
이렇게 하면 조건에 따라 값이 없으면 null 이 반환되고  th:if 에서 null 은 실패로 처리되므로 값을 출력하지 않는다. 
```
