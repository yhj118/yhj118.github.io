---
title: MyBatis Mapper 인터페이스와 XML
date: 2026-07-20 22:00:00 +0900
categories:
  - Develop
  - Spring
tags:
  - Spring
  - MyBatis
---

# MyBatis Mapper 인터페이스와 XML

매번 백엔드 개발을 할 때마다 헷갈려서 포스팅.

MyBatis XML 문서에서 CRUD를 구현할 때 `parameterType`과 `resultType`에 무엇을 작성해야 하는지 매번 헷갈려서 "다시" 정리한다.

## parameterType과 resultType

먼저 두 속성의 의미부터 정리하면 다음과 같다.

1. `parameterType` = XML로 들어오는 값의 타입
2. `resultType` = SQL 실행 후 조회되어 나가는 값의 타입

기본적인 내용이지만 여기에 무엇을 작성해야 하는지는 **Mapper 인터페이스의 메서드를 보면 알 수 있다.**

---

## INSERT의 경우

인터페이스에 회원가입(INSERT)을 담당하는 다음 메서드가 있다고 하자.

```java
int empSignUp(EmpRequest params);
```

메서드의 반환 타입은 `int`이고 파라미터 타입은 `EmpRequest`이다.

```text
int empSignUp(EmpRequest params);
↑               ↑
반환 타입         파라미터 타입
```

### parameterType

메서드의 파라미터 타입은 `EmpRequest` 클래스 객체이다.

따라서 XML에서는 다음과 같이 `parameterType`을 지정한다.

```xml
<insert id="empSignUp"
        parameterType="io.github.yhj118.groupware.employee.EmpRequest">

</insert>
```

`parameterType`에 `EmpRequest`를 지정하면 MyBatis는 전달받은 `EmpRequest` 객체의 프로퍼티를 기준으로 데이터를 가져올 수 있다.

예를 들어 `EmpRequest`에 다음 필드가 있다면,

```java
private String loginId;
private String password;
```

XML에서는 다음과 같이 사용할 수 있다.

```xml
#{loginId}
  #{password}
```

즉,

```text
EmpRequest
   ↓
parameterType="EmpRequest"
   ↓
#{loginId}
#{password}
```

와 같은 흐름이다.

### INSERT에는 왜 resultType이 없을까?

`empSignUp()`의 반환 타입은 `int`인데 XML의 `<insert>`에는 `resultType="int"`를 작성하지 않는다.

```xml
<insert id="empSignUp"
        parameterType="io.github.yhj118.groupware.employee.EmpRequest">
```

그 이유는 INSERT의 `int`가 SELECT를 통해 조회한 결과가 아니기 때문이다.

INSERT를 실행하면 MyBatis가 해당 SQL 실행으로 **영향받은 행(affected rows)의 개수**를 Mapper 메서드의 반환값으로 넘겨준다.

예를 들어 한 명의 사원이 정상적으로 등록되었다면,

```text
INSERT 실행
    ↓
1행 추가
    ↓
영향받은 행 = 1
    ↓
MyBatis
    ↓
int empSignUp(...) → 1
```

이런 식으로 동작한다.

따라서 INSERT, UPDATE, DELETE의 반환 타입이 `int`라고 해서 XML에 `resultType="int"`를 작성하는 것은 아니다.

---

## SELECT의 경우

이번에는 인터페이스에 사원 한 명을 조회하는 메서드가 있다고 하자.

```java
EmpResponse findEmployee(String loginId);
```

이 메서드를 보면 반환 타입은 `EmpResponse`이고 파라미터 타입은 `String`이다.

```text
EmpResponse findEmployee(String loginId);
↑                        ↑
resultType               parameterType
```

따라서 XML에서는 다음과 같이 작성할 수 있다.

```xml
<select id="findEmployee"
        parameterType="String"
        resultType="io.github.yhj118.groupware.employee.EmpResponse">

  SELECT
  employee_id,
  login_id,
  employee_name
  FROM employee
  WHERE login_id = #{loginId}

</select>
```

Mapper 인터페이스에서 전달하는 값이 `String`이므로,

```xml
parameterType="String"
```

이 되고, SQL 조회 결과를 `EmpResponse` 객체로 반환받으므로,

```xml
resultType="EmpResponse"
```

