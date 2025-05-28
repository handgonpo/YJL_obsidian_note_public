🔹 Django 튜토리얼 Part 1- Django App 만들기 (설문조사앱)
###### 📖 공식 문서 링크:  
🔗  https://docs.djangoproject.com/ko/5.2/intro/tutorial01/

목표
- 사람들이 투표할 수 있는 공개 사이트
- 관리자가 설문을 추가/수정/삭제할 수 있는 관리용 사이트

✅ 1단계: Django 설치 확인
```bash
python -m django --version
```

✅ 2단계: 프로젝트를 위한 폴더 만들기
```bash
mkdir djangotutorial
cd djangotutorial
```
---
✅ 3단계: 앱 생성
```bash
django-admin startproject mysite .
```

◽ polls 앱 생성후 전체구조는 다음과 같습니다:
```makdown
djangotutorial/
├── manage.py
└── mysite/
    ├── __init__.py
    ├── settings.py
    ├── urls.py
    ├── asgi.py
    └── wsgi.py
```
- `manage.py`: Django 프로젝트를 관리하는 명령어 도구
- `settings.py`: 환경 설정
- `urls.py`: URL 연결(라우팅) 설정
- `asgi.py` / `wsgi.py`: 서버용 진입점

✅ 4단계: 개발 서버 실행 및 서버 종료
```python
python manage.py runserver

Ctrl + C
```

✅ 5단계: 앱(App) 만들기
이제 실제 설문조사 기능을 담을 앱(app) 을 생성합니다.
```python
python manage.py startapp polls
```

`polls/` 폴더가 생기고 구조는 다음과 같습니다:
```
polls/
├── __init__.py
├── admin.py
├── apps.py
├── migrations/
│   └── __init__.py
├── models.py
├── tests.py
├── views.py
```

✅ 6단계:  라우터 

`mysite/urls.py`
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),  # polls 앱의 URL 연결
    path("admin/", admin.site.urls),
]
```

`Admin(관리자)`
```python
from django.contrib import admin
```
- `django.contrib`는 장고가 기본으로 제공하는 내장 앱 모음 패키지입니다.
- 그 안의 `admin` 모듈은 관리자(admin) 웹사이트를 생성하고 관리할 수 있게 해주는 기능을 포함합니다.
- 자동 생성된 admin 사이트를 통해 데이터베이스의 모델 데이터를 시각적으로 편리하게 추가/편집/삭제할 수 있습니다.
---
`polls/urls.py` 파일 새로 만들기
```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```

🔹 `path( )`와 `include()`:

`모듈 불러오기`
```python
from django.urls import path, include
```

◽ `path()` 함수란?
- 특정 URL 경로(URL Pattern) 가 들어왔을 때
- 어떤 View 함수(또는 클래스) 를 실행할지 직접 연결하는 함수입니다.

📖 문법, 구문(syntax): 
```python
path('URL패턴', 실행할뷰함수, name='이름')  
```

◽ URL에 이름 부여 (`name`)
	URL 경로에 이름을 부여하여, 나중에 템플릿이나 코드에서 하드코딩 없이링크 참조가 가능합니다.

💬  하드코딩이란?
	고정된 값(데이터, 설정 등)을 코드 내부에 직접 써넣는 것을 의미합니다.  
	그래서 나중에 바꾸려면 코드를 수정해야만 합니다.
```python
path('', views.index, name='index')
```

템플릿 예시:
```python
<a href="{% url 'index' %}">홈으로</a>
```
---
◽ `include()` 함수란?
- 다른 앱의 `urls.py`를 현재 프로젝트의 URLconf에 포함(include) 시킬 때 사용합니다.
- 앱 단위로 URL을 관리할 수 있게 해주는 함수입니다.
```python
path("polls/", include("polls.urls")), # polls 앱의 urls.py를 연결
```

💬 한 줄 요약
- `path()` → URL 경로와 view 함수를 연결
- `include()` → 앱 단위로 만든 URL 파일(앱의 urls.py) 을 불러와 연결

즉, include는 “앱 통째로 붙이기”,  
path는 “URL 한 줄 붙이기”라고 이해하면 됩니다.

---
✅ 7단계: 첫 번째 뷰(view) 만들기

📁 `polls/views.py` 열고 아래 코드 작성:
```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

◽ 뷰 함수 (views.py) `polls/views.py`
- URL 요청이 들어오면 실행되는 비즈니스 로직을 담당합니다.
- HTML 출력, DB 조회, 템플릿 렌더링 등 다양한 역할을 합니다

💬 비즈니스 로직이란?
	사용자의 요청을 실제 서비스나 도메인 규칙에 따라 처리하는 핵심 규칙을 말합니다.
	
