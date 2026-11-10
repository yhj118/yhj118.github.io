---
title: "내부망 개발일지"
date: 2026-09-14 00:00:00 +0900
categories:
  - Project
  - Intranet GroupWare
tags:
  - Project
  - Spring Security
  - Network
  - Deployment
---

# Intranet Groupware

<br>

## 1. 내부망 구성 목표

> 그룹웨어를 맥에서 실행하고, 허용된 내부망의 기기만 접속할 수 있도록 구성한다. 인터넷에 연결하지 않아도 주요 화면과 기능을 사용할 수 있도록 외부 리소스 의존도도 줄인다.

내부망을 구성하면서 두 가지를 나눠 확인했다. **첫째는 어떤 기기가 서버에 접속할 수 있는가**, **둘째는 접속한 뒤 화면을 사용하는 데 인터넷이 필요한가**다. 같은 Wi-Fi에 연결돼도 서버가 요청을 받지 못할 수 있고, 접속에 성공해도 외부 CDN의 스크립트를 불러오지 못하면 버튼이나 모달이 동작하지 않을 수 있다.

<br>

| 구성 | 맡은 역할 |
| --- | --- |
| Mac · Spring Boot · 내장 Tomcat | 그룹웨어를 실행하고 `8081` 포트에서 HTTP 요청 처리 |
| MariaDB | 같은 맥에서 실행하며 사원·게시판·결재·일정 데이터 저장 |
| Spring Security · `InternalNetworkFilter` | 요청의 IP 대역 검사와 로그인 접근 제어 |
| Thymeleaf · 로컬 정적 리소스 | Tabler CSS·JS 및 조직도 캡처 라이브러리 제공 |
| Gradle · 실행용 JAR | 애플리케이션과 의존성을 묶어 IDE 없이 실행 |

<br>

## 2. 맥을 서버로 사용하기

Spring Boot와 MariaDB를 맥에서 실행했다. 맥 자체에서 테스트할 때는 `localhost:8081`을 사용하고, 같은 Wi-Fi에 연결한 휴대폰에서는 **맥의 내부 IP와 포트**로 접속했다.

<br>

| 접속하는 기기 | 주소 예시 |
| --- | --- |
| 서버를 실행한 맥 | `http://localhost:8081/openLogin` |
| 같은 내부망의 휴대폰 | `http://192.168.1.25:8081/openLogin` |

<br>

두 번째 주소의 `192.168.1.25`는 예시다. 실제 테스트에서는 맥에 할당된 내부 IP를 사용한다. 휴대폰에서 `localhost`를 입력하면 맥이 아니라 **휴대폰 자신**을 가리키므로, 맥에서 실행 중인 서버에 연결되지 않는다.

<br>

접속 흐름은 **휴대폰 브라우저 → 같은 Wi-Fi → 맥의 8081 포트 → Spring Boot → 필요하면 MariaDB 조회** 순서다. 휴대폰이 DB에 직접 연결하지는 않는다. 브라우저는 웹 서버에 요청을 보내고, Spring Boot가 맥에서 실행 중인 MariaDB에 연결한다.

<br>

> **한 줄 요약**<br>
> 같은 내부망의 다른 기기는 서버를 실행한 맥의 IP와 포트로 접속한다.

<br>

## 3. CIDR로 접속 허용 범위 설정

내부망에 연결된 기기라도 모든 요청을 허용하는 대신, `application.properties`에서 접근을 허용할 IP 대역을 지정했다. 쉼표로 나열한 대역은 서버 자체 접속을 위한 **IPv4·IPv6 루프백**과 **내부망 IPv4 대역**이다.

<br>

아래는 실제 값을 그대로 옮긴 것이 아닌 설정 예시다. 마지막 대역은 서버를 설치한 네트워크에 맞게 바꾼다.

```properties
server.port=8081
groupware.network.allowed-cidrs=127.0.0.1/32,::1/128,192.168.1.0/24
```

