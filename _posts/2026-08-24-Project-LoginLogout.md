---
title: 로그인 및 로그아웃 개발일지(feat. Spring Security 프레임워크)
date: 2026-08-24 22:00:00 +0900
categories:
  - Project
  - Intranet GroupWare
tags:
  - Project
---

# Intranet Groupware

---

## 로그인 및 로그아웃

### 1. Spring Security

- 기존 로그인 구현은 DB 조회 후 사용자 정보에 세션을 선언하여 구현했다면 Spring Security는 좀 더 실무에 맞게 구현.
- Spring Security 또한 세션 방식의 구현이며 둘의 차이는 전자는 직접 코드를 작성하며 관리, 후자는 프레임워크가 관리.
- 특징으론 컨트롤러를 사용하지 않는다.

### 2. Spring Security로 로직 구현 순서

1. gradle 의존성 추가

2. SecurityConfig 클래스 생성
  - SecurityFilterChain을 Bean으로 등록
  - 접근 권한 설정
  - 로그인 폼 설정

3. DB에서 사용자 조회를 위한 로직 작성
  - AuthMapper * Mapper XML * AuthService

4. CustomUserDetailService 작성
  - loadUserByUsername()에서 AuthService를 호출하여 사용자 조회 로직 구현
  - DB로부터 반환받은 EmpResponse 객체를 UserDetails로 변환

5. SecurityConfig 클래스에서 로그아웃 및 로그인 상태 유지 로직 추가

6. 로그인 후 Spring Security가 인증된 Authentication 객체인 사용자 정보를 SecurityContext에 저장.
- SecurityContext를 HttpSession에 저장하여 로그인 유지


### 3. 플로우차트

```text
        [로그인 form submit 발생]
                  │
                  ▼
            POST /goLogin http 요청
                  │
                  ▼
      Spring Security Filter가 요청 수신
                  │
                  ▼
      loginKey / password 값 추출
                  │
                  ▼
       CustomUserDetailsService 호출
                  │
                  ▼
    loadUserByUsername(loginKey) 실행
                  │
                  ▼
             AuthService 호출
                  │
                  ▼
             AuthMapper 호출
                  │
                  ▼
          employee 테이블 조회
                  │
                  ▼
          EmpResponse 객체 반환
                  │
                  ▼
       EmpResponse → UserDetails 변환
                  │
                  ▼
      Spring Security에 UserDetails 반환
                  │
                  ▼
       PasswordEncoder로 비밀번호 비교
                  │
          ┌───────┴───────┐
          │               │
         불일치             일치
          │               │
          ▼               ▼
      로그인 실패      Authentication 생성
                          │
                          ▼
               SecurityContext에 저장
                          │
                          ▼
                     로그인 성공
                          │
                          ▼
                    /openMain 이동
                          │
                          ▼
              인증이 필요한 요청 발생
                 예: /myPage
                          │
                          ▼
                 인증 상태 확인
                          │
                 ┌────────┴────────┐
                 │                 │
               미인증              인증됨
                 │                 │
                 ▼                 ▼
          /openLogin 이동       접근 허용
```
