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
