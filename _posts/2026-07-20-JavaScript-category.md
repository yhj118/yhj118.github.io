---
title: JavaScript 기본 개념과 동작 원리
date: 2026-07-20 22:00:00 +0900
categories:
  - Develop
  - JavaScript
tags:
  - JavaScript
  - Node.js
---

# JavaScript 기본 개념과 동작 원리

## 1. JavaScript란?

JavaScript는 웹 페이지에 동적인 기능을 구현하거나 데이터를 처리할 때 사용하는 프로그래밍 언어이다.

HTML, CSS, JavaScript는 각각 다음과 같은 역할을 한다.

| 구분 | 역할 |
|---|---|
| HTML | 웹 페이지의 구조 정의 |
| CSS | 웹 페이지의 디자인과 스타일 |
| JavaScript | 이벤트 처리, 데이터 가공, 화면 변경 등의 동작 |

JavaScript는 웹 브라우저에서 주로 사용되지만, Node.js와 같은 실행 환경을 이용하면 브라우저 외부에서도 실행할 수 있다.

## 2. JavaScript의 실행 원리

JavaScript는 IntelliJ와 같은 개발 도구에서 코드를 작성했다고 해서 IntelliJ 자체에서 실행되는 것은 아니다.

JavaScript 코드를 실제로 실행하려면 JavaScript 엔진을 포함한 실행 환경이 필요하다.

대표적인 환경은 다음과 같다.

| 실행 환경 | JavaScript 엔진 | 특징 |
|---|---|---|
| Chrome | V8 | 웹 페이지에서 JavaScript 실행 |
| Firefox | SpiderMonkey | 웹 페이지에서 JavaScript 실행 |
| Safari | JavaScriptCore | 웹 페이지에서 JavaScript 실행 |
| Node.js | V8 | 브라우저 외부에서 JavaScript 실행 |

### 2.1. 브라우저에서 실행하는 과정

1. 개발자가 IntelliJ에서 HTML 및 JavaScript 코드를 작성한다.
2. 사용자가 브라우저에서 웹 페이지에 접근한다.
3. 브라우저가 HTML 및 JavaScript 파일을 불러온다.
4. 브라우저의 JavaScript 엔진이 코드를 실행한다.
5. JavaScript가 사용자 이벤트를 처리하거나 화면을 변경한다.

예시:

```javascript
document.getElementById("message").textContent = "안녕하세요";
```

위 코드는 브라우저가 제공하는 DOM API를 이용하여 HTML 요소의 내용을 변경한다.

즉, IntelliJ는 코드를 작성하는 도구이고, 실제 JavaScript 실행은 브라우저에서 이루어진다.

### 2.2. JavaScript 실행 환경이란?

JavaScript 엔진은 JavaScript 코드를 해석하고 실행하는 역할을 한다.

실행 환경은 JavaScript 엔진과 함께 프로그램 실행에 필요한 기능을 제공한다.

예를 들어 브라우저는 `document`, `window`와 같은 웹 페이지 관련 기능을 제공한다.

반면 Node.js는 파일 시스템 접근, 서버 생성 등 브라우저 외부에서 필요한 기능을 제공한다.

따라서 같은 JavaScript 언어를 사용하더라도 실행 환경에 따라 사용할 수 있는 기능이 다르다.

## 3. Node.js란?

Node.js는 **JavaScript를 웹 브라우저 외부에서 실행할 수 있도록 해주는 런타임 환경**이다.

Node.js는 V8 JavaScript 엔진을 기반으로 하며, JavaScript로 서버 프로그램이나 다양한 도구를 개발할 수 있도록 지원한다.

주요 사용 목적은 다음과 같다.

- JavaScript 파일 실행
- 백엔드 서버 및 REST API 개발
- 파일 읽기 및 쓰기
- 데이터 처리 및 자동화
- JavaScript 기반 개발 도구 실행

### 3.1. Node.js와 IntelliJ의 차이

Node.js는 IntelliJ처럼 코드 작성 화면을 제공하는 통합 개발 환경이 아니다.