| CIDR | 허용 범위 |
| --- | --- |
| `127.0.0.1/32` | 맥 자체에서 접속하는 IPv4 루프백 주소 하나 |
| `::1/128` | 맥 자체에서 접속하는 IPv6 루프백 주소 하나 |
| `192.168.1.0/24` | 앞 24비트가 `192.168.1`에 해당하는 IPv4 대역 |

<br>

`192.168.1.0/24`는 접속할 서버 주소가 아니라, **요청을 보낸 IP가 속해도 되는 범위**다. 예를 들어 휴대폰 IP가 `192.168.1.37`이면 이 대역에 속하지만, `192.168.2.37`이면 속하지 않는다. 반대로 브라우저 주소창에 넣는 것은 접속 대상인 맥의 주소다.

<br>

## 4. Spring Security에 내부망 필터 추가

### 요청 IP 검사

`InternalNetworkFilter`는 `OncePerRequestFilter`를 상속한다. 생성자에서 허용 대역 문자열을 쉼표로 나누고, 각 항목의 공백을 제거한 뒤 Spring Security의 `IpAddressMatcher`로 만든다. 요청이 오면 `request.getRemoteAddr()`로 서버가 확인한 접속 IP를 읽고, 허용 목록과 비교한다.

<br>

아래는 `doFilterInternal()`의 핵심 부분이다.

```java
String clientIp = request.getRemoteAddr();

for (IpAddressMatcher network : allowedNetworks) {
    if (network.matches(clientIp)) {
        filterChain.doFilter(request, response);
        return;
    }
}

response.setStatus(HttpServletResponse.SC_FORBIDDEN);
response.setContentType("text/plain;charset=UTF-8");
response.getWriter().write("허용된 내부망에서만 접속할 수 있습니다.");
```

- **허용 대역에 속함:** `filterChain.doFilter()`로 다음 처리 단계에 요청을 넘긴다.
- **허용 대역에 속하지 않음:** `403 Forbidden`과 안내 문구를 응답한다. 다음 필터나 Controller에는 요청을 넘기지 않는다.

<br>

### IP 검사와 로그인 검사 구분

`SecurityConfig`에서 `@Value`로 허용 CIDR 설정을 읽는다. `http.addFilterBefore()`를 사용해 `InternalNetworkFilter`를 **`CsrfFilter` 앞에** 등록했다.

```java
@Value("${groupware.network.allowed-cidrs}")
private String allowedCidrs;

// SecurityFilterChain 구성 중
http.addFilterBefore(
        new InternalNetworkFilter(allowedCidrs),
        CsrfFilter.class
);
```

IP 검사에 통과했다는 것은 **허용된 네트워크에서 요청이 들어왔다**는 뜻이다. 로그인한 것으로 처리되지는 않는다. 로그인·사원 등록처럼 `permitAll()`로 지정한 경로를 제외하면, 기존 Spring Security 설정에 따라 로그인한 사용자만 접근할 수 있다. 반대로 `permitAll()` 경로라도 IP 필터에서 차단되면 열리지 않는다.

<br>

> **한 줄 요약**<br>
> IP 필터는 어느 네트워크에서 요청할 수 있는지 정하고, 로그인 설정은 그 안에서 누가 기능을 사용할 수 있는지 정한다.

<br>

## 5. 인터넷 의존 리소스 제거

내부망에서는 브라우저가 인터넷에 나갈 수 없더라도 화면이 열려야 한다. CDN에서 불러오던 파일은 프로젝트에 포함하고, 외부 주소 검색은 직접 입력 방식으로 바꿨다.

<br>

| 변경한 부분 | 적용 방식 |
| --- | --- |
| Tabler CSS·JS | `static/tabler/`의 로컬 파일 사용. 스타일과 모달 동작을 그룹웨어 서버에서 제공한다. |
| 조직도 이미지 저장 | `static/js/html2canvas.min.js`를 포함해 조직도 화면에서 같은 서버의 파일을 불러온다. |
| 사원 주소 | 다음 주소 검색을 제거하고, 사원 등록·수정 폼에 도로명주소와 상세주소를 직접 입력한다. |

