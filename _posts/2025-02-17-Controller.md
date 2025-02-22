---
title: "[Spring MVC] @Controller, @RequestMapping"
excerpt: "컨트롤러 정리"

categories: # 카테고리 설정
  - Web
tags: # 포스트 태그
  - [tag1, tag2]

permalink: /Web/controller/ # 포스트 URL

toc: true # 우측에 본문 목차 네비게이션 생성
toc_sticky: true # 본문 목차 네비게이션 고정 여부

date: 2025-02-17 # 작성 날짜
last_modified_at: 2025-02-17 # 최종 수정 날짜
---

기존에 사용하던 컨트롤러를 스프링 MVC의 기능을 적용하여 개선해 볼 것이다.

본격적으로 애노테이션 기반의 컨트롤러를 사용하기 전에 스프링MVC에서 컨트롤러를 만들 때 필요한`@Controller`, `@RequestMapping`에 대해 알아보고 예제 코드를 살펴볼것이다.

### @Controller
- 스프링이 자동으로 스프링 빈으로 등록한다.(내부에 `@Component`가 있기때문에 컴포넌트 스캔의 대상)
- 스프링 MVC에서 애노테이션 기반의 컨트롤러로 인식한다.

### @RequestMapping
- 요청정보를 매핑하여 해당 URL이 호출되면 이 메서드가 호출된다.
- 애노테이션을 기반으로 동작하기 때문에 메서드 이름은 임의로 작성 가능하다.

---
## SpringMemberFormControllerV1
### 회원 등록
```java
@Controller
public class SpringMemberFormControllerV1 {

    @RequestMapping("/springmvc/v1/members/new-form")
    public ModelAndView process() {
        return new ModelAndView("new-form");
    }
}
```
`ModelAndVeiw`는 모델과 뷰 정보를 담아서 반환한다.

### 회원 저장
```java
@Controller
public class SpringMemberSaveControllerV1 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members/save")
    public ModelAndView process(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;
    }
}
```
스프링이 제공하는 `ModelAndView`을 통해 데이터를 추가할 경우 `addObject()`사용한다.
이 데이터는 이후 뷰를 렌더링 할 때 이용된다.

### 회원 리스트
```java
@Controller
public class SpringMemberListControllerV1 {

    private final MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/springmvc/v1/members")
    public ModelAndView process() {

        List<Member> members = memberRepository.findAll();

        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);
        return mv;
    }
}
```

위의 예제 코드들을 살펴보면 스프링 MVC를 이용하여 애노테이션 기반의 컨트롤러를 사용하면 훨씬 편하게 코드 작성이 가능함을 알 수 있지만 `@RequestMapping`이 클래스 단위가 메서드 단위에 적용된 것을 알 수 있다. 따라서 이를 개선하여 컨트롤러를 하나의 클래스로 통합할 것이다.

---

## SpringMemberControllerV2

```java
@Controller
@RequestMapping("/springmvc/v2/members")
public class SpringMemberControllerV2 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @RequestMapping("/new-form")
    public ModelAndView newForm() {
        return new ModelAndView("new-form");
    }

    @RequestMapping("/save")
    public ModelAndView save(HttpServletRequest request, HttpServletResponse response) {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));

        Member member = new Member(username, age);
        memberRepository.save(member);

        ModelAndView mv = new ModelAndView("save-result");
        mv.addObject("member", member);
        return mv;
    }

    @RequestMapping
    public ModelAndView members() {

        List<Member> members = memberRepository.findAll();

        ModelAndView mv = new ModelAndView("members");
        mv.addObject("members", members);
        return mv;
    }
}
```
`@RequestMapping`가 클래스 레벨과 메서드 레벨에 조합되어 3개의 클래스가 하나의 클래스로 통합된것을 확인 할 수 있다.
V1에 비해 간결해진 코드를 확인 할 수 있으나 지난 강의에서 MVC 프레임워크를 만들어서 개선했던 것 처럼 `ModelAndView`를 매번 생성하고 반환하는것이 아니라 실무에서 이용하는 방식에 가깝도록 코드를 개선해볼 것이다.

---

## SpringMemberControllerV3

```java
@Controller
@RequestMapping("/springmvc/v3/members")
public class SpringMemberControllerV3 {

    private MemberRepository memberRepository = MemberRepository.getInstance();

    @GetMapping("/new-form")
    public String newForm() {
        return "new-form";
    }

    @PostMapping("/save")
    public String save(
            @RequestParam("username") String username,
            @RequestParam("age") int age,
            Model model) {

        Member member = new Member(username, age);
        memberRepository.save(member);

        model.addAttribute("member", member);
        return "save-result";
    }

    @GetMapping
    public String members(Model model) {

        List<Member> members = memberRepository.findAll();

        model.addAttribute("members", members);
        return "members";
    }
}
```
- Model 도입
- String을 이용하여 ViewName 직접 반환
	
    - ModelAndView를 매번 생성하지않아도된다.
- `@RequestParam` 도입
	
    - 스프링은 HTTP 요청 파라미터를 `@RequestParam`으로 받을 수 있다.
    - 이는 이전 버전에서 사용하던 `request.getParameter()`와 비슷한 코드라고 생각하면 된다.
- `@RequestMapping` -> `@PostMapping`, `@GetMapping` 이용




