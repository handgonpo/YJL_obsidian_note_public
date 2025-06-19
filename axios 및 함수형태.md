🔹 Axios란?
	Axios는 JavaScript에서 HTTP 요청을 보내기 위한 라이브러리입니다.  
	쉽게 말하면, 웹페이지(JavaScript)에서 백엔드(API 서버)에 데이터를 보내거나 가져오는 도구입니다.

❓ Axios를 왜 쓰나요?
- `API 요청 간편화`	
	`fetch()`보다 코드가 짧고 직관적입니다.
- `응답 자동 처리`	
	JSON을 자동으로 파싱합니다.
- `에러 처리 간편`	
	`try...catch`로 예외를 쉽게 다룰 수 있습니다.
- `브라우저 호환성 좋음`	
	Internet Explorer까지 지원합니다.
- `요청 취소, 인터셉터 지원`	
	고급 기능도 내장돼 있습니다.

자바스크립트 함수는 기본적으로 현재 로드된 HTML 페이지에서만 동작합니다.

---
📖 axios 문법:
```js
axios.get('https://example.com/api/data') //서버에 GET 요청
  .then(response => {
    console.log(response.data);  // 요청 성공시 실행
  })
  .catch(error => {
    console.error(error);  // 요청 실패시 실행
  });
```

📖 클릭이벤트 + axios:
```js
<button id="loadBtn">데이터 불러오기</button>

<script>
document.getElementById('loadBtn').addEventListener('click', function () {
  axios.get('/api/data')
    .then(function (response) {
      console.log("응답 데이터:", response.data);
    })
    .catch(function (error) {
      console.error("에러 발생:", error);
    });
});
</script>
```
---
`axios`와 `axiosInstance`는 같은 Axios 라이브러리를 기반으로 하지만, 사용 방식에 차이가 있습니다.

| 항목   | axios            | axiosInstance                  |
| ---- | ---------------- | ------------------------------ |
| 정의   | Axios 기본 객체      | 사용자가 설정한 맞춤형 인스턴스              |
| 특징   | 매번 직접 옵션 설정      | 공통 설정을 미리 지정하고 재사용 가능          |
| 사용 예 | `axios.get(...)` | `axiosInstance.get(...)`       |
| 활용도  | 간단한 요청에 적합       | 공통 헤더, 토큰, 기본 URL 등이 필요한 경우 유용 |

---
📖 기본 함수 선언:
```js
function sayHello() {
  console.log("안녕하세요!");
}

sayHello();  // 호출
```
`function 함수이름()`으로 시작하면 어디서든 호출 가능

📖 함수 표현식 (Expression):
```js
const sayHi = function () {
  console.log("안녕!");
};

sayHi();  // 호출
```
함수도 변수처럼 다룰 수 있고, 함수명이 없어도 됨 (익명 함수)

📖 화살표 함수 (Arrow Function):
```js
const multiply = (x, y) => {
  return x * y;
};

console.log(multiply(2, 4));  // 8
```
한 줄일 때는 생략 가능:
```js
const double = n => n * 2;
```
n은 함수의 매개변수(파라미터)입니다.


📖 콜백 함수 (함수 안에 함수):
```js
function greet(name, callback) {
  console.log("안녕하세요, " + name);
  callback();  // 함수 실행
}

greet("철수", function () {
  console.log("콜백 함수 실행!");
});
```
`greet("철수", ...)` : greet 함수 호출
`"철수"` : 첫번째 인자
`function() { ... }` : 두번째 인자 - 콜백함수(익명함수정의)
greet 함수 안의 `callback()` : 이때 전달된 함수가 실제로 실행되는 위치

```js
             // ↓ 두 번째 인자 (익명 함수)
greet("철수", function () {
  console.log("콜백 함수 실행!");
});

// →  greet 함수 안으로 들어감
    name = "철수"
    callback = function () { ... }

greet 내부에서:
  console.log("안녕하세요, 철수");
  callback();  // →  여기서 아까 전달된 함수를 실행!
```
---
📖 이벤트 핸들러 함수:
```js
document.getElementById('myBtn').addEventListener('click', function () {
  alert("버튼이 클릭되었습니다!");
});
```

---
모듈화한 함수들이 모두 서로를 호출하는 연쇄(chain) 구조이기 때문에, 하나라도 빠뜨리면 그 다음 단계로 넘어가지 못해서 리스트가 아예 그려지지 않습니다.
그래서 처음부터 기능별 섹션 모듈을 나누어 짜두는것이 중요합니다.
```JS
// 1) 설정(Config) ----------------------------
const axiosInstance = …;

// 2) 초기화 & 이벤트 바인딩(Init) ------------
function init() { … }
function bindUIEvents() { … }

// 3) 데이터 처리(Data) -------------------------
function fetchTodoData(page) { … }
function extractTodoArray(data) { … }

// 4) 렌더링(View) -----------------------------
function renderTodoList(todos) { … }
function createTodoElement(todo) { … }
function renderPagination(data, page) { … }

// 5) 핸들러(Handlers) --------------------------
function onCreateClick() { … }
function detailView(id) { … }
function toComplete(id) { … }

// 6) 유틸(Util) -------------------------------
function datetimeToString(str) { … }

// 7) 실행(Entry Point) -------------------------
document.addEventListener('DOMContentLoaded', init);
```

