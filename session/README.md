# Session

세션이란 서버에 중요한 정보를 보관하면서 연결을 유지하는 방법을 의미한다. 

이 저장소에서는 스프링이 제공하는 세션 기능과 스프링 시큐리티에서 세션 값을 관리하는 메커니즘에 대해 기술한다. 

스프링이 제공하는 세션 기능을 직접 만들어서 사용하는 것보다 스프링 시큐리티를 사용하는 것이 유용하기 때문에 

스프링 시큐리티에 관한 내용위주로 보는 것을 추천한다. 

### 세션동작 예시
```
사용자가 아이디와 패스워드로 인증에 성공하면 서버에서는 고유한 세션Id 를 생성해서 세션 저장소에 연결 정보(사용자) 를 보관한다.
그리고 서버에서 쿠키에 세션Id 값을 포함해서 클라이언트에 응답하면 웹 브라우저는 쿠키 저장소에 세션Id 쿠키를 보관한다.

(세션Id 는 추정 불가능한 UUID 를 사용한다.)

클라이언트는 애플리케이션에 요청할 때마다 쿠키에 세션 아이디값을 포함한 요청을 보내게 되고 애플리케이션 서버는
세션 아이디값을 이용해서 사용자를 식별할 수 있다.

세션을 사용하면 요청할 때마다 회원과 관련된 정보를 전달하지 않고 애플리케이션 서버에서 사용자를 식별할 수 있게 된다.
```
```
* 쿠키란?

쿠키 정보는 항상 서버에 전송된다. 무상태로 유지되는 HTTP 에 쿠키를 사용해서 세션 정보를 포함하면 애플리케이션 서버에서
쿠키에 포함된 세션 정보를 사용할 수 있다.

즉 클라이언트와 애플리케이션 서버는 쿠키로 연결된다. 

쿠키에는 중요 정보가 아닌 최소한의 정보만을 사용한다. 
```
## HttpSession 을 사용해서 간단한 세션 논리 적용하기

서블릿은 HttpSession 이라는 기능을 제공한다. HttpSession 으로 세션을 생성하면 JSESSIONID 라는 이름의 쿠키에 UUID 를 사용한 랜덤한 세션id 값을 생성해준다.

```
HttpSession 에 데이터를 보관하고 조회할 때 같은 이름이 중복되어 사용되므로 상수 하나를 정의했다.

pulbic class SessionConst{
  public static final String LOGIN_MEMBER = "loginMember";
}
```
```
* 세션을 이용한 간단한 로그인 처리 로직

@PostMapping("/login")
public String login(@Valid @ModelAttribute LoginForm form, BindingResult bindingResult, HttpServletRequest request){

  if(bindinResult.hasErrors()){ // 검증 실패시 로그인 화면으로
     return "login/loginForm";
  }

  Member loginMember = loginService.login(form.getLoginId(), form.getPassword());

  if(loginMember == null){
    bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
    return "login/loginForm";
  }

  // 로그인 성공
  // 세션이 있다면 있는 세션값을 반환, 없으면 신규 세션을 생성하는 메서드
  HttpSession session = request.getSession();
  // 세션에 로그인 회원 정보 보관
  session.setAttribute(SessionConst.LOGIN_Member, loginMember); 

   return "redirect:/";
}

request.getSession() 의 기본값은 true 이다 세션이 있으면 기존 세션을 반환하며 세션이 없으면 새로운 세션을 생성하고 반환한다.
false 를 사용하면 세션이 있을 때만 기존 세션을 반환하고 세션이 없으면 새로운 세션을 생성하지 않는다.
```
```
* 세션 로그아웃 구현

@PostMapping("/logout")
public String logout(HttpServletRequest request){

  //세션을 삭제
  HttpSession session = request.getSession(false);
  if(session != null{
    session.invalidate();
  }
  return "redirect:/";
}

session.invalidate() 를 사용해서 쿠키로 넘어온 세션 값을 삭제할 수 있다. 
```
```
* 홈 화면에서 세션 유지하기

@GetMapping("/")
public String homeLogin(HttpServletRequest request, Model model){

  // 세션이 없으면 home
  HttpSession session = request.getSession(false);
  if(session == null){
    return "home";
  }

  Member loginMember = (Member) session.getAttribute(SessionConst.LOGIN_MEMBER);
  // 세션에 회원 데이터가 없으면 home
  if (loginMember == null) {
     return "home";
  }

  // 세션이 유지되면 로그인으로 이동
  model.addAttribute("member", loginMember);
   return "loginHome";
  }
```

