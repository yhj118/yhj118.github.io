---
title: Java 첫 글입니다.
date: 2026-07-20 22:00:00 +0900
categories: # study의 java 카테고리에 이 글을 넣어라
  - Develop
  - Java
tags: # 태그 목록에 java를 넣어라
  - Java 
---

# Java 
<!-- 제목, html로 치면 <h1>Java</h1> -->

1. Bean의 생명주기(생성, 관리)는 스프링컨테이너가 한다. (💡스프링컨테이너에 대해서)
2. thymeleaf는 서버가 가지고 있는 "데이터"를 HTML에 출력하는 템플릿 엔진이다.
3. templates는 HTML을 관리한다.
4. 컨트롤러에서 서비스 호출과 사용은 별개이다.
생성자를 통해 호출한 서비스를 컨트롤러에 사용 가능하다.(생성자 = 호출한 서비스 객체를 컨트롤러로 주입해주는 통로)
@RequredArgsConstructor 어노테이션 사용 시 Lombok이 자동으로 생성자를 생성해준다.
5. 컨트롤러와 서비스의 메서드 타입(반환 타입)은 달라도 되며 별개이다. 
컨트롤러 String 타입은 보통 이용할 view/redirect를 반환하고 서비스 int는 DB를 처리 결과 값을 반환한다.(몇 행 처리했는지?) 
컨트롤러와 서비스의 타입은 String, int, Long 등 다양하며 리턴으로 뭘 받고 싶을지에 따라 설계한다.
6. 
```text
웹 통신 기초
│
├─ 1. HTTP 기본
│   ├─ 클라이언트 / 서버
│   ├─ HTTP Request
│   └─ HTTP Response
│
├─ 2. HTTP 메시지 구조
│   ├─ 요청/응답 시작줄
│   ├─ Header
│   └─ Body
│
├─ 3. HTTP 요청 방식
│   ├─ GET
│   ├─ POST
│   ├─ Query Parameter
│   └─ Request Body
│
├─ 4. 데이터 형식
│   ├─ JSON
│   ├─ text
│   └─ HTML
│
├─ 5. AJAX / fetch
│   ├─ 비동기 통신
│   ├─ fetch()
│   └─ response.json()
│
└─ 6. Spring 연결
    ├─ @RequestParam
    ├─ @RequestBody
    ├─ @ResponseBody
    └─ Controller return
```

