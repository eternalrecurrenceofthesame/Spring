# Session

세션이란 서버에 중요한 정보를 보관하면서 연결을 유지하는 방법을 의미한다. 

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



```
