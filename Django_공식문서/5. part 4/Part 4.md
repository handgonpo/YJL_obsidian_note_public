🔹 Django 튜토리얼 Part 4 – 폼 처리와 제너릭 뷰로 코드 간결화하기
###### 📖 공식 문서 링크:  
🔗 [https://docs.djangoproject.com/ko/5.2/intro/tutorial04/](https://docs.djangoproject.com/ko/5.2/intro/tutorial04/)

목표
- HTML 폼 만들기:
    사용자가 질문에 답하도록 라디오 버튼과 전송 버튼이 포함된 폼 작성
- POST 요청 처리하기:
    사용자가 선택한 값을 서버에서 받아 처리하는 로직 구현
- 예외 처리 구현하기:
    선택하지 않고 폼을 제출했을 때 오류 메시지 출력
- 투표 결과를 DB에 저장하기:
    사용자가 선택한 항목의 `votes` 값을 1 증가시킴
- 중복 제출 방지 (리다이렉션):
    `HttpResponseRedirect`를 사용해 새로고침 시 중복 투표 방지
- `reverse()` 함수 사용하기:
	URL을 하드코딩하지 않고 `name` 기반으로 동적 생성
- CSRF 보안 처리:
    `{% csrf_token %}`을 사용해 보안상 안전한 폼 구성
- 투표 결과 페이지 만들기:
    선택 결과를 보여주는 `results.html` 템플릿 구현
- 제너릭 뷰 도입:
    `ListView`, `DetailView`를 사용해 기존 뷰 코드 단순화
- URL 패턴 변경하기:
	제너릭 뷰에 맞춰 URL에서 `<int:question_id>`를 `<int:pk>`로 수정
- 템플릿 이름 지정하기 (`template_name`):
	Django의 기본 템플릿 명 대신, 우리가 만든 템플릿 파일을 명시
- `context` 변수 이름 변경 (`context_object_name`):
	템플릿에서 사용할 변수명을 명확하게 지정해 가독성 향상

---
✨ 전체코드 실습부터 하기 CBV (Class-Based View)로 변경

`polls/views.py`
```python
from django.db.models import F
from django.urls import reverse
from django.views import generic
from django.http import HttpResponseRedirect

# 메인 페이지 (질문 목록)
class IndexView(generic.ListView):
    template_name = "polls/index.html"
    context_object_name = "latest_question_list"

    def get_queryset(self):
        return Question.objects.order_by("-pub_date")[:5]


# 질문 상세 페이지
class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"
	context_object_name = "question"

# 결과 페이지
class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"
	context_object_name = "question"

# 투표 처리 로직
def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST["choice"])
    except (KeyError, Choice.DoesNotExist):
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message":"You didn't select a choice.",
            },
        )
    else:
        selected_choice.votes = F("votes") + 1
        selected_choice.save()
        return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
```

`polls/urls.py`
```python
from django.urls import path
from . import views

app_name = "polls"

urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),
]
```
---
📁 html을 작성하기 위해 템플릿 폴더 및 html파일 추가 구조:
```
Django_first_for/   
├── manage.py       
├── mysite/         
│   ├── __init__.py
│   ├── asgi.py
│   ├── settings.py 
│   ├── urls.py     ← 전체 URL 라우팅 설정
│   └── wsgi.py     

├── polls/          
│   ├── __init__.py
│   ├── admin.py    
│   ├── apps.py     
│   ├── migrations/  
│   │   └── __init__.py
│   ├── models.py    ← 데이터베이스 모델 정의
│   ├── tests.py     
│   ├── views.py     ← 뷰 함수 정의 (index, detail, vote 등)
│   ├── urls.py      ← polls 전용 URL 설정

│   ├── static/                ← 정적 파일 디렉토리
│   │   └── polls/             ← polls 앱 전용 static 디렉토리
│   │       ├── styles.css     ← index 페이지 스타일
│   │       ├── detail.css     ← detail 페이지 스타일
│   │       └── results.css    ← results 페이지 스타일

│   ├── templates/             ← 템플릿 디렉토리
│       └── polls/             ← polls 앱 전용 템플릿 네임스페이스
│           ├── base.html      ← 공통 레이아웃 템플릿
│           ├── header.html    ← 공통 상단 템플릿
│           ├── footer.html    ← 공통 하단 템플릿
│           ├── index.html     ← 설문 목록 페이지
│           ├── detail.html    ← 설문 상세 페이지 (투표 가능)
│           └── results.html   ← 설문 결과 페이지
```
---
`polls/templates/polls/detail.html`
```html
{% extends "polls/base.html" %}
{% load static %}

{% block title %}{{ question.question_text }}{% endblock %}

{% block content %}
<link rel="stylesheet" href="{% static 'polls/detail.css' %}">

<div class="detail-container">
  <h2 class="question-title">{{ question.question_text }}</h2>

  <form action="{% url 'polls:vote' question.id %}" method="post" class="vote-form">
    {% csrf_token %}
	{% if error_message %}
    <p class="error-message"><strong>{{ error_message }}</strong></p>
	{% endif %}

      <ul class="choice-list">
        {% for choice in question.choice_set.all %}
        <li>
          <label>
        <input type="radio" name="choice" value="{{ choice.id }}">
            {{ choice.choice_text }}
          </label>
        </li>
        {% endfor %}
      </ul>
    <div class="submit-btn-wrap">
      <input type="submit" value="Vote" class="submit-btn">
    </div>
  </form>
</div>
{% endblock %}
```
![[Pasted image 20250531220210.png]]

---
`polls/templates/polls/results.html`
```html
{% extends "polls/base.html" %}
{% load static %}

{% block title %}{{ question.question_text }}{% endblock %}

{% block content %}
<link rel="stylesheet" href="{% static 'polls/results.css' %}">

<div class="results-container">
  <h2 class="question-title">{{ question.question_text }}</h2>

  <ul class="result-list">
    {% for choice in question.choice_set.all %}
      <li>
        <span class="choice-text">{{ choice.choice_text }}</span>
        <span class="vote-count">{{ choice.votes }} vote{{ choice.votes|pluralize }}</span>
      </li>
    {% endfor %}
  </ul>

  <div class="vote-again">
    <a href="{% url 'polls:detail' question.id %}" class="vote-again-btn">Vote again?</a>
  </div>
</div>
{% endblock %}

```
![[Pasted image 20250531224515.png]]

---
🔹 제너릭 뷰(Generic View)란?
	제너릭 뷰는 목록 보기(List), 상세 보기(Detail), 생성(Create), 수정(Update), 삭제(Delete) 등의 일반적인 작업을 빠르게 구현할 수 있도록 제공되는 클래스 뷰입니다. Django에서 자주 쓰이는 패턴을 간편하게 처리할 수 있도록 미리 만들어 놓은 클래스 기반 뷰(CBV)입니다.

📖 기본구조(Syntax)
```python
from django.views import generic

class MyView(generic.XXXView):#ListView,DetailView...
    model = MyModel
    template_name = 'app/template.html'
    context_object_name = 'object_name'

    def get_queryset(self):
        return MyModel.objects.all()
```

◽ 자주 사용하는 제너릭 뷰 종류:
- `ListView`: 객체 리스트를 보여줌
- `DetailView`: 하나의 객체 상세 보기
- `CreateView`: 객체 생성 폼 제공 및 저장
- `UpdateView`: 객체 수정
- `DeleteView`: 객체 삭제
- `TemplateView` : 템플릿 렌더링만 수행
- `RedirectView` : URL 리다이렉트 수행

💡 제너릭 뷰를 상속받아 개발자가 커스터마이징 할수 있습니다:
	- `IndexView`는 `ListView` 등을 상속받아 "인덱스 페이지용으로 커스터마이징해서 만든 클래스 이름입니다.
	- `ResultsView`는 Django의 제너릭 `DetailView`를 상속받아 만드는 뷰입니다.

◽ 공통속성(Attributes)
- `model` :연결할 모델 클래스
- `template_name` :사용할 템플릿 경로
- `context_object_name` :
	템플릿에 넘겨주는 이름 (기본: object 또는 object_list)
- `success_url` : 
	CreateView, UpdateView, DeleteView에서 성공 후 이동할 URL
- `form_class` : 사용할 폼 클래스 (ModelForm 등)

◽ 주요 메서드 (함수 오버라이드용)
- `get_queryset(self)` : 리스트로 반환할 쿼리셋 정의
- `get_context_data(self, **kwargs)` :템플릿에 전달할 context 수정 가능
- `form_valid(self, form)` :폼이 유효할 때 수행할 작업 정의
- `get_object(self)`	: DetailView, UpdateView 등에서 객체 가져오는 로직
---
</> 예시 1: `ListView`
```python
from django.views.generic import ListView
from .models import Question

class QuestionListView(ListView):
    model = Question  # 어떤 모델의 데이터를 보여줄지
    template_name = 'polls/question_list.html' 
    # 사용할 템플릿 경로
    
    context_object_name = 'question_list'
    # 템플릿에서 사용할 객체 이름
    
    paginate_by = 10  # 페이지당 항목 수 (선택사항)

    def get_queryset(self):
        return Question.objects.order_by('-pub_date')
        # 내림차순 정렬, 즉 후입선출(LIFO) 방식입니다.
```
- `order_by('pub_date')`:  
    → 오름차순 정렬  
    → 오래된 항목이 먼저 옴 (예: 2020 → 2021 → 2022)
    
- `order_by('-pub_date')`:  
    → 내림차순 정렬  
    → 최근 항목이 먼저 옴 (예: 2022 → 2021 → 2020)

---
</> 예시 2: `DetailView`
```python
from django.views.generic import DetailView
from .models import Question

class QuestionDetailView(DetailView):
    model = Question
    template_name = 'polls/question_detail.html'
    context_object_name = 'question'  # 템플릿에서 사용할 변수명
```

`context_object_name = 'question'` : 변수명 속성할당
	템플릿에서 사용할 변수명을 `question`으로 지정하겠다”는 뜻입니다.

---
</> 예시 3: `CreateView`
```python
from django.views.generic.edit import CreateView
from django.urls import reverse_lazy
from .models import Question

class QuestionCreateView(CreateView):
    model = Question
    fields = ['question_text', 'pub_date']  # 폼에서 입력받을 필드
    template_name = 'polls/question_form.html'
    success_url = reverse_lazy('polls:index')  # 저장 후 이동할 URL
```

✨ `fields`속성할당이란?
	어떤 모델 필드를 사용자 입력 폼에 표시할지 지정하는 것입니다.

`fields = ['question_text', 'pub_date']` :
	모델의 어떤 필드를 폼에 포함시킬 것인지 지정하는 리스트입니다.

🤔 상황가정: 사용자 이름과 전화번호 입력 받기

모델 만들기 (`models.py`)
```python
from django.db import models

class Contact(models.Model):
    name = models.CharField(max_length=100)         # 사용자 이름
    phone_number = models.CharField(max_length=20)  # 전화번호
```

뷰 만들기 (`views.py`)
```python
from django.views.generic.edit import CreateView
from .models import Contact
from django.urls import reverse_lazy

class ContactCreateView(CreateView):
    model = Contact
    fields = ['name', 'phone_number']  
    # 여기에서 어떤 입력 필드를 폼에 표시할지 지정
    
    template_name = 'contact_form.html'
    success_url = reverse_lazy('success')
```
	즉, "이름과 전화번호 필드를 포함한 입력 폼을 자동으로 만들어줘" 라고 
	Django에게 말하는 거예요.

템플릿 만들기 (`templates/contact_form.html`)
```html
<h2>연락처 입력</h2>
<form method="post">
    {% csrf_token %}
    {{ form.as_p }}  
    <!-- 여기서 위에서 지정한 fields가 폼 필드로 자동 출력됨 -->
    
    <button type="submit">저장</button>
</form>
```
이렇게 하면 자동으로 HTML에서 폼이 만들어집니다:
```html
<input type="text" name="name">
<input type="text" name="phone_number">
```
즉, `fields`는 "모델의 어떤 필드를 입력 폼으로 보여줄지" 정하는 거예요.

---
1️⃣ 제너릭 뷰로 뷰 함수 줄이기 `polls/views.py`

`IndexView` (목록 뷰)
```python
from django.views import generic # ListView 클래스 기반 뷰 제공
from .models import Question # 사용자 정의 모델 Question 사용

print("모듈속성목록",dir(generic)) 
# 제너릭안에 속성을 화면에 표시

class IndexView(generic.ListView): # 제너릭 뷰(ListView) 상속
    template_name = "polls/index.html" # 속성할당
    context_object_name = "latest_question_list" # 속성할당

    def get_queryset(self): # DB에서 데이터를 가져오는 역할
        return Question.objects.order_by("-pub_date")[:5]
```
- `ListView`를 상속받아 Question 목록을 보여줍니다.
	Django에서 여러 개의 데이터를 목록 형태로 보여주는 뷰(View)를  
	자동으로 만들어주는 클래스입니다.

- `ListView가 하는 일`
	- 데이터베이스에서 여러 개의 객체 가져와서 웹에서 보여줄때
	    → 예: `Question.objects.all()`
	- 템플릿에 그 목록을 전달하기  
	    → 예: `latest_question_list`라는 이름으로 넘겨줌
	- HTML로 자동 렌더링하기  
	    → 예: `polls/index.html` 파일에 연결

🔄 `IndexView` 처리 흐름 시각화
```python
[get_queryset()] → DB에서 데이터 가져옴
        ↓
context_object_name = "latest_question_list" 
# 가져온 데이터를 어떤 이름으로 템플릿에 넘길지 결정
        ↓
템플릿에 {{ latest_question_list }}로 전달됨
```

`polls/views.py(get_queryset)`  
```python
def get_queryset(self):
    return Question.objects.order_by("-pub_date")[:5]
```
- `get_queryset()` : DB에서 데이터를 가져옴.
- `[:5]` 슬라이싱을 통해 최신 5개 데이터만 가져오도록 제한하고 있음.

---
`polls/views.py(DetailView)`
```python
class DetailView(generic.DetailView):
    model = Question
    template_name = "polls/detail.html"
```
- `DetailView`는 기본적으로 URL에서 전달된 "pk" 값을 자동으로 인식해서  DB에서 해당 데이터를 찾아 보여주는 기능이 속성처럼 내장되어 있습니다.

```python
path("<int:pk>/", views.DetailView.as_view(), name="detail")
```
- `<int:pk>`는 `question.id` 같은 기본 키(primary key) 값을 의미합니다.
- 사용자가 `/polls/3/` 이런 URL에 접근하면 → `pk=3`이 자동으로 뷰에 전달
- 그러면 위의 views코드에서 DetailView함수가 작동합니다.
- `model = Question`: 이 모델에서 데이터를 찾겠다는 뜻이고,
- `pk=3`: 이 키로 `Question.objects.get(pk=3)`처럼 Django가 자동 조회함.

❓ `return render(...)`도 없고, `HttpResponse`도 없는데  어떻게 
    `polls/results.html`로 이동되는가?
    → Django의 `DetailView` 클래스가 내부적으로 자동 처리합니다.

🔄 `DetailView` 처리 흐름 시각화
```python
URL (예: /polls/3/) 
   ↓
pk=3 자동 추출
   ↓
DetailView가 Question 모델에서 pk=3인 객체 조회
   ↓
context = {"question": 조회된 객체}
   ↓
polls/detail.html에 렌더링
```

---
`polls/views.py(ResultsView)`
```python
class ResultsView(generic.DetailView):
    model = Question
    template_name = "polls/results.html"
```
 `DetailView와 ResultsView` 클래스가 하는 일은 딱 2가지입니다.
	 URL에서 전달된 `pk` 값으로 `Question.objects.get(pk=전달된 pk)` 수행
	 그 객체를 `question`이라는 이름으로 템플릿에 넘긴 뒤 지정된 `polls/results.html` 템플릿을 렌더링

---
2️⃣ URL 패턴 등록
```python
path("<int:pk>/", views.DetailView.as_view(), name="detail"),
```
- `.as_view()` 이것이 함수명뒤에 붙은건 클래스라는 뜻입니다.
- `DetailView`와 `ResultsView`는 `pk`로 primary key 값을 받습니다.
- `name="detail"` 이건 URL 경로에 붙이는 이름표(Tag)입니다.
`템플릿에서 URL을 생성할 수 있게 해줌`
```python
<a href="{% url 'polls:detail' question.id %}">
```
- 이때 `'polls:detail'` 중 `"detail"`이 바로 `name="detail"`입니다.  
	이 이름을 기반으로 Django가 내부적으로 **`/polls/3/`** 같은 주소를 만들어줍니다.

---
3️⃣ `try-except-else` 구문 블록의 역할
```python
try:
    # 예외가 발생할 수 있는 코드
except (예외 종류):
    # 예외가 발생했을 때 실행할 코드
else:
    # 예외가 발생하지 않았을 때 실행할 코드
```

`vote()` 함수에 적용된 구조
```python
try:
    selected_choice = question.choice_set.get(pk=request.POST["choice"])
except (KeyError, Choice.DoesNotExist):
    return render(...) # 오류 메시지와 함께 다시 detail 페이지 보여주기
else:
    selected_choice.votes = F("votes") + 1
    selected_choice.save()
    return HttpResponseRedirect(...)
```
- `try` : 에러가 날 수 있는 의심 구문을 시도합니다.	
	`request.POST["choice"]` → 사용자가 아무 선택 안 했을 수도 있습니다.
- `except` :	에러가 나면 여기로 점프합니다.	
	선택 안 했거나 잘못된 choice ID인 경우, 오류 메시지와 함께 다시 질문 페이지 보여줍니다
- `else` : try에서 에러가 안 나야 실행됩니다.	
	선택이 정상적으로 되었으면 → votes 증가 + 저장 + 결과 페이지로 이동합니다.
---
```python
from django.shortcuts import get_object_or_404
from .models import Question

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
```
`question_id` : URL을 통해 전달받는 질문 번호입니다.
	예: `/polls/3/vote/`이면 `question_id = 3`
	만약 해당 질문이 존재하면 그 객체를 `question`에 저장하고,
	만약 질문이 없으면: ❌ 자동으로 404 Not Found 에러 페이지를 보여줍니다. 
![[Pasted image 20250528132109.png]]

---
4️⃣ 폼 처리 (투표 기능)
```python
selected_choice = question.choice_set.get(pk=request.POST["choice"])
```
- 사용자가 제출한 라디오 버튼의 값(choice ID)을 가져옵니다.
- 없으면 `KeyError` 예외 → 에러 메시지 출력.
![[Pasted image 20250528132924.png]]
```python
except (KeyError, Choice.DoesNotExist):
        return render(
            request,
            "polls/detail.html",
            {
                "question": question,
                "error_message": "You didn't select a choice.",
            },
        )
```
- 사용자가 레디오 버튼을 클릭하지 않고 Vote이라는 버튼을 눌러 데이터를 전송했으므로 예외가 발생하여 error_message가 출력되었습니다.

`"polls/detail.html"` 탬플릿으로 이동하여 조건에 맞으면 화면에 뿌려준다.
```html
{% if error_message %}
  <p><strong>{{ error_message }}</strong></p>
{% endif %}
```
- 템플릿에서는 `error_message`가 존재할 때 해당 문구를 보여줍니다.
---
`render()`: 랜더함수는 템플릿을 불러오고,  데이터(context)를 함께 넘겨서 HTML 응답을 만드는 함수입니다. context는 문맥 정보이며 딕셔너리 구조로 넘겨집니다.

📖 문법, 구문(syntax): 
```python
from django.shortcuts import render

render(request, template_name, context=None, content_type=None, status=None, using=None)
```

```python
return render(request, "polls/detail.html", {"question": question})

return render(request, "polls/detail.html", {"error_message": "You didn't select a choice."})
```
위의 코드처럼 작성되면 탬플릿에서는 아래 코드와 같이 사용이 가능해 집니다.
```html
{{ question.question_text }}
```
---
```python
from django.db.models import F

selected_choice.votes = F("votes") + 1
```
- 위 코드는 Django ORM에서 데이터베이스의 값을 직접 연산해서 업데이트할 때 사용하는 코드입니다.
- 이 코드는 현재 투표 수(`votes`)에 1을 더하는 연산을 DB 수준에서 수행하는 Django 방식입니다. `selected_choice.votes += 1`와 같이 1씩 증가시키는 것입니다.
- `F("votes")`는 DB안의 `votes` 필드 값을 그대로 가져오는 Django의 특수 객체입니다.
- `selected_choice = question.choice_set.get(pk=request.POST["choice"])`
	사용자가 선택한 선택지(Choice)를 DB에서 가져와서 만든 객체변수입니다.
- `selected_choice`는 `try` 블록 안에서 정의된 지역 변수(local variable)이며,
	`try` 안에서 정의되었기 때문에, 그 아래 `else` 블록에서도 계속 사용할 수 있습니다.
---
```python
from django.http import HttpResponseRedirect

return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
```

◽ `reverse()` 함수란?
	Django의 URL 설정에서는 종종 아래처럼 `name="something"`을 지정합니다:
```python
path("<int:pk>/results/", views.ResultsView.as_view(), name="results")
```
- 이때 `reverse("polls:results", args=(3,))`를 쓰면 `/polls/3/results/`라는 URL 경로가 자동 생성됩니다.

◾ 상황: 
	사용자가 어떤 질문을 클릭해서 투표를 합니다.  
    예: 질문 id가 `2`인 항목에 투표
- 서버는 그 선택지의 질문 id (즉, `question.id`)를 파악합니다.
- 그런 다음 다음 코드가 실행됩니다:
```python
return HttpResponseRedirect(reverse("polls:results", args=(question.id,)))
```
- `question.id` : 사용자가 방금 투표한 질문의 id 값 (예: 2)
- `reverse("polls:results", args=(2,))` : `"polls:results"`라는 URL name에 `2`를 넣어 실제 URL을 생성 → `/polls/2/results/`를 생성해줌
- `HttpResponseRedirect(...)` : 그 URL로 브라우저를 리디렉션시킴 (이동시킴)
---
  5️⃣ HTML 폼 구조  (`detail.html`)
	사용자가 질문에 대한 선택지를 하나 고른 뒤,  "Vote" 버튼을 눌러 
	서버에 선택값을 제출하는 역할을 합니다.
```html
<form action="{% url 'polls:vote' question.id %}" method="post">
```
`action="{% url 'polls:vote' question.id %}"`
	사용자가 폼을 제출했을 때 요청이 전송될 URL을 동적으로 생성
	→ 예: `/polls/2/vote/`
`method="post"`
	폼 데이터를 POST 방식으로 서버에 전송함

---
🔗 [[GET과 POST의 차이점]]

CSRF (Cross-Site Request Forgery): 
	로그인된 사용자를 속여서, 원래 의도하지 않은 요청을 보내게 만드는 공격을 말합니다.
	
이해를 돕기 위한 예시 상황:
당신이 어떤 은행 사이트에 로그인한 상태이고, 브라우저는 로그인 세션 쿠키를 자동으로 보내고 있어요.   해커가 만든 악성 사이트의 폼(form)에 무심코 접근하거나 클릭하면,  당신이 의도하지 않았지만, 로그인된 상태로 은행에 요청이 전송됩니다.  서버는 단순히 요청만 보고는 "진짜 사용자 요청인지" 판단할 수 없습니다.

그래서 Django는 각 `POST` 요청에 `{% csrf_token %}`이라는 고유 토큰을 요구합니다.
- 이 토큰이 있으면: 정상적인 폼 요청으로 인식해 처리하고
- 이 토큰이 없거나 틀리면: CSRF 공격으로 간주하고 거부합니다.
---
```html
{% csrf_token %}
<fieldset>
    <legend>
	    <h1>{{ question.question_text }}</h1>
    </legend>
    {% if error_message %}
        <p><strong>{{ error_message }}</strong></p>
    {% endif %}
    {% for choice in question.choice_set.all %}
        <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
        <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}
        </label><br>
    {% endfor %}
</fieldset>
```

- `{% csrf_token %}`  
	CSRF(Cross-Site Request Forgery) 공격을 방지하기 위한 토큰
	→ 반드시 POST 요청에서는 포함해야 함

- `<legend><h1>{{ question.question_text }}</h1></legend>`
	`{{ question.question_text }}` : 현재 질문의 내용 출력
	→ 예: "What's new?"

- `오류 메시지 블록`
```html
{% if error_message %}
  <p><strong>{{ error_message }}</strong></p>
{% endif %}
```
- 사용자가 아무 항목도 고르지 않고 제출했을 때,
- 서버에서 전달한 `error_message`를 표시함

- `선택지 반복 출력`
```html
{% for choice in question.choice_set.all %}
    <input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}">
    <label for="choice{{ forloop.counter }}">{{ choice.choice_text }}
    </label><br>
{% endfor %}
```
- `for choice in question.choice_set.all`
	현재 질문에 연결된 모든 선택지를 반복 출력
- `<input type="radio" name="choice"`
	사용자에게 보이는 라디오 버튼 생성
- `value="{{ choice.id }}"`
	선택한 항목의 id가 서버로 전달됨
- `id="choice{{ forloop.counter }}"`
	각 항목에 고유한 ID 부여 (접근성과 label 연결용)
- `<label for="...">`
	라디오 버튼 옆에 표시될 선택지 텍스트

`제출 버튼`
```python
<input type="submit" value="Vote">
```
- 버튼 클릭 시: 
	선택한 항목을 포함해 폼 전체가 서버로 제출됨 (POST)
---
6️⃣ HTML 폼 구조  (`results.html`)
```html
<h1>{{ question.question_text }}</h1>
```

`모델(models.py`)`
```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
```
- `question_text` : html에 호출된 변수는 모델에서 생성한 객체의 이름임.

🔁 구조 흐름
```python
[models.py]
question_text = models.CharField(...)
     ↓
