---
title: "[Servlet] HttpReuqest"
excerpt: "HttpRequset 알아보기"

categories: # 카테고리 설정
  - Web
tags: # 포스트 태그
  - [tag1, tag2]

permalink: /Web/http-request/ # 포스트 URL

toc: false # 우측에 본문 목차 네비게이션 생성
toc_sticky: false # 본문 목차 네비게이션 고정 여부

date: 2025-02-17 # 작성 날짜
last_modified_at: 2025-02-17 # 최종 수정 날짜
---


## HttpServletRequest

#### HttpServletRequest의 역할
서블릿은 개발자가 HTTP 요청 메시지를 편리하게 사용할 수 있도록 개발자 대신에 HTTP 요청 메시지를 파싱한다. 그리고 그 결과를 `HttpServletRequest`에 담아 제공한다.

#### HTTP 요청 메시지
```POST /save HTTP/1.1
Host: localhost:8080
Content-Type: application/x-www-form-urlencoded
username=kim&age=20
```
`HttpServletRequest`을 이용하면 위와같은 HTTP 요청 메시지를 편리하게 조회 가능하다.

- START Line
	
    - HTTP 메소드
    - URL
    - 쿼리 스트링
    - 스키마, 프로토콜
- 헤더
	
    - 헤더 조회
- 바디
	
    - form 파라미터 형식 조회
    - message 바디 데이터 직접 조회
    
이 외에도, `HttpServletRequest`은 임시 저장소 기능, 세션 관리 기능과같은 부가 기능을 제공한다.

