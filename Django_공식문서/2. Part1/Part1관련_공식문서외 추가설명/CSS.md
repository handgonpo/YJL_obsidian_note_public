
- `reset.css`: 모든 스타일을 제거 (브라우저 차이 최소화)
```css
/* 전체 초기화 스타일 */
*,
*::before,
*::after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body {
  height: 100%;
  font-family: sans-serif;
}

a {
  color: inherit;
  text-decoration: none;
}

ul, ol {
  list-style: none;
}

button {
  background: none;
  border: none;
  cursor: pointer;
  font-family: inherit;
}

img, video {
  max-width: 100%;
  height: auto;
  display: block;
}

```
---

`normalize.css`: 브라우저 기본 스타일을 유지하되, 일관성 있게 조정
```css
/*! normalize.css v8.0.1 | MIT License | github.com/necolas/normalize.css */

/* 1. 기본 HTML5 요소 스타일 초기화 */
html {
  line-height: 1.15; /* 1 */
  -webkit-text-size-adjust: 100%; /* 2 */
}

/* 2. Body 기본 마진 제거 */
body {
  margin: 0;
}

/* 3. h1 기본 크기 조정 */
h1 {
  font-size: 2em;
  margin: 0.67em 0;
}

/* 4. HTML5 요소를 블록요소로 처리 */
main {
  display: block;
}

/* 5. strong, b → bold 처리 유지 */
strong, b {
  font-weight: bolder;
}

/* 6. 코드 및 문장요소 기본 폰트 설정 */
code, kbd, samp {
  font-family: monospace, monospace;
  font-size: 1em;
}

/* 7. 작은 글씨 요소 크기 */
small {
  font-size: 80%;
}

/* 8. 서브스크립트 요소 위치 초기화 */
sub, sup {
  font-size: 75%;
  line-height: 0;
  position: relative;
  vertical-align: baseline;
}

sub {
  bottom: -0.25em;
}

sup {
  top: -0.5em;
}

/* 9. 링크 스타일 */
a {
  background-color: transparent;
  text-decoration: none;
}

/* 10. 이미지 반응형 처리 */
img {
  border-style: none;
  max-width: 100%;
  height: auto;
}

/* 11. 버튼 및 폼 요소 초기화 */
button, input, optgroup, select, textarea {
  font-family: inherit;
  font-size: 100%;
  line-height: 1.15;
  margin: 0;
}

button, input {
  overflow: visible;
}

button, select {
  text-transform: none;
}

/* 12. 버튼 스타일 초기화 */
button,
[type="button"],
[type="reset"],
[type="submit"] {
  -webkit-appearance: button;
}

/* 13. 텍스트 영역 */
textarea {
  overflow: auto;
  resize: vertical;
}

/* 14. 박스 사이징 기본 설정 */
*, *::before, *::after {
  box-sizing: border-box;
}

```