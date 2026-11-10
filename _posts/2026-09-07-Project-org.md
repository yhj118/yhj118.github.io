---
title: "조직도 개발일지"
date: 2026-09-11 00:00:00 +0900
categories:
  - Project
  - Intranet GroupWare
tags:
  - Project
  - Organization Chart
  - Spring Boot
  - JavaScript
---

# Intranet Groupware

---

## 1. 조직도 구현 목표

> 사내 조직 구조를 **회사 → 본부(상위 부서) → 팀 → 사원** 순서로 확인하고, 필요한 팀의 구성원과 사원 프로필을 조회할 수 있도록 한다.

조직도는 사원 등록 때 저장한 소속과 프로필 사진을 활용하는 조회 화면이다. <br> 
처음에는 본부와 팀만 보여주고, 버튼을 눌러 팀원을 펼치도록 구성했다. <br>
사원을 선택하면 현재 페이지에서 프로필 모달이 열리고, 조직도는 PNG 이미지로 저장할 수 있다. <br>

### 사용 기술

| 기술 | 조직도에서 맡은 역할 |
| --- | --- |
| Java 17 · Spring Boot · Spring MVC | 조직도 페이지와 사원 프로필 요청 처리 |
| Lombok | `@RequiredArgsConstructor`로 생성자 주입에 필요한 생성자를 만들고, `@Getter`와 `@Setter`로 조회 객체의 접근 메서드를 생성 |
| MariaDB · MyBatis · HikariCP | 부서와 사원 데이터 JOIN, 조직 규모 집계, XML 조회 결과를 Java 객체로 매핑, DB 연결 재사용 |
| Thymeleaf | 요약 숫자를 HTML에 표시하고, 조직 목록을 브라우저의 JavaScript로 전달 |
| HTML · CSS · Tabler(Bootstrap 기반) | 본부와 팀 카드, 사원 카드, 프로필 모달, 상태 배지 구성 |
| 순수 JavaScript | 조회 결과 그룹화, DOM 생성, 팀원 펼치기·접기, `fetch()`로 프로필 조회 |
| html2canvas 1.4.1 | 조직도 DOM 영역을 Canvas로 그린 뒤 PNG 저장 |
| Spring Security · 내부망 필터 | 인증되지 않은 요청과 설정한 IP 대역을 벗어난 요청 제한 |

---

## 2. DB 구조와 조직 데이터 조회

### 본부와 팀을 한 테이블에서 관리

조직 단위는 `department` 테이블에 저장한다. <br>  
팀의 `parent_department_id`를 상위 부서 행의 `department_id`에 연결한다. <br>  
사원의 `employee.department_id`는 자신이 속한 조직 행을 참조한다.

| 관계 | 연결 기준 | 의미 |
| --- | --- | --- |
| 본부 → 팀 | `team.parent_department_id = parent.department_id` | 한 본부에 여러 팀이 속할 수 있다. |
| 팀 → 사원 | `employee.department_id = team.department_id` | 한 팀에 여러 사원이 속할 수 있다. |

`findOrgList()`에서는 같은 `department` 테이블을 `team`과 `parent`라는 별칭으로 두 번 사용한다. <br> 
이 **자기 조인(Self Join)**으로 팀과 상위 부서를 한 행에 연결하고, 사원을 다시 조인한다. <br>  
회사명인 `Groupware`는 DB에서 조회하는 조직 단위가 아니라 화면의 최상단에 둔 고정 노드다.

아래는 원본 `orgMapper.xml`에서 조회 관계를 중심으로 줄인 SQL이다.

```sql
SELECT e.employee_id,
       e.employee_name,
       e.level,
       pt.department_type,
       pt.department_name
FROM employee AS e
JOIN (
    SELECT team.department_id,
           parent.department_type,
           team.department_name
    FROM department AS team
    JOIN department AS parent
      ON team.parent_department_id = parent.department_id
) AS pt
  ON e.department_id = pt.department_id;
```

여기서 `parent.department_type`은 화면에서 **본부를 묶는 기준**, `team.department_name`은 **팀 이름**이다. <br>  
조회 결과는 사원 한 명당 한 행인 `List<OrgResponse>`로 받는다. <br>  
MyBatis의 `map-underscore-to-camel-case` 설정으로 `employee_name` 같은 컬럼을 `employeeName` 필드에 매핑한다.

### 조직 규모 요약

`findOrgSummary()`는 목록과 같은 조인 범위를 집계한다. <br>  
사원이 여러 명인 팀은 조회 결과에 여러 번 나오므로, 조직 수를 셀 때는 `DISTINCT`를 사용했다.

