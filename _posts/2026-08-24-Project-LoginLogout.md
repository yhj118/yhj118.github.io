---
title: 사원 로그인 및 로그아웃 개발일지(feat. Spring Security 프레임워크)
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

### 3. 구현도

```mermaid
flowchart TD

    A[사용자 로그인 폼<br/>loginKey + password 입력]
    --> B[POST /goLogin]

    B --> C[Spring Security<br/>UsernamePasswordAuthenticationFilter]

    C --> D[AuthenticationManager / Provider]

    D --> E[CustomUserDetailsService<br/>loadUserByUsername loginKey]

    E --> F[AuthService]

    F --> G[AuthMapper]

    G --> H[(DB<br/>employee 테이블)]

    H --> I[EmpResponse 반환<br/>employeeNum + password]

    I --> J[EmpResponse → UserDetails 변환]

    J --> K[Spring Security에<br/>UserDetails 반환]

    K --> L[PasswordEncoder<br/>비밀번호 비교]

    L --> M{비밀번호 일치?}

    M -- 아니오 --> N[로그인 실패]

    M -- 예 --> O[Authentication 생성]

    O --> P[SecurityContext에<br/>Authentication 저장]

    P --> Q[로그인 상태]

    Q --> R[/openMain 이동]

    R --> S[인증이 필요한 요청<br/>/myPage 등]

    S --> T{authenticated?}

    T -- 예 --> U[접근 허용]
    T -- 아니오 --> V[/openLogin 이동]
```
