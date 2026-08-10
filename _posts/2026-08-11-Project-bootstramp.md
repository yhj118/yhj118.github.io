---
title: 부트스트랩 적용
date: 2026-08-11
categories: 
  - Project
  - Intranet GroupWare
tags: 
  - Project 
---

# Intranet Groupware

1. 부트스트랩?
CSS와 JS를 미리 만들어놓아서 UI를 쉽게 적용하는 기술.
부트스트랩이 없다면 CSS와 JS를 개발자가 하나하나 작성해야 한다.

2. 부트스트랩 다운로드 ~ 적용하기.
 1) CSS와 JS는 정적 파일이므로 관리할 폴더를 생성.
    경로는 src/main/resource/static에 tabler 폴더와 하위로 css & js 폴더를 생성한다.
    static 폴더 바로 아래에 있는 css와 js 폴더는 부트스트랩인 tabler가 아닌 개발하면서 별도로 작성한 소스를 관리하는 곳이며 이것도 생성한다.
    ![](/assets/img/bootstrap1.png)

 2) tabler의 css와 js를 터미널에서 다운로드한다.
    curl -fL "https://cdn.jsdelivr.net/npm/@tabler/core@1.4.0/dist/css/tabler.min.css" -o src/main/resources/static/tabler/css/tabler.min.css
    curl -fL "https://cdn.jsdelivr.net/npm/@tabler/core@1.4.0/dist/js/tabler.min.js" -o src/main/resources/static/tabler/js/tabler.min.js

 3) html 파일을 하나 만들어 적용해본다.
<!DOCTYPE html>
<html lang="ko" xmlns:th="http://www.thymeleaf.org"> <!-- xmlns?? Thymeleaf 문법 사용을 위한 선언 -->
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link rel="stylesheet" th:href="@{/tabler/css/tabler.min.css}"> <!-- css 적용 링크 -->
    <title>Tabler 적용 테스트</title>
</head>
<body>
    <div class="container-xl py-5">
        <div class="card">
            <div class="card-body">
                <h1 class="card-title">
                    Intranet Groupware
                </h1>
                <p class="text-secondary">
                    Tabler 적용 테스트 화면입니다.
                </p>
                <button type="button" class="btn btn-primary">
                    로그인
                </button>
                <button type="button" class="btn btn-outline-danger">
                    취소
                </button>
            </div>
        </div>
    </div>

    <script th:src="@{/tabler/js/tabler.min.js}"></script> <!-- js 적용 링크 -->

</body>
</html>