<br>

예를 들어 조직도에서는 다음과 같이 로컬 경로를 사용한다.

```html
<link rel="stylesheet" th:href="@{/tabler/css/tabler.min.css}">
<script th:src="@{/tabler/js/tabler.min.js}"></script>
<script th:src="@{/js/html2canvas.min.js}"></script>
```

`@{/...}`는 Thymeleaf가 같은 웹 애플리케이션의 리소스 URL로 렌더링한다. 브라우저는 외부 CDN 대신 그룹웨어 서버에 해당 파일을 요청한다.

<br>

> **한 줄 요약**<br>
> 화면에 필요한 CSS와 스크립트를 프로젝트에 포함하고, 외부 주소 검색을 직접 입력으로 대체했다.

<br>

## 6. 방화벽과 실행 상태 확인

맥의 방화벽을 켠 상태에서도 같은 Wi-Fi의 휴대폰으로 접속되는지 확인했다. **맥 방화벽**은 서버까지 연결을 받아들일지 결정하고, **Spring Security의 IP 필터**는 서버에 도착한 HTTP 요청을 통과시킬지 결정한다.

<br>

또한 전원에 연결한 상태에서 **화면이 꺼져도 맥이 자동으로 잠들지 않도록** 설정하고, 화면이 꺼진 뒤 휴대폰에서 다시 접속했다. 화면이 꺼져도 운영체제와 서버 프로세스가 실행 중이면 요청을 처리할 수 있지만, 맥이 잠들면 접속이 끊길 수 있다.

<br>

| 점검 항목 | 확인 내용 |
| --- | --- |
| 맥에서 접속 | `localhost:8081`로 페이지를 열 수 있는지 확인 |
| 같은 Wi-Fi의 휴대폰 | 맥의 내부 IP와 `8081` 포트로 접속되는지 확인 |
| 허용 대역 밖 요청 | 요청이 서버까지 도달했다면 IP 필터의 `403` 안내가 반환되는지 확인 |
| 휴대폰에서 Wi-Fi 끄기 | 휴대폰이 셀룰러로 바뀌면 맥의 내부 IP에 접속되지 않는지 확인 |
| 인터넷 없이 화면 확인 | 맥 Wi-Fi를 끈 뒤 새 시크릿 창에서 `localhost:8081`에 접속해 주요 화면과 기능이 동작하는지 확인 |
| 화면 끄기 | 맥이 잠들지 않은 상태에서 화면만 꺼졌을 때 휴대폰 접속이 유지되는지 확인 |

<br>

휴대폰에서 Wi-Fi를 껐을 때 페이지가 안 열리는 것과 `403` 응답은 의미가 다르다. **접속 실패는 요청이 서버에 닿지 않았을 수 있고, `403`은 서버가 요청을 받아 거부했다는 뜻**이다. 또한 맥에서 Wi-Fi를 끄고 `localhost`를 연 테스트는 인터넷 없이 앱이 동작하는지 확인한 것이다. Wi-Fi가 끊긴 상태에서 다른 기기가 맥에 접속하는 테스트는 아니다.

<br>

## 7. 실행용 JAR로 빌드하고 실행

개발 중에는 IntelliJ에서 Spring Boot를 실행했지만, 내부망에서 사용할 때는 **IDE를 열지 않고도 서버를 실행**할 수 있도록 Gradle의 `bootJar`로 패키징했다.

<br>

프로젝트 루트에서 실행한다.

```bash
./gradlew bootJar
```

생성 파일은 `build/libs/groupware-0.0.1-SNAPSHOT.jar`다. JAR에는 컴파일된 애플리케이션, Thymeleaf 템플릿, 정적 파일, 의존 라이브러리와 내장 Tomcat이 들어 있다. **MariaDB의 데이터와 실행 중에 업로드한 파일은 JAR에 함께 들어가지 않는다.**

<br>

IntelliJ에서 기존 서버를 종료해 `8081` 포트를 비우고, MariaDB를 실행한 상태에서 다음 명령으로 그룹웨어를 실행한다.

