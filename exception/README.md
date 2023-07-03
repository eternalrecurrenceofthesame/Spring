# 스프링 부트가 제공하는 오류 페이지 사용하기

스프링 부트는 resource/templates/error 경로를 제공한다. 이 경로에 오류 템플릿을 등록하면 오류 유형에 맞는 템플릿이 호출된다.

```
* 뷰 선택 우선 순위

1. 템플릿
resources/templates/error/500.html
resources/templates/error/5xx.html

2. 정적 리소스 (static, public)
resources/static/error/400.html
resources/static/error/4xx.html

3. 적용 대상이 없을 때는 뷰 이름 (error 를 호출)
resource/templates/error.html

구체적인 것이 덜 구체적인 것보다 우선적으로 적용된다 500 > 5xx
```

## 스프링 부트가 제공하는 BasicErrorController 기본 기능 사용하기 

BasicErrorController 는 아래 정보를 model 에 담아서 뷰에 전달하고 뷰에서 값을 보여줄 수 있다.

```
th:text="|timestamp: ${timestamp}|"
th:text="|path: ${path}|"
th:text="|status: ${status}|"
th:text="|message: ${message}|"
th:text="|error: ${error}|"
th:text="|exception: ${exception}|"
th:text="|errors: ${errors}|"
th:text="|trace: ${trace}|"
```
```
* BasicErrorController 옵션

server.error.include-exception=false : exception 포함 여부( true , false )
server.error.include-message=never : message 포함 여부
server.error.include-stacktrace=never : trace 포함 여부
server.error.include-binding-errors=never : errors 포함 여부

기본 값이 never 인 부분은 never, always, on_param(파라미터가 있으면 사용) 3 가지 옵션을 사용할 수 있다.
```

보통 오류가 발생하면 BasicErrorController 에서 제공하는 값들을 보여주는 것은 고객에게 혼란을 줄 수 있기 때문에 노출하지 말고 직접 만든 오류 템플릿을 보여줘야 한다.

이러한 오류 값들은 개발 서버에서 로그로 남겨서 로그로 확인해야 한다.

```
* 그 외 오류 옵션 

server.error.whitelabel.enabled=true - 오류 페이지를 찾지 못하면 whitelabel 적용
server.error.path=/error - 오류 페이지 경로 설정 이 경로는 BasicErrorController 와 함께 사용된다.

ErrorController, BasicErrorController 를 상속받아서 공통 처리 컨트롤러를 확장할 수도 있다.  
```

# API 예외 처리하기
```
템플릿을 사용할 때는 BasicErrorController 를 사용해서 오류 화면을 호출하면 되지만 API 는 각 시스템 마다 응답 모양이
다르고 스펙도 모두 다르기 때문에 예외 상황에서 단순히 오류 화면을 출력하는 것이 아니라 예외에 따라서

각각 다른 데이터를 출력해야 할 수도 있다. 같은 예외라도 컨트롤러에서 발생했는가에 따라 다른 예외 데이터 응답이 필요하다.
스프링은 API 예외를 처리하는 편리한 기능을 제공한다.
```
## 스프링이 제공하는 기능으로 API 응답하기 

### 1. API 응답 객체 만들기 
```
* 예외 발생시 API 응답으로 사용할 객체

@Data
@AllArgsConstructor
public class ErrorResult{
  private String code;
  private String message;
}
```
### 2. @ExceptionHandler 로 예외 상황에 맞는 응답하기
```
@Slf4j
@RestControllerAdvice // @ControllerAdvice + @ResponseBody
public class ExControllerAdvice {

    @ResponseStatus(HttpStatus.BAD_REQUEST)
    @ExceptionHandler(IllegalArgumentException.class)
    public ErrorResult illegalExHandle(IllegalArgumentException e){
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("BAD", e.getMessage());
    }

    @ExceptionHandler
    public ResponseEntity<ErrorResult> userExHandle(ExecutionControl.UserException e){
        log.error("[exceptionHandle] ex", e);
        ErrorResult errorResult = new ErrorResult("USER-EX", e.getMessage());
        return new ResponseEntity<>(errorResult, HttpStatus.BAD_REQUEST);
    }

    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    @ExceptionHandler
    public ErrorResult exHandle(Exception e){
        log.error("[exceptionHandle] ex", e);
        return new ErrorResult("EX", "내부 오류");
    }
}
```
```
* @ExceptionHandler

@ExceptionHandler 를 사용하면 부모 클래스뿐만 아니라 자식 클래스 예외까지 잡을 수 있으며 다수의 예외를
처리할 수도 있다. (부모 자식 두 가지 예외가 있으면 구체적인 자식이 우선순위를 가짐)

@ExceptionHandler 의 예외를 생략할 수 있고 생략시 파라미터값을 사용한다.
```
```
* @ControllerAdvice

어드바이스에 대상을 지정하지 않으면 글로벌 설정을 가지며 모든 컨트롤러에 적용된다. 특정 패키지나 클래스에만
적용할 수도 있다.

@ControllerAdvice(annotations = RestController.class)
@ControllerAdvice("org.example.controllers")
@ControllerAdvice(assignableTypes = {ControllerInterface.class,AbstractController.class})
```
















