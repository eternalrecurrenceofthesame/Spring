# HTTP Message Converter

뷰 템플릿으로 HTML 을 생성해서 응답하지 않고, JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리하다.

```
스프링 MVC 는 다음의 경우 HTTP 메시지 컨버터를 적용한다

HTTP 요청: @RequestBody, HttpEntity(RequestEntity)
HTTP 응답: @ResponseBody, HttpEntity(ResponseEntity)
```
```
* 사용 예시

- @RequestBody

public HelloData requestBodyJson(@RequestBody HelloData data) {
// @RequestBody 는 생략할 수 없으며 생략하면 @ModelAttribute 가 자동으로 적용된다. 

- @ResponseBody

 @ResponseBody // @RestController 를 사용하면 자동으로 포함된다. 
 @GetMapping("/response-body-string-v3")
 public String responseBodyV3() {
 return "ok";}

 @ResponseStatus(HttpStatus.OK)
 @ResponseBody
 @GetMapping("/response-body-json-v2")
 public HelloData responseBodyJsonV2() {
 HelloData helloData = new HelloData();
 helloData.setUsername("userA");
 helloData.setAge(20);
 return helloData;}


@RequestBody(consumes = MediaType.APPLICATION_JSON_VALUE) 
@ResponseBody(consumes = MediaType.APPLICATION_JSON_VALUE)

미디어 타입을 따로 지정할 수도 있다. 
```
## 스프링 부트가 제공하는 기본 메시지 컨버터
```
0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter 
2 = MappingJackson2HttpMessageConverter
```
## 메시지 컨버터가 동작하는 위치 

RequestMappingHandlerAdapter -> ArgumentResolver(HTTP message converter) -> Controller -> ReturnValueHandler(HTTP message Converter) -> 

```
* ArgumentResolver

애노테이션 기반 컨트롤러를 처리하는 RequestMappingHandlerAdapter 는 @RequestParam, @ModelAttribute, @RequestBody HttpEntity 등
다양한 애노테이션을 다루는 ArgumentResolver 를 호출한다. Resolver 는 핸들러가 필요로 하는 파라미터 객체를 생성해서 핸들러를 호출한다.
```
```
* ReturnValueHandler

ModelAndView, @ResponseBody, HttpEntity 등의 응답 값을 변환하고 처리한다. 
```

@RequestBody 와 HttpEntity 를 처리하는 ArgumentResolver 는 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하며 마찬가지로 

@ResponseBody, HttpEntity 를 처리하는 ReturnValueHandler 도 HTTP 메시지 컨버터를 사용해서 반환 값을 생성한다.

## Bean Validation 과 HTTP 메시지 컨버터

API 의 경우 3 가지를 나누어서 생각해야 한다.
```
성공 요청: 성공
실패 요청: JSON 객체를 생성하는 것 자체를 실패하는 경우
검증 오류 요청: JSON 객체로 생성하는 것에는 성공했지만 검증에서 실패하는 경우
```
* 실패 요청

메시지 컨버터에서 요청 JSON 을 객체로 만드는 것에 실패하면 컨트롤러가 호출되지 않고 validation 이 실행되지 않는다. 

* 검증 오류 요청

메시지 컨버터는 객체를 JSON 으로 변환해서 클라이언트에 전달할 때 JSON 에 검증 실패 메시지가 포함된다. 

검증 오류 객체를 만들 때는 필요한 데이터만 별도의 API 스펙으로 정의해서 전달해야 한다.  교재 참고 

### @ModelAttribute 와 @RequestBody

@ModelAttribute 는 바인딩에 실패하더라도 정상 바인딩된 필드는 Validator 를 사용해서 검증할 수 있다

@RequestBody 는 메시지 컨버터가 제이슨 데이터를 객체로 변경하지 못하면 이후 단계가 진행되지 않고 예외가 발생한다. 