## HttpSession 을 사용해서 로그인 처리하기 2

앞서 살펴본 것보다 간편하게 세션값 처리하기 
```
* @SessionAttribute 을 사용해서 이미 로그인된 사용자 차기 

@GetMapping("/")
public String homeLogin(
        @SessionAttribute(name = SessionConst.LOGIN_MEMBER, required = false) Member loginMember, Model model){

  // 세션에 회원 데이터가 없으면 홈으로
  if(loginMember == null){
     return "home";
  }

 model.addAttribute("member", loginMember);
  return "loginHome";
```
```
* TrackingModes

jsessionid 를 url 로 노출하고 싶지 않다면 properties 에 다음 설정을 추가한다.
server.servlet.session.tracking-modes=cookie 
```
```
* 세션 타임아웃 설정하기

세션은 기본적으로 메모리에 생성되기 때문에 꼭 필요한 경우만 생성하고 메모리 관리를 위해 무한정 세션을
가지고 있을 수는 없다.

또한 세션이 탈취 당한 경우 세션 타임을 설정하지 않는다면 재사용의 위험성도 있다. 

세션 타임아웃 설정은 30 분을 기준으로 하며 최근 요청 기준으로 30 분 정도를 유지해주는 것이 좋다.
(HttpSession 은 기본적으로 이 방식을 사용함)

application.properties 세션 글로벌 설정
server.servlet.session.timeout=60 : 60초, 기본은 1800(30분)

session.setMaxInactiveInterval(1800); // 자바 설정

참고로 스프링 시큐리티를 사용할 경우 시큐리티 필터체인으로 설정하면 된다.
```

### 기타 세션 정보 출력 예시 
```
(HttpSession session)

session.getAttributeNames().asIterator()
 .forEachRemaining(name -> log.info("session name={}, value={}",
name, session.getAttribute(name)));
 log.info("sessionId={}", session.getId());
 log.info("maxInactiveInterval={}", session.getMaxInactiveInterval());
 log.info("creationTime={}", new Date(session.getCreationTime()));
 log.info("lastAccessedTime={}", new
Date(session.getLastAccessedTime()));
 log.info("isNew={}", session.isNew());

```

## 스프링 시큐리티에서 세션을 사용하기 

스프링 시큐리티 Authentication 에 성공하면 HttpSession 에 세션 값이 저장된다. 그리고 세션 쿠키르 가진 클라이언트의 요청이 

들어오면 인증 절차를 다시 거치지 않고 세션에 값이 있으면 SecurityContext 에서 Authentication 값을 그대로 사용할 수 있다. 

```
* 시큐리티에서 세션 값을 가져오는 메커니즘

스프링 시큐리티를 사용하면서 사용자 요청이 들어오면 항상 SecurityContextHolderFilter 를 거쳐간다. 
(SecurityContextPersistenceFilter -> HttpSecurityContextRepository 순으로 거쳐감)

인증하기 전 첫 요청이라면 Authentication 매커니즘을 수행하고 SecurityContext 에 Authentication 이 보관된다.

그리고 HttpSession 에 SecurityContext 값이 저장되며 스레드 요청이 끝나고 사용자 응답을 할 때 SecurityContextHolder 의
SecurityContext 값은 Clear() 된다.

(즉 세션에 SecuriytyContext 가 저장된 상태임)

인증후 쿠키에 세션값으로 SecurityContext 값을 같이 보낸다면 앞서 말한 SecurityContextHolderFilter 를 다시 거치고
SecurityContext 값이 있는지 확인 이때는 세션에 값이 있기때문에 Authentication 절차를 거치지 않고

세션에서 SecurityContext 값을 꺼내서 SecurityContextHolder 로 감싸고 이 값을 사용할 수 있다. 
```

```
* 스프링 시큐리티에서 HttpSession 값 사용 예시

(HttpSession session)

SecurityContext context = (SecurityContext) session
             .getAttribute(HttpSessionSecurityContextRepository.SPRING_SECURITY_CONTEXT_KEY);

Authentication authentication1 = context.getAuthentication();
System.out.println(authentication1.getPrincipal());

세션으로 전달되는 SecurityContext 값을 꺼내서 사용할 수 있다.

https://catsbi.oopy.io/f9b0d83c-4775-47da-9c81-2261851fe0d0 참고 
```
