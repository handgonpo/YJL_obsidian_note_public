🔹 Django 튜토리얼 Part8 – 서드파티 패키지 사용하기

###### 📖 공식 문서 링크:  
[https://docs.djangoproject.com/ko/5.2/intro/tutorial08/](https://docs.djangoproject.com/ko/5.2/intro/tutorial08/)
[Django Debug Toolbar](https://django-debug-toolbar.readthedocs.io/en/latest/)

목표
- Django 프로젝트에 서드파티 패키지(특히 `django-debug-toolbar`)를 설치하고 사용하는 법을 배우는 것

`가상환경이 활성화된 상태에서 아래 명령어를 실행하세요:`
```bash
pip install django-debug-toolbar
```
---
`settings.py` `INSTALLED_APPS`에 추가
```python
INSTALLED_APPS = [
    # 기존 항목들
    'django.contrib.staticfiles',  # 꼭 있어야 함
    'debug_toolbar',               # 추가
]
```
`'django.contrib.staticfiles'`는  CSS, JavaScript, 이미지 같은 정적 자원`(static assets)`을 관리하고 서빙할 수 있도록 해주는 Django의 기본 내장 앱입니다.

---
`'debug_toolbar.middleware.DebugToolbarMiddleware'`를 가장 앞쪽에 추가하는 것이 일반적입니다:
```python
MIDDLEWARE = [
    'debug_toolbar.middleware.DebugToolbarMiddleware',  # 추가
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    # 나머지 미들웨어들...
]
```
---
내부 IP 설정
디버그 툴바는 보안을 위해 `INTERNAL_IPS`에서 허용된 IP에서만 동작합니다. 개발 시에는 로컬호스트만 허용하면 됩니다.
```python
INTERNAL_IPS = [
    "127.0.0.1",
]
```
---
`urls.py` 설정
프로젝트의 최상위 `urls.py`에서 `debug_toolbar` 경로 추가
```python
from django.conf import settings
from django.conf.urls.static import static
from django.urls import include, path

urlpatterns = [
    path('admin/', admin.site.urls),
    # 다른 URL 패턴들...
]

if settings.DEBUG:
    import debug_toolbar
    urlpatterns += [
        path('__debug__/', include(debug_toolbar.urls)),
    ]
```

`if settings.DEBUG:`
- 이 조건은 `settings.py`의 `DEBUG = True`일 때만 안쪽 코드를 실행합니다.
- 즉, 개발 환경에서만 디버그 툴바를 활성화하려는 목적입니다.
- 운영환경(production)에서는 `DEBUG = False`이므로 디버그 툴바가 로드되지 않습니다.

Django 프로젝트의 `settings.py` 파일에 다음과 같은 코드가 있습니다:
```python
DEBUG = True
```
이 값이 `True`이면 개발 모드, `False`이면 운영 모드(배포 환경)로 간주됩니다. 배포시에는 반드시 `False`로 운영모드를 변경해야 합니다.

`import debug_toolbar`
- `django-debug-toolbar` 패키지를 가져옵니다.
- 이 패키지는 웹 페이지의 사이드에 디버깅 정보를 보여주는 툴바를 띄워줍니다.

 `urlpatterns += [...]`
- `urlpatterns`는 Django가 URL 요청을 어떤 뷰로 처리할지 결정하는 URL 패턴 목록입니다.
- `+=`는 기존의 URL 패턴 리스트에 디버그 툴바 관련 경로를 추가하는 것을 의미합니다.

`path('__debug__/', include(debug_toolbar.urls))`
- 디버그 툴바가 사용하는 URL들을 `/__debug__/` 경로 아래에 추가합니다.
- 예: `http://localhost:8000/__debug__/` 아래에 툴바 리소스 요청들이 존재합니다.
- 실제로는 브라우저에 표시되는 디버그 툴바도 이 URL로부터 JS/CSS를 불러옵니다.

`서버 실행 후 확인`
```bash
python manage.py runserver
```

---
###### ◽  주요 기능 설명
| 항목명 (한글)  | 원래 명칭               | 설명                                            |
| --------- | ------------------- | --------------------------------------------- |
| 히스토리      | History             | 요청 URL과 관련된 히스토리 추적 정보                        |
| 버전        | Versions            | Django 및 설치된 패키지의 버전 정보                       |
| 시각        | Timer               | 요청 처리에 걸린 시간 (CPU, 전체 시간 포함)                  |
| 설정        | Settings            | Django의 설정(`settings.py`) 내용                  |
| 헤더        | Headers             | 요청 및 응답 HTTP 헤더 정보                            |
| 요청        | Request             | 요청된 URL 패턴, View 함수, 이름 등                     |
| SQL       | SQL                 | 해당 요청 중 실행된 SQL 쿼리와 시간 측정                     |
| 정적 파일     | Static files        | 로딩된 정적 파일 목록                                  |
| 템플릿       | Templates           | 렌더링된 템플릿 목록 및 context                         |
| Alerts    | Alerts              | 경고나 문제 감지 여부 표시                               |
| 캐시        | Cache               | 사용된 캐시 백엔드, 히트/미스 여부                          |
| 시그널       | Signals             | Django signals 사용 내역 (예: pre_save, post_save) |
| 리디렉션 가로채기 | Intercept redirects | 3xx 응답을 가로채 디버깅 가능                            |
| 프로파일링     | Profiling           | 함수 실행 시간 측정 (Line 프로파일러 필요)                   |

어디서 문제가 생겼는지 한눈에 보여줍니다
- SQL 쿼리가 너무 많다 → 데이터베이스 문제일 수 있어요.
- 처리 시간이 오래 걸린다 → View 로직을 최적화해야 해요.
- 잘못된 템플릿 렌더링 → 잘못된 context 값을 확인할 수 있어요.
    
코드를 건드리지 않고도 내부 상태를 바로 볼 수 있습니다.
- 로그를 찍지 않아도 `request`, `settings`, `SQL`, `template` 등 거의 모든 정보를 실시간으로 확인 가능합니다.

---
🔹 Django Debug Toolbar 튜토리얼 & 디버깅 실습 자료

목표:
- 설문조사 앱에서 발생할 수 있는 SQL 비효율 (N+1 문제) 탐지
- 템플릿 context 디버깅
- 툴바 기능 익히기

📝 예제 시나리오: N+1 문제 유발하기: 
	기본적으로 `polls` 앱의 모델은 다음과 같습니다:
```python
# polls/models.py
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

💥 `실습용 뷰 작성: N+1 문제를 의도적으로 발생`
```python
# polls/views.py
from django.shortcuts import render
from .models import Question

def question_list(request):
    questions = Question.objects.all()  
    return render(request, 'polls/question_list.html', {'questions': questions})
```
`questions = Question.objects.all()` 
	모든 질문(`Question`) 객체를 가져오는 기본 ORM 명령어
	문제는 이 상태로 템플릿에서 `question.choice_set.count`를 사용할 경우, 질문마다 추가 SQL 쿼리가 실행됩니다.

`템플릿에서 관계 접근`
```html
<!-- templates/polls/questions.html -->
{% extends "polls/base.html" %}
{% block content %}

<h1>질문 목록</h1>
<ul>
  {% for question in questions %}
  <li>
    {{ question.question_text }} (선택지 수: {{ question.choice_set.count }})
  </li>
  {% empty %}
  <li>아직 등록된 질문이 없습니다.</li>
  {% endfor %}
</ul>
{% endblock %}
```
	여기서 .choice_set.count는 질문마다 SQL을 개별 실행하므로 N+1 문제
	발생

`URL 연결`
```python
# polls/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('questions/', views.question_list, name='question_list'),
]
```

◽ 디버그 툴바로 확인
- 브라우저에서 `http://127.0.0.1:8000/polls/questions/` 접속
- 툴바의 SQL 패널 클릭
- `SELECT ... FROM polls_choice WHERE question_id = ?` 쿼리가 질문 수만큼 발생하는 것을 확인

![[Pasted image 20250603150733.png]]
`쿼리 3개 실행됨`	polls_question에서 1개 + polls_choice에서 2개
`유사한 쿼리 감지됨`	choice_set.count 때문에 question당 1개씩 추가 쿼리 발생
`총 소요 시간`	0.29ms (아주 짧지만, 쿼리 수 증가가 중요)

`SELECT * FROM polls_question`
- 모든 질문을 조회하는 쿼리입니다.
- 질문 리스트를 보여주기 위한 기본 쿼리입니다.
    
 `SELECT COUNT(*) FROM polls_choice WHERE question_id = ?`
- 각 질문에 연결된 선택지의 개수를 세기 위한 쿼리입니다.
- `{{ question.choice_set.count }}` 사용으로 인해 발생합니다.
- 질문이 2개라서 쿼리도 2개 발생 (→ 질문이 100개면 100개 발생)

Debug Toolbar는 이처럼 질문 1개마다 쿼리 1개가 추가로 실행되는 비효율적인 구조를 `유사한 쿼리`로 묶어서 경고해줍니다.

---
⚠️ 이것이 바로 N+1 문제
- `N+1 문제`란?
	 하나의 쿼리로 N개의 객체를 가져온 후, 각각에 대해 추가 쿼리(N번)를 실행하는 비효율적인 패턴입니다.


개선 예시: `annotate()`를 이용한 사전 카운트 집계
```python
from django.db.models import Count

def question_list(request):
    questions = Question.objects.annotate(num_choices=Count('choice'))
    return render(request, 'polls/question_list.html', {'questions': questions})
```

`annotate()`
- 각 객체에 추가적인 계산 결과 필드를 붙이는 Django ORM 함수입니다.
- 여기서는 각 `Question` 객체에 `num_choices`라는 필드를 덧붙입니다.

`Count("choice")`
- `Question` 모델과 연결된 `Choice` 모델의 개수를 셉니다.
- 즉, 질문 하나당 연결된 선택지 개수를 DB에서 미리 계산합니다.

`그리고 템플릿 변경:`
```html
<li>{{ q.question_text }} (선택지 수: {{ q.num_choices }})</li>
```
→ 쿼리 수가 1개로 줄어듦 (Debug Toolbar SQL 패널에서 확인 가능)

◽  Debug Toolbar 활용 포인트 요약:
- `SQL`: SQL 패널	
	과도한 쿼리 확인 (N+1 문제)
- `Templates`	: Templates 패널	
	어떤 템플릿이 렌더링되고 있는지
- `Timer`	: Timer 패널	
	뷰 처리 시간 분석
- `Request` : Request 패널	
	GET/POST 값 확인 가능

---
쿼리(QUERY)란?
데이터베이스(DB)에게 무언가를 “요청”하는 명령어입니다. 
예를 들어 이런 명령이 바로 SQL 쿼리입니다:
```sql
SELECT * FROM polls_question;
```
→ "질문 테이블에서 모든 데이터를 가져와!" 라는 뜻입니다.  
Django는 우리가 `Question.objects.all()` 같은 파이썬 코드를 쓰면,  
뒤에서 자동으로 이런 SQL 쿼리를 만들어서 DB에 요청합니다.

❓ 그러면 “쿼리 3개 발생했다”는 건 무슨 뜻일까요?
	당신이 `/polls/questions/` 페이지를 열 때, Django는 화면을 그리기 위해 DB에 질문과 선택지 정보를 가져와야 해요.

현재 상황:
1. 질문 목록 1개 가져오는 쿼리 (SELECT * FROM polls_question)
2. 질문 1번의 선택지 개수 세는 쿼리 (COUNT FROM polls_choice WHERE question_id=1)
3. 질문 2번의 선택지 개수 세는 쿼리 (COUNT FROM polls_choice WHERE question_id=2)

즉, Django가 HTML을 보여주기 위해 총 3번 DB에 요청을 보내고 있는 거예요.  질문이 100개면? → 쿼리 101개가 실행될 수 있다는 뜻입니다.

💥 예: 서버 부하가 커지는 상황
```python
questions = Question.objects.all()
```
그리고 템플릿에서:
```html
<li>
  {{ question.question_text }} (선택지 수: {{question.num_choices }})
</li>
```
이건 질문 1개마다 쿼리가 하나씩 추가로 실행되니까  
질문이 많아지면 DB에 계속 부하를 줘요 (N+1 문제)

`그래서 이렇게`
```python
from django.db.models import Count

questions = Question.objects.annotate(num_choices=Count('choice'))
```
이건 Django가 질문과 연결된 선택지 수를 미리 계산해서 가져오게 하니까  
DB 쿼리가 한 번에 끝나고, 부하가 훨씬 줄어듭니다.

`Question.objects.all()` + `.choice_set.count()`
	각 질문마다 쿼리 발생 N+1
`annotate(Count('choice'))` 사용
	한 쿼리로 모두 처리 1개

![[Pasted image 20250603152857.png]]

```python
questions = Question.objects.annotate(num_choices=Count('choice'))
```
N+1 문제를 방지하기 위해 자주 쓰이는 공식 같은 코드입니다.  
그리고 실제로 쿼리셋 문제(과도한 쿼리 발생) 없이 효율적인 성능을 보장해주는 안전한 코드입니다.

사용 목적 요약:
`연관된 객체의 개수를 미리 알고 싶을 때` :  
	예: 질문당 선택지 개수
`리스트 뷰에서 반복적으로 .count() 호출하는 것을 방지하고 싶을 때` :  	
	성능 최적화
`쿼리 수(N+1 문제)를 줄이고 싶을 때` :  
	데이터가 많아질수록 효과적
`정렬 조건, 조건문 등에 개수를 활용하고 싶을 때` :  	
	예: 선택지 많은 질문만 보기

</> 예시코드: 선택지가 0개 이상 있는 질문만 필터링
```python
questions = Question.objects.annotate(num_choices=Count('choice')).filter(num_choices__gt=0)
```
	선택지가 1개 이상인 질문만 가져옴

</> 예시코드: 선택지 개수가 많은 순서로 정렬
```python
questions = Question.objects.annotate(num_choices=Count('choice')).order_by('-num_choices')
```
	가장 인기 있는(선택지가 많은) 질문을 먼저 보여줄 수 있음

</> 예시코드:
```html
{% if question.num_choices >= 3 %}
  <p>선택지가 충분합니다.</p>
{% else %}
  <p>선택지를 더 추가해주세요.</p>
{% endif %}
```
	추가 쿼리 없이 조건 분기 가능

- 사용하지 않아도 되는 상황:
	`질문 1개만 조회할 때` :`.choice_set.count()`1회는 큰 부담 아님
	`성능보다 코드 단순성이 중요한 경우` :  학습 목적, 빠른 테스트 등

---
비효율적인 템플릿 렌더링 문제 + Debug Toolbar로 디버깅하기

목표:
- 템플릿에서 `for` 루프 안에 `if`문과 `쿼리 접근`을 섞어 사용해 성능 이슈를 발생시킴
- Debug Toolbar의 `Templates`, `SQL`, `Timer` 패널을 활용해 문제를 추적
- `prefetch_related`로 해결하고 성능 향상 확인

모델 (`polls/models.py`)
```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField('date published')

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE, related_name='choices')
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

```python
from django.shortcuts import render, get_object_or_404
from .models import Question, Choice

def question_detail_manual(request, pk):
    # 일부러 select_related 또는 prefetch_related 사용하지 않음
    question = get_object_or_404(Question, pk=pk)

    # ❌ 여기에 불필요하게 모든 Choice 쿼리 실행 (질문 필터링 안함)
    all_choices = Choice.objects.all()

    # ❌ 템플릿에서는 question.choice_set.all 대신 전체 choice에서 filter 반복
    return render(request, "polls/detail_manual.html", {
        "question": question,
        "all_choices": all_choices,
    })
```

```html
{% extends "polls/base.html" %}
{% block content %}

<h2>{{ question.question_text }}</h2>
<p>선택지를 고르세요:</p>
<ul>
  {% for choice in all_choices %}
    {% if choice.question.id == question.id %}
      <li>{{ choice.choice_text }}</li>
    {% endif %}
  {% endfor %}
</ul>

{% endblock %}
```