[DB 저장]
"What's new?" → question_text 컬럼에 저장
     ↓
[views.py]
question = Question.objects.get(pk=...)
     ↓
[results.html]
{{ question.question_text }} → "What's new?" 출력
```
---
`results.html`
```html
<ul>
    {% for choice in question.choice_set.all %}
        <li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
    {% endfor %}
</ul>
```
- `question.choice_set.all`
	-`question`은 하나의 질문(Question 객체)
	-`choice_set`은 Question과 연결된 모든 Choice 객체를 가져오는 역참조 이름
	-`.all`은 해당 질문에 대한 모든 선택지를 가져오는 함수
	
`choice_set`에서 `_set`은 Django가 자동으로 만들어주는 역참조 이름입니다.
예를 들어, `ForeignKey(Question)` → `Choice` 라고 하면,  
Django는 자동으로 `question.choice_set`이라는 이름으로 연결된 데이터를 다룰 수 있도록 해줍니다.

만약 이 자동 생성된 이름이 마음에 들지 않으면,  
`ForeignKey`에 `related_name` 속성을 지정하여 이름을 아래 코드와 같이 
변경할 수 있습니다.
```python
question = models.ForeignKey(Question, on_delete=models.CASCADE, related_name="choices")
```
그러면 아래 코드와 같이 사용할수 있습니다.
```python
# 역참조로 자동생성해준 이름
q.choice_set.create(choice_text="Not much", votes=0)

