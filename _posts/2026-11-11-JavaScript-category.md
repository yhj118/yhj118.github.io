---
title: JSON 기본 문법
date: 2026-11-11 22:00:00 +0900
categories:
  - Develop
  - JavaScript
tags:
  - JSON
---

JSON 기본 문법과 JavaScript에서 JSON을 다루는 방법을 연습한 코드입니다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>JSON 연습</title>
</head>
<body>

<h1 id="JStest1"></h1>
<hr>
<h1 id="JStest2"></h1>
<hr>
<h1 id="JStest3"></h1>
<hr>
<h1 id="JStest4"></h1>
<hr>
<h1 id="JStest5"></h1>
<hr>
<h1 id="JStest6"></h1>
<hr>
<h1 id="JStest7"></h1>
<hr>
<h1 id="JStest8"></h1>
<hr>
<h1 id="JStest9"></h1>
<hr>

<script>
    // 0. JSON
    // JSON은 데이터를 텍스트 형태로 표현하는 텍스트 포맷 표기법이다.
    // JavaScript뿐만 아니라 Java, Python 등 여러 언어에서 사용한다.
    // JavaScript에는 JSON.stringify()와 JSON.parse()가 기본 제공된다.


    // 1. JSON 객체의 기본 형태
    // { "key": value }
    // JSON 객체는 키(key)와 값(value)의 쌍으로 구성된다.
    // 키는 반드시 큰따옴표로 감싼 문자열이어야 하고 객체에는 여러 속성을 넣을 수 있다.
    // JSON은 객체뿐 아니라 배열, 문자열, 숫자, Boolean, null도 표현할 수 있다.


    // 2. JSON.stringify()와 JSON.parse()

    // test1은 JavaScript 객체이다.
    const test1 = {
        "이름": "유훈재"
    };

    document.getElementById("JStest1").textContent = JSON.stringify(test1);
    // JSON.stringify()는 JavaScript 값을 JSON 문자열로 변환한다.
    // 화면 출력: {"이름":"유훈재"}

    // test2는 JSON 형식의 문자열이다.
    const test2 = '{ "나이": "35살" }';

    // JSON.parse()는 JSON 문자열을 JavaScript 값으로 변환한다.
    // 여기서는 객체 형태의 JSON 문자열이므로 JavaScript 객체가 된다.
    const parsedTest2 = JSON.parse(test2);

    document.getElementById("JStest2").textContent = parsedTest2["나이"];

    // 화면 출력: 35살
    // 개별 속성 값을 출력할 때는 JSON.stringify()가 필요하지 않다.
    // 일반 객체 전체를 textContent에 직접 대입하면
    // 보통 [object Object]라는 문자열로 표시된다.


    // 3. 객체에 여러 속성 넣기
    const test3 = {
        "이름": "유훈재",
        "나이": 35,
        "생년월일": "1992-06-09",
        "진실": true,
        "퇴사일": null
    };

    document.getElementById("JStest3").textContent = JSON.stringify(test3);

    // 속성과 속성 사이는 쉼표로 구분한다.
    // 여러 속성이 들어 있어도 하나의 객체이다.
    // 문자열, 숫자, Boolean, null 등 여러 종류의 값을 담을 수 있다.
    // JSON에는 별도의 날짜 자료형이 없으므로 생년월일은 문자열로 표현했다.


    // 4. JavaScript 객체에 속성 추가하기
    const test4 = {
        "이름": "유훈재",
        "나이": 35,
        "생년월일": "1992-06-09"
    };

    test4["주거지"] = "서울";

    document.getElementById("JStest4").textContent = JSON.stringify(test4);

    // 객체["속성 이름"] = 값; 형태로 속성을 추가할 수 있다.
    // 점 표기법도 사용할 수 있다.
    // 예: test4.주거지 = "서울";
    // 이미 존재하는 속성에 값을 대입하면 해당 값이 수정된다.
    // 속성을 추가하거나 수정하는 작업은 JavaScript의 기능이다.
    // const로 선언해도 객체 내부의 속성은 변경할 수 있다.


    // 5. 배열과 배열 안의 객체

    // 배열에는 여러 값을 순서대로 넣는다.
    const test5 = ["커피", 3500, false, null];

    document.getElementById("JStest5").textContent = JSON.stringify(test5);

    // 배열은 대괄호 []로 묶는다.
    // 객체처럼 키와 값의 쌍이 아니라 값들을 쉼표로 구분하여 나열한다.
    // JavaScript에서는 0부터 시작하는 인덱스로 배열의 값에 접근한다.
    // test5[0] → "커피"
    // test5[1] → 3500
    // test5[2] → false
    // test5[3] → null

    // 배열 안에는 객체도 넣을 수 있다.
    const test6 = [
        {
            "이름": "유훈재",
            "나이": 35
        },
        {
            "이름": "홍길동",
            "나이": 20
        }
    ];

    document.getElementById("JStest6").textContent = JSON.stringify(test6);

    // test6은 객체 두 개를 담은 배열이다.
    // test6[0] → 첫 번째 사원 객체
    // test6[0]["이름"] → "유훈재"
    // test6[1]["이름"] → "홍길동"

    // 추가할 객체를 변수에 담아서 배열에 넣기
    const test7 = [
        {
            "이름": "유훈재",
            "나이": 35
        },
        {
            "이름": "홍길동",
            "나이": 20
        }
    ];

    const plusData = {
        "이름": "유관순",
        "나이": 25
    };

    test7.push(plusData);

    document.getElementById("JStest7").textContent = JSON.stringify(test7);

    // push()는 JavaScript 배열의 맨 뒤에 값을 추가하는 메서드이다.
    // JSON 자체에 데이터를 추가하는 기능이 있는 것은 아니다.


    // 6. 중첩 구조
    const test8 = {
        "부서": "개발팀",
        "위치": {
            "도시": "서울",
            "층": 3
        },
        "사원목록": [
            {
                "이름": "유훈재",
                "나이": 35
            },
            {
                "이름": "홍길동",
                "나이": 20
            }
        ]
    };

    document.getElementById("JStest8").textContent = JSON.stringify(test8);

    // 중첩 구조는 객체나 배열 안에 또 다른 객체나 배열이 들어 있는 구조이다.
    // "부서"의 값은 문자열이다.
    // "위치"의 값은 객체이다.
    // "사원목록"의 값은 객체 두 개를 담은 배열이다.

    // 중첩된 값에 접근하는 예
    // test8["부서"] → "개발팀"
    // test8["위치"]["도시"] → "서울"
    // test8["위치"]["층"] → 3
    // test8["사원목록"][0]["이름"] → "유훈재"


    // 7. 중첩된 값을 꺼내 HTML 화면에 표시하기
    const people = [
        {
            "이름": "유훈재",
            "주소": {
                "도시": "서울",
                "구": "구로구"
            }
        },
        {
            "이름": "홍길동",
            "주소": {
                "도시": "부산",
                "구": "사하구"
            }
        }
    ];

    let t1 = "";

    for (let i = 0; i < people.length; i++) {
        // 두 번째 사람부터 앞에 구분자를 붙인다.
        if (i > 0) {
            t1 += " / ";
        }

        t1 += people[i]["이름"] + " "
            + people[i]["주소"]["도시"] + " "
            + people[i]["주소"]["구"];
    }

    document.getElementById("JStest9").textContent = t1;

    // people[i]는 배열의 i번째 객체이다.
    // people[i]["주소"]는 해당 사람의 주소 객체이다.
    // people[i]["주소"]["도시"]는 주소 객체 안의 도시 값이다.
    // t1은 이미 문자열이므로 그대로 textContent에 대입한다.
    // 화면 출력: 유훈재 서울 구로구 / 홍길동 부산 사하구


    // 8. JSON 문법 규칙
    /*
    1. 키는 반드시 큰따옴표로 감싼다.
       올바른 예: { "이름": "유훈재" }
       잘못된 예: { 이름: "유훈재" }
       잘못된 예: { '이름': "유훈재" }

    2. 문자열도 반드시 큰따옴표로 감싼다.
       올바른 예: { "이름": "유훈재" }
       잘못된 예: { "이름": '유훈재' }

    3. 키와 값은 콜론(:), 항목 사이는 쉼표(,)로 구분한다.
       객체: { "이름": "유훈재", "나이": 35 }
       배열: ["개발팀", "인사팀", "영업팀"]

    4. 마지막 항목 뒤에는 쉼표를 넣지 않는다.
       올바른 예: { "이름": "유훈재", "나이": 35 }
       잘못된 예: { "이름": "유훈재", "나이": 35, }
       올바른 예: [10, 20, 30]
       잘못된 예: [10, 20, 30,]

    5. 값으로 사용할 수 있는 종류는 여섯 가지다.
       - 문자열: "유훈재"
       - 숫자: 35, 3.14, -10
       - Boolean: true, false
       - null: null
       - 객체: { "도시": "서울" }
       - 배열: ["개발팀", "인사팀"]

    6. true, false, null은 소문자로 작성하며 따옴표를 붙이지 않는다.
       true는 Boolean, "true"는 문자열이다.
       null은 null 값, "null"은 문자열이다.

    7. undefined, 함수, NaN, Infinity는 값으로 사용할 수 없다.

    8. JSON 데이터 자체에는 주석을 작성할 수 없다.
       이 설명은 JavaScript 주석이며, JSON 데이터가 아니다.

    9. JSON 자체에는 변수 선언, 대입문, 함수 호출을 작성할 수 없다.
       const person = { "이름": "유훈재" }; 는 JavaScript 코드다.
       { "이름": "유훈재" } 는 JSON 문법으로 표현한 데이터다.

    10. 문자열 안의 특수문자는 이스케이프한다.
        - 큰따옴표: \"
        - 역슬래시: \\
        - 줄바꿈: \n
        - 탭: \t
        예: { "대사": "그가 \"안녕\"이라고 말했다." }
        문자열 안에서 실제로 엔터를 눌러 줄을 바꾸면 안 된다.

    11. 숫자 표기에 주의한다.
        - 불필요한 앞자리 0 금지: 035 → 35
        - 소수점 앞 숫자 필요: .5 → 0.5
        - 양수 앞의 + 기호 금지: +10 → 10
        - 소수점 뒤 숫자 필요: 3. → 3.0
        - 지수 표기 가능: 1e3

    12. 문자열 바깥에는 공백, 줄바꿈, 들여쓰기를 넣을 수 있다.
        데이터를 한 줄로 쓰거나 여러 줄로 나눠도 의미는 같다.

    13. JSON의 최상위 값은 객체나 배열일 필요는 없다.
        "문자열", 35, true, null도 각각 유효한 JSON이다.

    14. 같은 객체 안에서는 키가 중복되지 않도록 작성한다.
        중복 키는 처리하는 프로그램에 따라 결과가 달라질 수 있다.
        피해야 할 예: { "이름": "유훈재", "이름": "홍길동" }
    */
</script>

</body>
</html>
```
