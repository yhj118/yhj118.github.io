---
title: 사원 등록, 수정, 삭제 개발일지
date: 2026-08-20
categories:
  - Project
  - Intranet GroupWare
tags:
  - Project
---

# Intranet Groupware

## 사원 등록 구현

1. HTML UI 적용 (Bootstrap Tabler)
2. DB `employee`, `department` 테이블 생성
- `employee.department_id(FK)`는 `department.department_id(PK)`를 참조
- `department : employee = 1 : N`
3. MyBatis Mapper Interface / Mapper XML 작성
4. Service 작성
5. Controller 작성
6. JavaScript ID 중복 및 유효성 검사
7. 주소 API 연동
8. 프로필 사진 업로드 구현
- `MultipartFile` 사용
- JPG / JPEG / PNG 제한
- 5MB 용량 제한
- UUID 파일명 생성
- `uploads/profile` 경로에 실제 파일 저장
- DB에는 이미지가 아닌 이미지 경로만 저장
9. 사원 등록 전체 테스트

### 주요 이슈

#### 1. ImageHandler 의존성 주입 오류

프로필 사진 업로드 로직을 수행하는 `ImageHandler` 클래스를 별도로 작성한 뒤  
Controller에 의존성 주입하는 과정에서 오류가 발생했다.

- `ImageHandler`가 Spring Bean으로 등록되지 않은 것이 원인
- `@Component`를 사용하여 Bean으로 등록하여 해결
- 이 과정에서 `@RequiredArgsConstructor`와 생성자 주입의 동작 원리를 이해

#### 2. Controller와 Service 반환 타입에 대한 혼동

Controller와 Service의 반환 타입을 동일하게 맞춰야 한다고 생각했다.

- 각 메서드는 독립적이므로 반환 타입이 일치할 필요가 없음
- Service가 `int`를 반환하더라도 Controller에서 반환값이 필요하지 않다면 호출만 할 수 있음
- Service의 반환값을 View에서 사용하고 싶다면 Controller에서 `Model`에 담아 전달

예시:

```java
int result = empService.empSignUp(params);
model.addAttribute("result", result);

return "index";
```

View에서는 다음과 같이 사용할 수 있다.

```html
<span th:text="${result}"></span>
```

#### 3. JavaScript 함수 내부의 return 동작 이해

`if`문 내부에 `return`이 존재할 경우 `if`문만 종료되는 것이 아니라  
`return`이 속해 있는 함수 전체가 종료된다는 점을 이해했다.

### 회원가입 유효성 검사 흐름

```text
              [form submit 발생]
                       │
                       ▼
            ID 중복확인 했는가?
                       │
             ┌─────────┴─────────┐
             │                   │
            아니오                예
             │                   │
             ▼                   ▼
      preventDefault()       비밀번호 형식 검사
         가입 중단                  │
                            ┌───────┴───────┐
                            │               │
                           실패              성공
                            │               │
                            ▼               ▼
                     preventDefault()   비밀번호 확인값과
                        가입 중단          같은지 검사
                                            │
                                     ┌──────┴──────┐
                                     │             │
                                    다름            같음
                                     │             │
                                     ▼             ▼
                              preventDefault()   이름 정규식 검사
                                 가입 중단              │
                                               ┌─────┴─────┐
                                               │           │
                                              실패          성공
                                               │           │
                                               ▼           ▼
                                        preventDefault()  정상 제출
                                           가입 중단          │
                                                             ▼
                                                      POST /signUp
                                                             │
                                                             ▼
                                                        Controller
```

## 사원 등록 구현 결과

최종적으로 구현된 사원 등록 화면이다.

![사원 등록 화면](/assets/images/groupware/register1.png)

## 사원 수정 구현

- 예정

## 사원 삭제 구현

- 예정