예를 들어, 쇼핑몰 시스템이라면 다음과 같은 것들이 비즈니스 로직입니다:
- 장바구니에 담긴 상품의 총 금액 계산
- 재고가 없을 경우 주문 불가 처리
- 특정 회원 등급에 따른 할인율 적용
- 결제 완료 시 포인트 적립

◽ 확장된 HTML + CSS 포함 예시 (브라우저에 꾸며서 출력)

📁 `polls/views.py` (연습용)
```python
from django.http import HttpResponse

def index(request):
    html = """
    <html>
    <head>
        <title>투표 사이트</title>
        <style>
            body {
                font-family: Arial, sans-serif;
                background-color: #f2f2f2;
                padding: 2em;
            }
            h1 {color: #333399;}
            .box {
                background: white;
                padding: 1.5em;
                border-radius: 10px;
                box-shadow: 0 0 10px rgba(0,0,0,0.1);
                max-width: 600px;
                margin: auto;
            }
        </style>
    </head>
    <body>
        <div class="box">
            <h1>안녕하세요!</h1>
            <p>여기는 <strong>설문조사(polls)</strong> 
            사이트의 메인 페이지입니다.
            </p>
            <p>관리자는 설문을 추가하고, 사용자는 투표할 수 있습니다.</p>
        </div>
    </body>
    </html>
    """
    return HttpResponse(html)
```

---
🔹 Django 프로젝트 흐름도
![[Group 355.png]]

🔹 이미지 흐름도 요약 설명
	이미지에는 사용자의 요청이 프로젝트의 `urls.py` → 앱의 `urls.py` → 앱의 `views.py` → `models.py`와 DB로 이어지는 흐름이 그려져 있습니다.

◽ 1. 클라이언트 요청 (웹 브라우저에서 URL 요청이 들어옵니다.)
```python
예: http://localhost:8000/polls/
```

◽ 2. 프로젝트 루트의 `urls.py` (`mysite/urls.py`)
	요청 URL을 앱(polls)의 URL로 연결합니다.
```python
from django.contrib import admin
from django.urls import include, path

urlpatterns = [
    path("polls/", include("polls.urls")),  # polls 앱의 URL 연결
    path("admin/", admin.site.urls),
]
```
- 역할: 전체 프로젝트의 URL 라우팅 담당, 앱별로 연결을 분배

◽ 3. 앱의 `urls.py` (`polls/urls.py`)
- 각 앱에서 요청 URL에 따라 적절한 view 함수를 호출합니다.
```python
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
]
```
- 역할: 앱 내부에서 어떤 함수를 실행할지 결정

◽ 4. 앱의 `polls/views.py`
	URL로 연결된 요청을 처리하고, 데이터를 가져오거나 템플릿에 넘겨줍니다.
```python
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
```

◽ 5. 앱의 `polls/models.py`
	데이터베이스와 연결되는 데이터 구조(테이블) 정의
```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')
```

◽ 6. 데이터베이스 (DB)
	`models.py`를 통해 정의된 구조가 마이그레이션 되어 실제 데이터가 저장됩니다. 예: SQLite(`db.sqlite3`)

🔚 마무리 구조 요약:
```python
사용자 요청 → mysite/urls.py → polls/urls.py → views.py → models.py → DB
```

이러한 방식으로 Django는 MTV(Model-Template-View) 아키텍처를 기반으로 모든 흐름을 관리하며, 각각의 파일은 이 흐름 속에서 고유한 역할을 수행합니다.

---
✅ 8단계: 앱을 프로젝트에 등록 (앱을 생성하면 자동생성됨)

`polls/apps.py`
```python
from django.apps import AppConfig

class PollsConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'polls'
```
`name = 'polls'` : 이 속성은 앱의 이름을 Django에게 알려주는 용도임.
`default_auto_field = 'django.db.models.BigAutoField'` :
Django가 모델의 기본 자동 필드(primary key)로 어떤 타입을 사용할지를 지정하기 위한 설정입니다.

`mysite/settings.py`의 `INSTALLED_APPS`에 `'polls.apps.PollsConfig'` 추가:
```python
INSTALLED_APPS = [
    'polls.apps.PollsConfig',  # 이 줄 추가
]
```

두 코드를 꼭 같이 써야 하는것은 아니지만 apps.py 사용이 권장됩니다.
최소 설정만 할 경우:
```python
# settings.py
INSTALLED_APPS = [
    'polls',  # 가능함
]
```
- 이렇게 `'polls'`만 써도 Django는 자동으로 `polls` 앱을 인식해서 잘 작동합니다.
- Django는 내부적으로 `polls/apps.py`가 있는지도 찾아보고, 없으면 그냥 `polls` 디렉토리를 기준으로 앱을 등록하며, apps.py 없이도 동작합니다.

❓ 왜 `apps.py`를 쓰는 이유는 `apps.py`를 사용하면 앱의 추가 설정을 커스터마이즈할 수 있기 때문입니다.

