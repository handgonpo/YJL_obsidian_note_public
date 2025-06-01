🔹 Django 튜토리얼 Part 3 – View와 Template 만들기
###### 📖 공식 문서 링크:  
🔗 [https://docs.djangoproject.com/ko/stable/intro/tutorial03/](https://docs.djangoproject.com/ko/stable/intro/tutorial03/)

목표
- `view()` 함수 작성 (FBV)
- URL과 view 연결 (`urls.py`)
- 템플릿 사용 및 렌더링 (`render`)
- 404 예외 처리 (`get_object_or_404`)
- 템플릿의 하드코딩 URL 제거 (`{% url %}`)
- URL 네임스페이스 지정 (`app_name`)

```
FBV (Function-Based View) : 함수로 뷰를 정의하는 방식
CBV (Class-Based View) : 클래스로 뷰를 정의하는 방식
```
----
✨ 전체코드 실습부터 하기 (CRUD에서 Read에 해당합니다.) `polls/views.py`

✅ `index` 전체 질문중 5개만 조회 (FBA 기반 View)
```python
from django.shortcuts import render, get_object_or_404
from .models import Question, Choice

# index(최신글 list)
def index(request):
	# return HttpResponse("Hello) 기존코드
	
	latest_question_list = Question.objects.order_by("-pub_date")[:5]
	context = {"latest_question_list": latest_question_list}
	return render(request, "polls/index.html", context)
```

✅ `detail` : 상세조회
```python
def detail(request, question_id):
	question = get_object_or_404(Question, pk=question_id)
	return render(request, "polls/detail.html", {"question": question})
```

✅ `results` : 결과보기
```python
def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})
```
`/polls/1/results/`  
`"You're looking at the results of question 1."` 같은 더미 텍스트 출력

✅ `vote` : 투표하기 
```python
def vote(request, question_id):
    return HttpResponse(f"You're voting on question {question_id}.")
```
`/polls/1/vote/`  
`"You're voting on question 1."` 같은 더미 텍스트 출력

---
✅ `polls/urls.py`
```python
from django.urls import path
from . import views

app_name = "polls"  # 네임스페이스 지정
  
urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
---
📁 html을 작성하기 위해 템플릿 폴더 및 html파일 추가 구조:
```
mysite/                     ← 최상위 프로젝트 디렉토리
├── manage.py               ← Django 명령어 실행 파일
├── mysite/                 ← 프로젝트 설정 디렉토리
│   ├── __init__.py
│   ├── settings.py         ← ==환경 설정 파일==
│   ├── urls.py             ← ==전체 URL 연결 파일==
│   ├── asgi.py
│   └── wsgi.py
├── polls/                  ← polls 앱 디렉토리
│   ├── __init__.py
│   ├── admin.py            ← 관리자 설정
│   ├── apps.py             ← 앱 설정 정보
│   ├── migrations/         ← 마이그레이션 파일 저장 폴더
│   │   └── __init__.py
│   ├── models.py           ← 데이터베이스 모델 정의
│   ├── tests.py            ← 테스트 코드
│   ├── views.py            ← ==뷰(View) 함수 정의==
│   ├── urls.py             ← ==polls 전용 URL 설정==
│   └── templates/          ← 템플릿 폴더 (HTML 보관)
│       └── polls/    ← 앱 이름과 동일한 하위 폴더 (템플릿 네임스페이스)
│           ├── index.html  ← ==설문 목록 페이지==
│           └── detail.html ← ==설문 상세 페이지==
```
---
✅ `mysite/settings.py` 
```python
import os

TEMPLATES = [
    {
        'DIRS': [os.path.join(BASE_DIR, 'polls', 'templates')],
    },
]
```
	Django가 템플릿 파일을 어디에서 찾을지 알려주는 설정입니다.

---
✅ `polls/templates/polls/index.html`
```html
<h1>최근 질문</h1>
{% if latest_question_list %}
    <ul>
    {% for question in latest_question_list %}
        <li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
    {% endfor %}
    </ul>
{% else %}
    <p>No polls are available.</p>
{% endif %}
```

![[Pasted image 20250528103419.png]]

---
✅ `polls/templates/polls/detail.html`
```html
<h1>{{ question.question_text }}</h1>
<ul>
  {% for choice in question.choice_set.all %}
    <li>{{ choice.choice_text }}</li>
  {% endfor %}
</ul>
```

![[Pasted image 20250528103508.png]]

---
`개발 서버 실행 및 및 결과 확인:`
```python
python manage.py runserver
```
---
◽  CSS 꾸미기와 확장형 수정이 용이하게 분기하기

📁 폴더 구조 : `html과 css 분리하기`
```
polls/
├── static/
│   └── polls/
│       ├── index.css
│       └── detail.css
├── templates/
│   └── polls/
│       ├── base.html
│       ├── header.html
│       ├── footer.html
│       ├── index.html
│       └── detail.html
```

✅ `templates/polls/index.html`
```html
{% extends "polls/base.html" %}
{% block title %}최근 질문{% endblock %}

{% block content %}
<h1>최근 질문</h1>
{% if latest_question_list %}
<ul>
  {% for question in latest_question_list %}
    <li>
      <a href="{% url 'polls:detail' question.id %}">
        {{ question.question_text }}
      </a>
    </li>
  {% endfor %}
</ul>
{% else %}
<p>No polls are available.</p>
{% endif %}
{% endblock %}
```
![[Pasted image 20250531175237.png]]

✅`polls/templates/polls/detail.html`
```html
{% extends "polls/base.html" %}
{% load static %}

{% block title %}{{ question.question_text }}{% endblock %}

{% block content %}
<link rel="stylesheet" href="{% static 'polls/detail.css' %}">

<div class="detail-container">
  <h2 class="question-title">{{ question.question_text }}</h2>
  <ul class="choice-list">
    {% for choice in question.choice_set.all %}
      <li>{{ choice.choice_text }}</li>
    {% endfor %}
  </ul>
</div>
{% endblock %}
```
![[Pasted image 20250531175756.png]]

✅`templates/polls/header.html`
```html
{% load static %}
<header>
  <div class="site-header">
    <h1 class="site-title">설문조사 시스템</h1>
    <nav class="site-nav">
      <ul>
        <li><a href="{% url 'polls:index' %}">HOME</a></li>
      </ul>
    </nav>
  </div>
</header>
```

✅`templates/polls/footer.html`
```html
{% load static %}
<footer>
  <div class="site-footer">
    <p>&copy; 2025 Django Polls. All rights reserved.</p>
    <p>
      <a href="https://github.com/yourusername">My GitHub |</a>
      <a href="/terms">Terms of Service</a>
    </p>
  </div>
</footer>
```
	저작권 문구나 링크를 여기 담아 두면, 프로젝트 전체 페이지에서 동일한
	footer가 사용됩니다.

✅`templates/polls/base.html` (템플릿 상속용 베이스 템플릿)
```html
{% load static %}
<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <title>{% block title %}Polls{% endblock %}</title>
    <link rel="stylesheet" href="{% static 'polls/styles.css' %}">
</head>
<body>
    {% include "polls/header.html" %}

    <main>
        {% block content %}
        <!-- 자식 템플릿에서 이 부분이 채워집니다 -->
        {% endblock %}
    </main>

    {% include "polls/footer.html" %}

</body>
</html>
```

✅`polls/index.css`
```css
body {
    background-color: #f5f7fa;
    font-family: 'Segoe UI', sans-serif;
    margin: 0;
    padding: 0;
}

header {
    background-color: #4a4e69;
    color: white;
    padding: 20px;
    justify-content: space-between;
    align-items: center;
    position: relative;
} 

main {
    padding: 20px;
    text-align: center;
}

h2 {
    color: #333;
    font-size: 28px;
    margin-bottom: 20px;
}

/* 질문 리스트 중앙 정렬 */
ul {
    list-style: none;
    padding: 0;
    margin: 0 auto;
    max-width: 700px;
}

li {
    background-color: white;
    border-radius: 10px;
    margin-bottom: 12px;
    padding: 15px 20px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
    transition: background-color 0.2s ease;
}

li:hover {
    background-color: #f0f4f8;
}

a {
    text-decoration: none;
    font-size: 18px;
    color: #1d3557;
}  

footer {
    margin-top: 50px;
    padding: 20px;
    background-color: #f1f1f1;
    text-align: center;
    font-size: 14px;
    color: #666;
}
 
footer a {
    color: #1e1c2b;
    text-decoration: none;
    font-size: 14px;
}

/*우선순위를 위해서*/
.site-header {
    width: 100%;
    position: relative;
    text-align: center;
} 

.site-title {
    margin: 0;
    font-size: 24px;
}

.site-nav {
    position: absolute;
    left: 100px;
    top: 50%;
    transform: translateY(-50%);
}

.site-nav ul {
    list-style: none;
    margin: 0;
    padding: 0;
}

.site-nav li {
    display: inline;
    background: none;
    box-shadow: none;
}

.site-nav li:hover {
    background: none;
}

.site-nav li:hover a {
    color: #b8bee6;
}

.site-nav a {
    color: white;
    text-decoration: none;
    font-weight: bold;
    font-size: 16px;
}

```

✅`polls/detail.css`
```css
.detail-container {
  background-color: #ffffff;
  max-width: 600px;
  margin: 3rem auto;
  padding: 3rem;
  border-radius: 15px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  text-align: center;
}

.question-title {
  font-size: 26px;
  color: #1d3557;
  margin-bottom: 20px;
}

.choice-list {
  list-style: none;
  padding: 0;
  margin: 0 auto 20px;
  text-align: left;
  max-width: 400px;
}

.choice-list li {
  margin-bottom: 12px;
  background-color: #f8f9fa;
  padding: 10px 15px;
  border-radius: 8px;
  border: 1px solid #ddd;
  transition: background-color 0.2s ease;
}

.choice-list li:hover {
  background-color: #e2eafc;
}
```

✅`mysite/settings.py`
```python
STATIC_URL = 'static/'
```

✅`서버 실행 후 결과` : `http://127.0.0.1:8000/polls/`
```python
python manage.py runserver
```

---
✨ 코드리뷰

1️⃣ `views.py`에 뷰 함수 만들기 `polls/views.py`
함수 기반 뷰 Function-Based View (FBV)
```python
from django.shortcuts import render, get_object_or_404
from .models import Question

# index(최신글 list)
def index(request):	
	latest_question_list = Question.objects.order_by("-pub_date")[:5]
	context = {"latest_question_list": latest_question_list}
	return render(request, "polls/index.html", context)
		
# detail(상세조회)		
def detail(request, question_id):
	question = get_object_or_404(Question, pk=question_id)
	return render(request, "polls/detail.html", {"question": question})

def results(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/results.html", {"question": question})

def vote(request, question_id):
    return HttpResponse(f"You're voting on question {question_id}.")
```
`Question.objects`	
	Question 모델의 모든 객체를 가져오는 쿼리셋(QuerySet)
`.order_by("-pub_date")`	
	pub_date 필드를 기준으로 내림차순 정렬
	(-가 붙으면 내림차순, 안 붙으면 오름차순)
`[:5]`
	상위 5개만 슬라이싱 (Python 리스트 슬라이싱처럼 작동)

- **`order_by()` → `[:5]`** 순서는 Django ORM에서 정해진 규칙입니다.
- 바뀌면 동작하지 않거나, 의미가 달라지거나, 오류가 납니다.
---
💡 메서드 체이닝 순서 팁:
- `objects`	: 쿼리셋 시작
- `filter(), exclude()` : 조건 필터링
- `order_by()` : 정렬
- `[:n]` : 슬라이싱 (결과 개수 제한)
- `values(), annotate()` 등: 	필요 시
---
`context = {"latest_question_list": latest_question_list}`	
	`context` 딕셔너리는 템플릿(`index.html`)에 전달할 데이터 묶음입니다.
	템플릿 안에서는 이 데이터의 변수명을 사용해서 화면에 표시할 수 있어요.
```python
context = {"latest_question_list": latest_question_list}
```
- 여기서 `latest_question_list`는 DB에서 가져온 최신 질문 5개입니다.
- 이걸 템플릿에 `latest_question_list`라는 이름으로 넘깁니다.

context를 딕셔너리 형태로 넘기는 이유는
```python
context = {
    "템플릿에서 사용할 이름(문자열)": 실제 변수명(파이썬 변수)
}
```
`"latest_question_list"`  : 템플릿에서 사용할 이름 → 문자열 키
`latest_question_list`  : 실제 Python 데이터 → 값(Question 객체 리스트)

비유하면  딕셔너리 키 "name"    딕셔너리 값 "홍길동"
즉, 템플릿에 넘겨줄 제목:내용 또는 이름:값 구조입니다.

템플릿에서 실제로 출력되는 위치
```python
<h1>최근 질문</h1>
<ul>
  {% for question in latest_question_list %}
    <li>{{ question.question_text }}</li>
  {% endfor %}
</ul>
```
- `latest_question_list`는 context에서 전달된 이름 그대로 사용됩니다.
- 이 리스트를 템플릿에서 `for`문으로 하나씩 꺼내서 `question.question_text`를 출력합니다.

화면에 출력하면 다음과 같이 출력됩니다.
DB에 아래와 같은 질문들이 있다면:
![[Pasted image 20250527155242.png]]

템플릿을 통해 브라우저에 보이는 화면은 이렇게 됩니다:
```python
<h1>최근 질문</h1>
<ul>
  <li>What’s new?</li>
  <li>How are you?</li>
  <li>What's up?</li>
</ul>
```

F12 개발자 도구 (Developer Tools)로 직접 넘어온 데이터를 확인할수 있습니다.

---
`detail` 
```python
def detail(request, question_id):
	question = get_object_or_404(Question, pk=question_id)
	return render(request, "polls/detail.html", {"question": question})
```

📖 문법, 구문(syntax): 
```python
render(request, template_name, context=None, content_type=None, status=None, using=None)
```
`request`	: 필수. 현재 요청 객체 (HttpRequest)
`template_name`	: 필수. 렌더링할 템플릿 파일 경로 (문자열)
`context`	: 선택. 템플릿에서 사용할 데이터 딕셔너리
`content_type` : 선택. 반환할 콘텐츠 유형 (기본값: text/html)
`status` : 선택. HTTP 응답 코드 (예: 200, 404 등)
`using`	: 선택. 템플릿 엔진 이름 (여러 엔진이 있는 경우 지정할 때 사용)

위의 코드는 `template_name`과 `status=변수` 이 상태이며 그외는 모두 기본값으로 치환됩니다.

---
◽ 404 처리 (질문이 없을 때 예외 발생)

방식 1: try-except (기본)
```python
# polls/views.py
from django.http import Http404
from django.shortcuts import render
from .models import Question



def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        raise Http404("Question does not exist")
    return render(request, "polls/detail.html", {"question": question})
```

방식 2: 단축 함수 (`get_object_or_404`)
```python
# 단축 함수 사용
from django.shortcuts import get_object_or_404

def detail(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    return render(request, "polls/detail.html", {"question": question})
```
---
`results()` 함수
```python
def results(request, question_id):
    return HttpResponse(f"You're looking at the results of question {question_id}.")
```
- 질문의 결과 페이지를 처리하는 뷰입니다.
- 예: `/polls/2/results/` 접근 시, 
	`"You're looking at the results of question 2."` 같은 형태로 출력

`vote()` 함수
```python
def vote(request, question_id):
    return HttpResponse(f"You're voting on question {question_id}.")
```
- 해당 질문에 대한 투표를 처리하는 뷰입니다.
- 예: `/polls/5/vote/` 접근시,
	`"You're voting on question 5."` 같은 형태로 출력

---
2️⃣ 🔗 `urls.py`에 경로 연결하기
```python
# polls/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path("", views.index, name="index"),
    path("<int:question_id>/", views.detail, name="detail"),
    path("<int:question_id>/results/", views.results, name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
`<int:question_id>` : 
	이건 Django의 URL 경로 패턴 문법입니다. 
	HTML 주소(URL)에서 숫자 값(정수)을 추출해서 변수로 저장하고,  
	그 값을 뷰 함수에 인자로 넘겨주는 역할을 합니다.

</> 예시코드:
```python
path("<int:question_id>/", views.detail, name="detail")
```
이렇게 작성하면,
- 사용자가 `/polls/5/`로 접속하면
- Django는 `question_id = 5`로 자동으로 꺼내서
- `views.detail(request, question_id=5)` 이렇게 호출합니다.

구성요소설명:
![[Group 368.png]]
`< >` :	
	URL 경로에서 값을 추출하겠다는 표시
`int:` :
	추출되는 값은 정수형이어야 함. (예: 3, 12, 99)
`question_id` :
	이 이름으로 값을 뷰 함수에 넘김 (함수의 매개변수 이름과 일치해야 함)
	
---
3️⃣ `polls/templates/polls/index.html`
```html
{% extends "polls/base.html" %}
  {% block title %}최근 질문{% endblock %}

  {% block content %}
  <h1>최근 질문</h1>
  {% if latest_question_list %}
  <ul>
    {% for question in latest_question_list %}
      <li>
        <a href="{% url 'polls:detail' question.id %}">
          {{ question.question_text }}
        </a>
      </li>
    {% endfor %}
  </ul>
  {% else %}
  <p>No polls are available.</p>
  {% endif %}
{% endblock %}
```

###### 템플릿 태그 문법
| 문법         | 역할                 | 예시                                                   |
| ---------- | ------------------ | ---------------------------------------------------- |
| `{{ }}` .. | 데이터를 출력 <br>(변수)   | `{{ question.question_text }}` <br>실제 질문 내용이 화면에 출력됨 |
| `{% %}`    | 로직(제어문, 템플릿 태그) 실행 | `{% for question in latest_question_list %}`         |
| `{# #}`    | 주석 (브라우저에 안보임)     | `{# 이것은 주석입니다. #}`                                   |

```
{% extends "base.html" %} ← 이 템플릿은 base.html을 기반으로 한다

{% block content %} ← 여기부터 base.html의 content 영역에 들어갈 내용
    {% if something %} ← 조건이 참이면
        <p>무언가 있음</p>
    {% else %} ← 조건이 거짓이면
        <p>없음</p>
    {% endif %}  ← 조건문 끝
{% endblock %} ← block 종료
```

❌ 템플릿에서 하드코딩된 URL 제거 (하드코딩)
```html
<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
```

수정 방식 (템플릿 태그 사용)
```html
<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
```
템플릿에서 `path(..., name="detail")`에 정의된 이름을 활용


  