가 된다.

즉 Mapper 인터페이스를 기준으로 보면 다음과 같다.

```text
EmpResponse findEmployee(String loginId);
↑                        ↑
│                        │
resultType               parameterType
```

---

## SELECT 결과가 int인 경우

아이디 중복 확인을 위해 다음과 같은 메서드가 있다고 하자.

```java
int countLoginId(String loginId);
```

SQL에서는 아이디가 존재하는지 `COUNT(*)`를 이용해 조회한다고 가정한다.

```sql
SELECT COUNT(*)
FROM employee
WHERE login_id = #{loginId}
```

이 경우 Mapper 메서드의 반환 타입은 `int`이고 파라미터 타입은 `String`이다.

따라서 XML에서는 다음과 같이 작성한다.

```xml
<select id="countLoginId" parameterType="String" resultType="int">
  SELECT COUNT(*) FROM employee WHERE login_id = #{loginId}
</select>
```

여기서 중요한 것은 앞에서 살펴본 INSERT의 `int`와 의미가 다르다는 것이다.

```text
int empSignUp(EmpRequest params);
↑
INSERT 실행으로 영향받은 행의 개수
→ XML에 resultType 없음


int countLoginId(String loginId);
↑
SELECT COUNT(*)를 통해 실제 조회한 결과
→ XML에 resultType="int"
```

---

## 파라미터가 없는 경우

이번에는 모든 사원을 조회하는 메서드가 있다고 하자.

```java
List<EmpResponse> findAllEmployee();
```

괄호 안에 파라미터가 없다.

```text
findAllEmployee()
                ↑
          전달하는 값 없음
```

따라서 XML에는 `parameterType`을 작성할 필요가 없다.

```xml
<select id="findAllEmployee" resultType="io.github.yhj118.groupware.employee.EmpResponse">
  SELECT employee_id, login_id, employee_name FROM employee
</select>
```

여기서 Mapper 인터페이스의 반환 타입은 `List<EmpResponse>`이지만 XML의 `resultType`에는 `List<EmpResponse>`가 아닌 `EmpResponse`를 작성한다.
그 이유는 `resultType`이 **조회 결과 한 행(row)을 어떤 타입의 객체로 만들 것인지**를 나타내기 때문이다.

예를 들어 DB에서 사원 3명이 조회된다면,

```text
1행 → EmpResponse
2행 → EmpResponse
3행 → EmpResponse
        ↓
MyBatis가 묶어줌
        ↓
List<EmpResponse>
```

이런 구조가 된다.

따라서,

```java
List<EmpResponse> findAllEmployee();
```

라고 되어 있더라도 XML에서는,

```xml
resultType="EmpResponse"
```

를 사용한다.

---

## 정리

XML의 `parameterType`과 `resultType`을 작성할 때는 **Mapper 인터페이스의 메서드를 기준으로 생각하면 된다.**

| Mapper Interface                | parameterType | resultType    |
| ------------------------------- | ------------- | ------------- |
| `int insert(EmpRequest p)`      | `EmpRequest`  | 없음            |
| `int update(EmpRequest p)`      | `EmpRequest`  | 없음            |
| `int delete(Long id)`           | `Long`        | 없음            |
| `EmpResponse select(String id)` | `String`      | `EmpResponse` |
| `String selectName(Long id)`    | `Long`        | `String`      |
| `int count(String id)`          | `String`      | `int`         |
| `List<EmpResponse> selectAll()` | 없음            | `EmpResponse` |

### 핵심

**Mapper 메서드의 파라미터 타입 → `parameterType`**

```java
findEmployee(String loginId)
             ↑
parameterType
```

**SELECT한 한 행의 결과 타입 → `resultType`**

```java
EmpResponse findEmployee(...)
↑
resultType
```

**INSERT / UPDATE / DELETE의 반환 타입 `int` → 영향받은 행의 개수**

```java
int empSignUp(EmpRequest params);
↑
영향받은 행의 개수
```

따라서 INSERT / UPDATE / DELETE에서는 반환 타입이 `int`이더라도 XML에 `resultType="int"`를 작성하지 않는다.

결국 `parameterType`, `resultType`이 헷갈리면 **XML부터 보지 말고 Mapper 인터페이스 메서드부터 확인하자.**