`ready()` 메서드 예시
```python
class PollsConfig(AppConfig):
    name = 'polls'

    def ready(self):
        import polls.signals # 앱 시작 시 실행할 초기 설정
```

---
✅ 9단계: 개발 서버 실행 테스트 
```bash
python manage.py runserver
```
[http://127.0.0.1:8000/](http://127.0.0.1:8000/) 에 접속하여 Django 환영 페이지가 잘 나오는지 확인

---
✅ 10단계: 결과 확인
```markdown
http://127.0.0.1:8000/polls/
```
	출력 결과:  
	Hello, world. You're at the polls index.

==공식문서에 있는 part1 완료==

---
###### 🔹 첫 Commit

1️⃣ .gitignore파일을 만든다.
```.gitignore
# Python 기본 캐시 및 가상환경 파일 제외
venv/
env/
__pycache__/
*.pyc
*.pyo
*.pyd

# 환경 변수 파일 (보안 중요)
.env
# 개발중에만 사용

# 데이터베이스 파일 제외 (SQLite 등)
db.sqlite3
*.sqlite3

# Django 마이그레이션 캐시 제외
**/migrations/*.pyc
**/migrations/*.py
!**/migrations/__init__.py

# 로그 파일 제외
*.log
*.out
*.err

# 미디어 및 정적 파일 (수동 업로드 방지)
# media/
# staticfiles/
# static/
node_modules/

# VS Code 및 IDE 설정 파일 제외
.vscode/
.idea/
*.sublime-workspace

# Docker 관련 파일 제외 (사용할 경우)
docker-compose.override.yml
```

◽  .gitignore파일 경로를 반드시 확인:
![[Pasted image 20250510104827.png|300]]

---
###### 🔹 requirements.txt 파일 생성

1️⃣  가상환경을 활성화한 상태에서 실행:
```bash
source venv/bin/activate
venv\Scripts\activate
```

2️⃣ requirements 명령어 실행:
```bash
pip freeze > requirements.txt
```

3️⃣ requirements.txt 파일 생성 확인
```python
asgiref==3.8.1
Django==5.2.1
sqlparse==0.5.3
Python==3.12.3
```

---
###### 🔹 Git Commit Push & Pull

◽ Git 첫 Commit : CLI 명령어 방식
```bash
# Git 초기화
git init

# 브랜치 이름을 바로 Eunice로 지정
git checkout -b Eunice

# 커밋할 임시 파일 만들기
touch README.md

# Git에 추가하고 커밋
git add README.md
git commit -m "Initial dummy commit for Eunice branch"

# 원격 저장소 연결
git remote add origin https://github.com/handgonpo/Django_first.git

# 푸시
git push -u origin Eunice
```

`git init` :	
	로컬 프로젝트 디렉토리를 Git 저장소로 초기화
`.gitignore 설정` :	
	- 추적하지 않을 파일 지정 (예: .env, `__pycache__`/, db.sqlite3 등)
	- 보안에 민감한 API키(.env) 
	- 개인개발환경에서만 생성되는 임시파일 `__pycache__/`나 `.DS_Store`
	- 용량절약: `db.sqlite3` DB는 자주 바뀌고 용량이커서 버전 관리에 부적
	- 협업방지: 개발자마다 개발환경이 다르므로 로컬설정을 고유하면 오류
`git add .`	: 
	모든 변경 사항을 Git 스테이징 영역에 추가
`git commit -m "처음 커밋"`	: 커밋 메시지는 반드시 입력해야 커밋이 됩니다.
	커밋메시지는 변경사항의 의미를 적어서 기록에 남긴다.
`git remote add origin 깃헙주소 :	
	GitHub 원격 저장소 연결 (한 번만 설정하면 됨)
`git branch -M main` : 
	브랜치를 main으로 명명 (기본 브랜치 명 통일용)
```bash
git checkout -b Eunice        # 새 브랜치 생성
git push origin Eunice        # GitHub에 푸시
```

![[Pasted image 20250527080642.png]]
![[Pasted image 20250527080710.png]]
브런치를 자신의 이니셜로 바꾸면 협업, 분류, 관리 측면에서 매우 실용적입니다

`git push -u origin main` :	
	원격 저장소에 처음으로 푸시 (앞으로는 git push만 해도 됨)

초기 설정 이후 커밋하는 방법: 터미널에서 CLI로 입력하는 방법과 VSCode 이용
◾ Git  Commit :
```bash
# 전체 파일 add 
git add .

# 커밋 메시지 작성
git commit -m "전달할 메시지"
```

![[Pasted image 20250527081116.png]]

◾ GitHub에서 수정한 내용을 VSCode로 가져오는 방법:
```bash
# 현재 브랜치 확인
git branch

# 현재 브랜치가 'Eunice' 라면:
git pull origin Eunice
```

◾ VSCode에서 Commit & Push
![[Pasted image 20250510121743.png]]


