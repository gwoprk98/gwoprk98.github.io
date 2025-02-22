---
published: true
title: "[Spring] @Configuration, 싱글톤"
excerpt: "@Configuration, 싱글톤 패턴 개념 정리"

categories: # 카테고리 설정
  - Web
tags: # 포스트 태그
  - [tag1, tag2]

permalink: /Web/configuration-singleton/ # 포스트 URL

toc: true # 우측에 본문 목차 네비게이션 생성
toc_sticky: true # 본문 목차 네비게이션 고정 여부

date: 2025-02-17 # 작성 날짜
last_modified_at: 2025-02-17 # 최종 수정 날짜
---


#### AppConfig
```java
@Configuration
public class AppConfig {
 
 @Bean
 public MemberService memberService() {
 	return new MemberServiceImpl(memberRepository());
 }
 
 @Bean
 public OrderService orderService() {
 	return new OrderServiceImpl(
 	memberRepository(),
 	discountPolicy());
 }
 
 @Bean
 public MemberRepository memberRepository() {
 	return new MemoryMemberRepository();
 }
 ...
}
```

이전에 작성한 AppConfig 파일을 살펴보면 `memberRepository()`를 호출하는 코드가 `memberService()`, `orderService()`를 만드는 코드에서 반복되는 모습을 볼 수 있다. 단순하게 생각하면 이는 각기 다른 두 개의 `memberRepository`가 생성되어 싱글톤 패턴이 깨지는게 아닐까?라고 볼 수 있다. 하지만 스프링 컨테이너가 이 문제를 어떻게 해결하는지 예제 코드들을 살펴보며 알아보도록 하자.

#### 테스트 코드
```java
public class ConfigurationSingletonTest {
 @Test
 void configurationTest() {
 ApplicationContext ac = newAnnotationConfigApplicationContext(AppConfig.class);
 	MemberServiceImpl memberService = ac.getBean("memberService",

MemberServiceImpl.class);
 OrderServiceImpl orderService = ac.getBean("orderService",

OrderServiceImpl.class);
 	MemberRepository memberRepository = ac.getBean("memberRepository",
	MemberRepository.class);
 
 //모두 같은 인스턴스를 참고하고 있다.
 System.out.println("memberService -> memberRepository = " +
	memberService.getMemberRepository());
 
 System.out.println("orderService -> memberRepository = " +
	orderService.getMemberRepository());
 
 System.out.println("memberRepository = " + memberRepository);
 //모두 같은 인스턴스를 참고하고 있다.

assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);

assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
 }
}
```
테스트 코드를 작성하여 테스트를 돌려보아도 `memberRepository` 인스턴스는 모두 같은 인스턴스가 공유되어 사용되는 것을 알 수 있다. 그렇다면 이러한 방식이 가능한 이유는 무엇일까?

## @Configuration과 바이트코드 조작
스프링 컨테이너는 싱글톤 레지스트리이므로 스프링 빈이 싱글톤이 되도록 보장해 주어야한다. 하지만 스프링이 자바코드 자체를 어떻게 조작하기는 어렵다. 위의 예제 코드를 살펴 보면 알 수 있듯이 저 코드가 실행 되면 멤버 리포지토리가 2번 호출되어야 하는 것이 정상이다. 이를 해결하기 위해 스프링은 **클래스의 바이트 코드를 조작하는 라이브러리를 사용한다.** 예제 코드를 통해 이를 살펴보도록 하자.

#### configurationDeep
```java
@Test
void configurationDeep() {
 ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
 
 //AppConfig도 스프링 빈으로 등록된다.
 AppConfig bean = ac.getBean(AppConfig.class);

 System.out.println("bean = " + bean.getClass());
 //출력: bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70
}
```

위의 코드를 보면 알 수 있듯이 `AnnotationConfigApplicationContext`에 파라미터로 넘긴 값은 스프링 빈으로 등록이 된다. 따라서 `AppConfig`도 스프링 빈으로 등록이 되는 것이다. `AppConfig` 스프링 빈을 조회하여 클래스정보를 출력해보면 `xxxCGLIB`가 붙은 복잡한 클래스명이 출력되는것을 알 수 있다.
즉, 내가 직접 만든 클래스가 아니라 스프링이 CGLIB라는 바이트 코드 조작 라이브러리를 사용하여 `AppConfig`를 상속받은 임의의 다른 클래스를 만들고 그 다른 클래스를 스프링 빈으로 등록한다는 것이다.

![](https://velog.velcdn.com/images/gwoprk/post/635235f6-6a6e-4650-9385-33ec7bcfcf19/image.png)

임의의 다른 클래스가 싱글톤이 되도록 보장해준다.

- `@Bean`이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성하여 스프링 빈으로 등록하고 반환하는 코드가 동적으로 생성된다.
- 이로인하여 싱글톤이 보장된다.

> `@Configuration`을 사용하지 않고 `@Bean`만 적용한다면 스프링 빈으로 등록은 가능하지만 싱글톤이 보장되지 않는다. 따라서 스프링 설정 정보를 사용할때는 항상 `@Configuration`을 사용하는것이 좋다.