| SQL | Java 필드 | 화면 사용 |
| --- | --- | --- |
| `COUNT(DISTINCT pt.department_type)` | `totalDepartmentType` | 요약 객체에 저장 |
| `COUNT(DISTINCT pt.department_name)` | `totalDepartmentName` | 상단의 팀 수 표시 |
| `COUNT(e.employee_id)` | `totalEmployeeId` | 상단과 회사 노드의 인원 수 표시 |

> **한 줄 요약**  
> 팀과 상위 부서를 자기 조인하고 사원과 연결해, 조직도를 그릴 목록과 화면 상단의 요약 숫자를 각각 조회한다.

---

## 3. 조직도 페이지와 데이터 전달

사이드바에서 조직도 메뉴를 누르면 `GET /openOrgMain`으로 요청한다. `OrgController`는 `OrgService`를 통해 조직 목록과 요약을 가져온다. `OrgService`는 각 요청을 `OrgMapper`에 위임하고, SQL은 `orgMapper.xml`에서 실행한다.

1. `findOrgList()`로 본부·팀·사원 목록을 조회한다.
2. `findOrgSummary()`로 조직 규모를 집계한다.
3. 목록의 직급 코드 `lv1`~`lv5`를 사원·대리·과장·차장·부장으로 바꾼다.
4. 조회 결과를 `findOrgList`, `findOrgSummary` 이름으로 `Model`에 담고 `org/orgMain` 뷰를 반환한다.

```java
model.addAttribute("findOrgList", findOrgList);
model.addAttribute("findOrgSummary", findOrgSummary);

return "org/orgMain";
```

요약 숫자는 `th:text`로 HTML에 표시하고, 조직 목록은 `th:inline="javascript"`로 JavaScript 배열에 넣는다.

```html
<script th:inline="javascript">
    const orgList = /*[[${findOrgList}]]*/ [];
</script>
```

`/*[[${findOrgList}]]*/` 부분은 서버가 페이지를 렌더링할 때 실제 목록으로 바뀐다. <br> 
브라우저는 이미 데이터가 들어 있는 HTML을 받기 때문에 **초기 조직도를 그리려고 별도의 목록 API를 호출하지 않는다.**

> **한 줄 요약**  
> Controller가 모델을 채우고, Thymeleaf가 조직 목록을 JavaScript에 전달한다.

---

## 4. 조직도를 화면에 그리기

### 평평한 조회 결과를 본부와 팀으로 묶기

SQL 결과에는 본부와 팀 이름이 사원마다 반복된다. <br>  
그대로 출력하면 한 팀이 사원 수만큼 반복되므로, JavaScript에서 **본부 이름 → 팀 이름 → 사원 배열** 구조로 재구성했다.

```js
const departments = {};

orgList.forEach(function (org) {
    const departmentType = org.departmentType;
    if (!departments[departmentType]) departments[departmentType] = {};

    const departmentName = org.departmentName;
    if (!departments[departmentType][departmentName]) {
        departments[departmentType][departmentName] = [];
    }

    if (org.employeeId != null) {
        departments[departmentType][departmentName].push({
            employeeId: org.employeeId,
            employeeName: org.employeeName,
            level: org.level,
            profileImgPath: org.profileImgPath
        });
    }
});
```

예를 들어 개발본부 소속 플랫폼팀 사원이 두 명이면, `departments["개발본부"]["플랫폼팀"]` 배열에 두 사원이 들어간다. <br>  
화면에서 팀 인원은 이 배열의 `length`로 표시한다.

### DOM과 CSS 구성

`renderOrganization()`은 `Object.entries()`로 본부와 팀을 순회한다. <br>  
`document.createElement()`로 본부 영역을 만들고, 팀 카드와 사원 목록을 템플릿 문자열로 구성한 뒤 `#departmentRow`에 추가한다. <br> 
펼친 팀원 목록에는 사원 이름과 직급이 보이고, 사진은 사원 프로필 모달에서 사용한다.

- 회사 노드는 트리 상단에 두고, 얇은 연결선으로 본부와 팀의 계층을 나타낸다.
- 본부 카드는 Flexbox로 가로 배치한다. `width: max-content`와 `margin: 0 auto`를 적용한 본부 행은 내용 폭에 맞춰 배치되며, 본부가 하나일 때도 중앙에 놓인다.
- 팀 카드는 각 본부 아래에 Grid로 세로 배치한다. 본부가 많아 화면 폭을 넘으면 `.organization-wrapper`의 가로 스크롤로 확인할 수 있다.
- 카드와 배경에 Tabler의 CSS 변수를 사용해 프로젝트의 색상 설정과 어울리도록 구성했다.

### 팀원 펼치기·접기