# 이름 변경후 사용방식
q.choices.create(choice_text="Not much", votes=0)
```
---
`results.html`
```html
<li>{{ choice.choice_text }} -- {{ choice.votes }} vote{{ choice.votes|pluralize }}</li>
```
`{{ choice.choice_text }}` 각 선택지(Choice)의 내용(문구)를 출력합니다. 예: `"The sky"`
`--` 단순한 텍스트 구분 기호입니다. HTML 구조에는 영향 없음
`{{ choice.votes }}` 해당 선택지의 투표 수를 출력합니다.

---
◽ `pluralize` 필터란? 
- `|pluralize`는 숫자에 따라 단어 뒤에 복수형 's'를 자동으로 붙여줍니다.
- `1`일 때는 아무것도 붙이지 않고, `0`, `2`, `3` 등일 때는 `s`를 붙입니다.
- 다른 언어에도 맞춰 커스터마이징 가능합니다:  
    예) `|pluralize:"개,개들"`처럼 한글도 가능

```html
vote{{ choice.votes|pluralize }}
```
`choice.votes`가 1이면 → `vote` 
`choice.votes`가 0 또는 2 이상이면 → `votes`

`태그로 변환하면 아래 코드와 같이 출력됩니다:`
```html
<li>Not much -- 1 vote</li>
<li>The sky -- 0 votes</li>
<li>Just hacking again -- 3 votes</li>
```
	- 0은 ‘없음’을 뜻하지만, 영어에서는 복수형을 씁니다. 그 이유는 ‘0개
	의 항목’이라는 개념이 복수 형태를 지칭하기 때문입니다.
	- 수학적 개념에서도  0은 1 미만의 수로서, 여러 개의 항목이 있는 
	상황에서 개수가 "없다"는 것을 나타냅니다.
	- 즉, 개수가 존재하지 않아도 개체 집합은 여전히 "다수 개체를 셀 
	수 있는 항목들"이기 때문에 복수로 취급합니다.

![[Pasted image 20250601102024.png]]

`커스터마이징도 가능해요`
`pluralize` 필터는 영문 외에도 한글이나 다른 형태로도 바꿀 수 있습니다.
```html
{{ count|pluralize:"개,개들" }}
```
- `count = 1`이면 → `개`
- `count = 2`면 → `개들`
---
6️⃣ URL 네임스페이스 활용
```python
<a href="{% url 'polls:detail' question.id %}">Vote again?</a>
```
- `<a href="...">` : 
	HTML의 하이퍼링크 태그로, 링크를 클릭하면 특정 페이지로 이동합니다.
- `{% url 'polls:detail' question.id %}`
	Django 템플릿 태그입니다. 해당 이름의 URL을 찾아서 실제 경로로 바꿔줍니다.
![[Pasted image 20250528172746.png]]

---
❓ `{% url 'polls:detail' question.id %}`는 무슨 뜻?
- `'polls:detail'`: 
	`polls/urls.py`의 path함수에서 보면,
	`path("<int:pk>/", views.DetailView.as_view(), name="detail")`
	`polls` 앱 안에 있는 **`name='detail'`** 이라고 정의된 URL을 찾습니다.

- `question.id`
```
Question(id=2, question_text="What's new?")
 ├── Choice(id=1, choice_text="Not much")
 ├── Choice(id=2, choice_text="The sky")
 └── Choice(id=3, choice_text="Just hacking again")
```
- 위처럼 질문(Question) 하나에 선택지(Choice) 여러 개가 연결되어 있어요.
- `question.id`는 이 질문의 ID (여기서는 2)
![[Pasted image 20250528174543.png]]
```python
path("<int:pk>/", views.DetailView.as_view(), name="detail")
```
즉 `<int:pk>`가 `question.id`이다.

---
- `choice.id`는 각 선택지의 고유 ID (1, 2, 3)
![[Pasted image 20250528174626.png]]