```bash
java -jar build/libs/groupware-0.0.1-SNAPSHOT.jar
```

`java -jar`로 JVM이 시작되면 Spring Boot 애플리케이션과 내장 Tomcat이 함께 실행된다. 별도 Tomcat을 설치할 필요는 없다. 다른 컴퓨터에서 실행한다면 호환되는 Java 실행 환경과 MariaDB 연결 설정을 준비해야 한다. 업로드 경로가 실행 위치를 기준으로 잡혀 있으므로, 기존 파일도 새 환경의 `uploads/` 경로에 따로 준비한다.

<br>

> **한 줄 요약**<br>
> Gradle로 만든 JAR를 실행해 IntelliJ 없이 웹 서버를 띄우고, 브라우저로 접속을 확인한다.

<br>

## 주요 이슈 및 개선 방향

### 1. 실행 시 URL 매핑과 뷰 이름 점검

`java -jar`로 실행한 뒤에는 접속이 되는지만 확인하지 않고 전자결재 같은 주요 메뉴도 열어봤다. Thymeleaf 뷰 이름은 `electronicApproval/eAMain`처럼 `templates` 아래의 경로로 반환한다. `/openEAMain`처럼 브라우저가 요청하는 URL과는 구분한다.

<br>

첨부한 소스에는 `AuthController`와 `MainController`가 둘 다 `GET /`를 선언하는 부분이 남아 있다. 그대로 빌드할 때 매핑 충돌로 애플리케이션이 시작되지 않으면 **로그인 화면은 `/openLogin`, 메인은 `/openMain`으로 구분하고 `/`는 한 곳에서만 처리**하도록 확인한다. 수정했다면 새 JAR를 빌드한 뒤 다시 실행해야 변경이 반영된다.

<br>

### 2. 접속이 안 될 때 원인을 단계별로 나누기

1. 맥에서 `localhost:8081`도 안 열리면 애플리케이션이 실행 중인지, 시작 로그에 포트·DB·URL 매핑 오류가 있는지 확인한다.
2. 맥에서는 열리는데 다른 기기에서 연결되지 않으면 접속 주소, 같은 내부망 여부, 맥 방화벽부터 확인한다.
3. 내부망 안내와 함께 `403`이 나오면 서버까지는 요청이 도달한 것이다. 서버에서 보이는 접속 IP와 `allowed-cidrs` 설정을 비교한다.
4. 로그인 화면은 열리지만 화면이 깨지거나 모달이 동작하지 않으면 브라우저 개발자 도구에서 CSS·JS 요청 결과를 확인한다.

<br>

### 3. 배포 환경이 달라질 때 확인할 부분

현재 필터는 맥이 직접 요청을 받는 환경을 기준으로 `getRemoteAddr()`를 사용한다. 나중에 앞단에 프록시를 두면 이 값에 프록시 IP가 들어올 수 있다. 그때는 원본 IP를 어디에서 판별할지, 어떤 프록시를 신뢰할지 배포 구조에 맞게 정해야 한다.

<br>

허용 대역 설정은 접근 범위를 제한하는 기능이다. 현재 테스트의 `http://` 연결을 암호화하지는 않는다. 실제 사내 배포에서 전송 암호화가 필요하다면 HTTPS 설정을 별도로 적용한다.

<br>

## 구현 결과

맥에서 실행한 그룹웨어에 같은 내부망의 휴대폰으로 접속하고, **허용 IP 대역과 로그인 여부를 각각 검사**하도록 구성했다. Tabler와 html2canvas는 로컬 파일로 전환하고 주소는 직접 입력하도록 바꿔, 인터넷 연결 없이도 주요 화면을 확인할 수 있었다.

<br>

이 과정을 통해 **다른 기기가 서버에 연결되는 것**, **서버가 요청을 통과시키는 것**, **브라우저가 화면에 필요한 파일을 불러오는 것**을 따로 확인해야 한다는 점을 정리했다.