| 구분 | IntelliJ | Node.js |
|---|---|---|
| 종류 | 통합 개발 환경(IDE) | JavaScript 런타임 |
| 코드 작성 | 가능 | 전용 코드 편집기 없음 |
| 코드 실행 | 실행 환경과 연동 | JavaScript 직접 실행 |
| 주요 역할 | 코드 작성, 편집, 디버깅 | JavaScript 프로그램 실행 |

즉, IntelliJ에서 코드를 작성하고 Node.js를 통해 실행할 수 있다.

### 3.2. Node.js 실행 예시

IntelliJ에서 `practice.js` 파일을 작성한다.

```javascript
const a = 10;
const b = 20;

console.log(a + b);
```

터미널에서 다음 명령어를 실행한다.

```bash
node practice.js
```

실행 결과:

```text
30
```

Node.js가 JavaScript 파일을 실행하고, `console.log()`의 출력 결과가 터미널에 나타난다.

**Node.js를 사용하면 Chrome을 실행하지 않고도 JavaScript 코드의 결과를 확인할 수 있다.**

### 3.3. Node.js 설치 확인

터미널에서 다음 명령어를 입력한다.

```bash
node -v
```

설치된 Node.js의 버전이 표시되면 정상적으로 사용할 수 있다.

Node.js를 설치한 후에는 IntelliJ뿐 아니라 VS Code나 다른 편집기로 작성한 JavaScript 파일도 실행할 수 있다.

### 3.4. Node.js의 대화형 실행

Node.js는 JavaScript 파일을 만들지 않고도 간단한 코드를 실행할 수 있다.

터미널에 다음 명령어를 입력한다.

```bash
node
```

이후 JavaScript 코드를 직접 입력할 수 있다.

```javascript
> 10 + 20
30
```

이러한 대화형 실행 환경을 REPL이라고 한다.

REPL을 종료하려면 `.exit`을 입력하면 된다.

## 4. Chrome과 Node.js의 차이

Chrome과 Node.js는 모두 JavaScript를 실행할 수 있지만 사용 목적이 다르다.

| 기능 | Chrome | Node.js |
|---|---|---|
| 변수, 조건문, 반복문 | 가능 | 가능 |
| 배열, 객체, 함수 | 가능 | 가능 |
| JSON 데이터 처리 | 가능 | 가능 |
| console.log() | 가능 | 가능 |
| DOM을 이용한 HTML 조작 | 가능 | 기본적으로 불가능 |
| 로컬 파일 시스템 접근 | 제한적 | 가능 |
| 백엔드 서버 개발 | 일반적인 용도 아님 | 가능 |

### 브라우저에서 실행하는 코드

```javascript
document.getElementById("title").textContent = "Hello";
```

`document`는 브라우저가 제공하는 객체이므로 일반적인 Node.js 환경에서는 사용할 수 없다.

### Node.js에서 실행하는 코드

```javascript
const fs = require("node:fs");

fs.writeFileSync("test.txt", "Hello Node.js");

console.log("파일 생성 완료");
```

Node.js에서는 파일 시스템 API를 이용하여 로컬 파일을 생성하거나 읽을 수 있다.

즉, JavaScript라는 언어는 같지만 실행 환경에 따라 제공되는 기능이 달라진다.

## 5. Spring Boot 프로젝트에서 JavaScript가 동작하는 과정

Spring Boot 기반 웹 애플리케이션에서는 일반적으로 Java와 프론트엔드 JavaScript가 서로 다른 환경에서 실행된다.

- Java: Spring Boot 서버에서 실행
- JavaScript: 사용자의 웹 브라우저에서 실행
- IntelliJ: Java 및 JavaScript 코드를 작성하는 개발 도구

### 5.1. 기본 실행 과정

1. IntelliJ에서 Java, HTML, JavaScript 코드를 작성한다.
2. Spring Boot 서버를 실행한다.
3. 사용자가 Chrome에서 웹 페이지에 접속한다.
4. Spring Boot 서버가 HTML 및 JavaScript 파일을 전달