| 기능 | 구현 방식 |
| --- | --- |
| 처음 열었을 때 | 각 `.employee-list`에 `d-none`이 붙어 팀원 목록이 숨겨져 있다. |
| 팀별 보기·접기 | `toggleMembers(this)`가 `closest(".team-node")`로 클릭한 팀을 찾고, 그 팀의 `d-none`과 버튼 문구를 바꾼다. |
| 전체 펼치기·접기 | `toggleAllMembers()`가 `allMembersExpanded`를 반전시키고 모든 팀원 목록, 팀별 버튼, 전체 버튼의 상태를 일괄 변경한다. |

팀원 목록은 이미 브라우저에 만들어져 있으므로, 펼치기·접기에는 서버 요청이 필요하지 않다.

> **한 줄 요약**  
> 사원별 조회 결과를 중첩 객체로 묶어 조직도를 그리고, CSS 클래스로 필요한 팀의 구성원만 보여준다.

---

## 5. 사원 프로필 상세 조회

사원 이름을 선택하면 `openEmployeeProfile(employeeId)`가 선택한 사원 ID를 경로에 넣어 `GET /org/employee/{employeeId}`로 요청한다. 서버는 `@PathVariable`로 ID를 받아 `findByEmployeeId()`를 호출한다. 이 쿼리는 `employee`와 사원이 소속된 `department` 행을 조인하고, `WHERE employee_id = #{employeeId}`로 한 명을 조회한다.

초기 페이지 요청은 **HTML을 반환**하지만, 프로필 요청은 `@ResponseBody`를 사용해 **JSON을 반환**한다. 상세 쿼리의 부서 값은 소속 행인 `d.*`에서 읽는다. 상위 부서의 `department_type`을 쓰는 조직 목록 쿼리와는 가져오는 기준이 다르다.

서버에서 직급과 상태 코드를 아래와 같이 화면용 문구로 바꾼다.

| 항목 | 코드 → 표시 문구 |
| --- | --- |
| 직급 | `lv1`~`lv5` → 사원·대리·과장·차장·부장 |
| 재직 상태 | `EMPLOYED` / `ON_LEAVE` / `RESIGNED` → 재직중 / 휴가중 / 퇴사 |
| 등록 승인 상태 | `PENDING` / `APPROVED` / `REJECTED` → 사원등록 승인 전 / 사원등록 승인 / 사원등록 거절 |
| 권한 | `EMPLOYEE` / `ADMIN` → 비관리자 / 관리자 |

브라우저는 `fetch()`의 응답을 `response.json()`으로 읽고 다음과 같이 처리한다.

1. `textContent`로 이름·사번·로그인 ID·본부·팀·직급 등을 사원증 형태의 모달에 표시한다.
2. 재직 상태와 등록 승인 상태에 따라 배지 색을 바꾼다. 이전 사원의 배지 클래스는 먼저 제거한다.
3. 퇴사일시가 없으면 해당 칸에 `재직중`을 표시하고, 값이 있으면 퇴사일시를 표시한다.
4. `img.src`에 `profileImgPath`를 설정한다. 파일은 `uploads/profile/`에 있고, `WebConfig`가 URL `/uploads/**`를 해당 저장 폴더에 연결한다.
5. 숨겨 둔 모달 트리거 버튼을 클릭해 Tabler의 Bootstrap 모달을 열고, 닫힐 때 `hidden.bs.modal` 이벤트에서 기존 세로 스크롤 위치를 복원한다.

> **한 줄 요약**  
> 조직도는 그대로 두고 클릭한 사원의 상세 정보만 비동기로 조회해 프로필 모달에 표시한다.

---

## 6. 조직도 PNG 저장

`조직도 캡처` 버튼은 `captureOrganization()`을 실행한다. html2canvas는 페이지 전체가 아니라 `.organization-tree` 영역만 Canvas로 그린다. 조직도 바깥의 헤더, 버튼, 프로필 모달은 캡처 대상에 포함되지 않는다.

```js
async function captureOrganization() {
    const target = document.querySelector(".organization-tree");

    const canvas = await html2canvas(target, {
        scale: 2,
        backgroundColor: "#ffffff",
        useCORS: true
    });

    const link = document.createElement("a");
    link.download = "조직현황.png";
    link.href = canvas.toDataURL("image/png");
    link.click();
}
```

- `scale: 2`로 캡처 해상도를 높이고, `backgroundColor`로 흰색 배경을 지정한다.
- `useCORS`는 외부 출처 이미지가 있는 경우 CORS 방식으로 불러오도록 시도하는 옵션이다. 상대 서버의 허용이 필요하다.
- Canvas를 PNG 데이터 URL로 변환해 브라우저에서 직접 다운로드한다. 별도 서버 저장이나 다운로드 API는 없다.
- 현재 DOM 상태를 그리므로 **팀원을 접은 상태에서는 해당 팀원 목록이 이미지에 나오지 않는다.** 팀원까지 담으려면 먼저 `전체 펼치기`를 누른다.
- Tabler와 html2canvas는 프로젝트의 정적 리소스에서 불러온다. 조직도 화면과 캡처 기능에 외부 CDN 연결이 필요하지 않도록 구성했다.

