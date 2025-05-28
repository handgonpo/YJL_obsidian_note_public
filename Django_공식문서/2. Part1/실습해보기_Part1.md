영화 소개 페이지 – HTML로 출력하기

🔹 목적:
- 데이터베이스 없이, 파이썬 코드에 영화 데이터를 직접 작성
- 뷰에서 `HttpResponse()`로 HTML 코드 반환
- 영화 제목, 장르, 줄거리, 추천 버튼(가짜 링크) 등을 HTML로 꾸며보기

✅ 작업순서:
- 작업 폴더 만들기
- 가상환경 생성 및 활성화
- Django 설치
- Django 프로젝트 생성 : 
	  `mysite/` 폴더와 `manage.py`가 생성됨
- 앱 생성
- 앱 등록 (프로젝트에 앱 연결) : 
	  `mysite/settings.py`의 `INSTALLED_APPS`에 `movies` 추가
- 기본 뷰 함수 작성 : `movies/views.py`
- URL 연결 설정 : 
	  앱 내부에 `urls.py` 생성 (`movies/urls.py`)
	  메인 프로젝트의 `urls.py`에 앱 포함 (`mysite/urls.py`)
- 개발 서버 실행 : `http://127.0.0.1:8000/movies/`
- 작업한 결과를 gitHub에 commit합니다.

출력화면 예시 `http://127.0.0.1:8000/movies/`로 접속하면 다음과 같은 화면이 보여야 합니다.

#### 🎬 오늘의 영화 추천
---
###### ▫️ 인터스텔라

장르: SF, 드라마  
우주를 배경으로 한 아버지의 위대한 사랑과 모험 이야기.  
🔵 [추천하기]

---
###### ▫️ 기생충

장르: 드라마, 스릴러  
상류층과 하류층의 현실을 꼬집는 블랙코미디 영화.  
🔵 [추천하기]

---
###### ▫️ 토이 스토리

장르: 애니메이션, 가족  
장난감들의 유쾌한 우정과 모험 이야기.  
🔵 [추천하기]


`html 및 CSS 제공`
```html
<html>
    <head>
        <title>🎬 영화 추천</title>
        <style>
            body { font-family: Arial, sans-serif; padding: 2em; }
            .movie { border: 1px solid #ccc; padding: 1em; margin-bottom: 1em; border-radius: 10px; }
            .title { font-size: 1.5em; font-weight: bold; }
            .genre { color: gray; }
            .description { margin-top: 0.5em; }
            .btn { margin-top: 1em; display: inline-block; padding: 0.5em 1em; background: #007bff; color: white; text-decoration: none; border-radius: 5px; }
        </style>
    </head>
    <body>
        <h1>🎬 오늘의 영화 추천</h1>

        <div class="movie">
            <div class="title">인터스텔라</div>
            <div class="genre">장르: SF, 드라마</div>
            <div class="description">우주를 배경으로 한 아버지의 위대한 사랑과 모험 이야기.</div>
            <a href="#" class="btn">추천하기</a>
        </div>

        <div class="movie">
            <div class="title">기생충</div>
            <div class="genre">장르: 드라마, 스릴러</div>
            <div class="description">상류층과 하류층의 현실을 꼬집는 블랙코미디 영화.</div>
            <a href="#" class="btn">추천하기</a>
        </div>

        <div class="movie">
            <div class="title">토이 스토리</div>
            <div class="genre">장르: 애니메이션, 가족</div>
            <div class="description">장난감들의 유쾌한 우정과 모험 이야기.</div>
            <a href="#" class="btn">추천하기</a>
        </div>
    </body>
    </html>
```