---

## 7. 접근 제어

조직도 페이지와 프로필 API는 `SecurityConfig`의 공개 경로에 포함되지 않으므로 로그인 후 접근할 수 있다. 공통 `InternalNetworkFilter`에서는 접속 IP가 설정한 CIDR 대역에 속하는지 검사하고, 허용 대역 밖이면 `403 Forbidden`을 반환한다. 프로필에 관리자 여부를 표시하는 것과 프로필 API에 별도 조회 권한을 적용하는 것은 별개다. 현재 조직도 API에는 별도의 관리자·본인 조건이 없다.

---

## 주요 이슈 및 개선 방향

### 1. 목록과 상세 응답의 조회 범위 줄이기

현재 목록 쿼리는 `e.*`를 읽고, 상세 쿼리는 `e.*, d.*`를 읽는다. 두 조회가 공통 `OrgResponse`를 사용하는데, 이 객체에는 `password`·생년월일·주소 같은 필드도 있다. **화면에서 표시하지 않는 값도 초기 HTML의 인라인 데이터나 상세 JSON에 포함될 수 있다.** 비밀번호 해시를 포함한 인증 정보는 두 응답에서 모두 제외해야 한다.

개선할 때는 **목록용·프로필용·요약용 응답 객체를 나누고, SQL에서도 각 화면에 필요한 컬럼만 명시**한다. 프로필 조회 권한과 공개 가능한 필드도 업무 기준에 맞게 정한다.

### 2. 팀 이름과 조직 식별번호 구분하기

현재 화면은 본부 분류와 팀 이름을 키로 묶으며, 팀 수 집계는 `COUNT(DISTINCT pt.department_name)`과 같은 이름 기준이다. 다른 본부에 이름이 같은 팀이 있으면 조직도에는 각각 보이더라도 전체 팀 수는 적게 집계될 수 있다. 또한 `INNER JOIN`을 사용하므로 사원이 없는 팀이나 상위 부서와 연결되지 않은 사원은 목록에 포함되지 않는다. 현재 쿼리에는 재직·등록 상태 필터도 없어서, 집계 결과를 곧바로 **현재 재직자 수**로 해석할 수는 없다.

조직의 정확한 수와 구성이 필요하면 팀과 본부의 **ID를 기준으로 그룹화·집계**하고, 빈 팀과 퇴사자를 포함할지 정의한 뒤 조인 방식과 조회 조건을 맞춘다. 본부와 팀의 표시 순서가 중요하다면 서브쿼리 안의 정렬에 의존하지 않고 최종 조회에 `ORDER BY`를 적용한다.

### 3. 화면 상태와 예외 처리 보완하기

- **권한 표시:** 서버가 `ADMIN`을 `관리자`로 바꿔 내려주는데, 모달은 여전히 `ADMIN`을 비교한다. 이 상태에서는 관리자가 비관리자로 표시될 수 있으므로, 권한 코드는 그대로 보내고 브라우저에서 문구를 정하거나 별도 표시용 필드를 사용한다.
- **프로필 조회 실패:** 존재하지 않는 사원 ID나 네트워크 오류에 대한 처리를 추가한다. 서버는 미존재를 `404`로 응답하고, 브라우저는 `response.ok`를 확인해 실패 안내를 보여줄 수 있다. 사진 경로가 비어 있을 때는 기본 이미지를 지정한다.
- **펼치기와 스크롤 상태:** 팀별 토글을 해도 `allMembersExpanded`는 바뀌지 않아 전체 버튼 문구와 실제 상태가 어긋날 수 있다. 팀원 목록의 현재 상태에서 전체 버튼 문구를 계산하고, 모달 닫기 이벤트도 클릭마다 추가하지 않도록 정리한다.
- **HTML 렌더링:** 본부명·팀명·사원명이 `innerHTML` 템플릿 문자열에 직접 들어간다. 사용자 입력이 반영되는 값은 `textContent`로 삽입하고, 클릭은 `addEventListener()`로 연결하는 방식으로 바꾸면 HTML 삽입 위험을 줄일 수 있다.

---

## 구현 결과

조직도 화면에서 **본부와 팀 구조 확인, 팀원 목록의 개별·전체 펼치기, 사원 프로필 조회, PNG 다운로드**를 한곳에서 처리하도록 구성했다.<br> 
이 기능을 만들면서 DB에서 조회한 행을 화면에 필요한 계층 구조로 재구성하는 방법과, **처음 화면에 필요한 데이터는 서버 렌더링으로 전달하고 상세 데이터는 클릭할 때 조회하는 방법**을 함께 적용했다.
