polls/urls.py
```python
from django.urls import path
from . import views

app_name = "polls"

urlpatterns = [
    path("", views.IndexView.as_view(), name="index"),
    path("<int:pk>/", views.DetailView.as_view(), name="detail"),
    path("<int:pk>/results/", views.ResultsView.as_view(), name="results"),
    path("<int:question_id>/vote/", views.vote, name="vote"),

    # 추가: CRUD
    path("create/", views.QuestionCreateView.as_view(), name="question_create"),
    path("<int:pk>/update/", views.QuestionUpdateView.as_view(), name="question_update"),
    path("<int:pk>/delete/", views.QuestionDeleteView.as_view(), name="question_delete"),
]
```

`polls/views.py`
```python
from django.urls import reverse_lazy
from .models import Question
from django.shortcuts import get_object_or_404, render
from django.views import generic

# 기존 IndexView, DetailView, ResultsView 생략

class QuestionCreateView(generic.CreateView):
    model = Question
    fields = ["question_text", "pub_date"]
    template_name = "polls/question_form.html"
    success_url = reverse_lazy("polls:index")

class QuestionUpdateView(generic.UpdateView):
    model = Question
    fields = ["question_text", "pub_date"]
    template_name = "polls/question_form.html"
    success_url = reverse_lazy("polls:index")

class QuestionDeleteView(generic.DeleteView):
    model = Question
    template_name = "polls/question_confirm_delete.html"
    success_url = reverse_lazy("polls:index")
```
---
`polls/templates/polls/question_form.html` : 기존코드
```html
{% extends "polls/base.html" %} 
{% block title %} 질문 작성/수정 {% endblock %}

{% block content %}
<h2>질문 입력</h2>
<form method="post">
  {% csrf_token %} 
  {{ form.as_p }}
  <button type="submit">저장</button>
</form>
{% endblock %}
```

`polls/templates/polls/question_form.html` : CSS와 UI를 위한 `수정`
```html
{% extends "polls/base.html" %}
{% block title %} 질문 작성/수정 {% endblock %}

{% block content %}
<form method="post">
  <div class="form-container">
    {% csrf_token %}

    <p>
      <label for="{{ form.question_text.id_for_label }}">질문:</label><br>
      <input type="text"
             name="{{ form.question_text.name }}"
             id="{{ form.question_text.id_for_label }}"
             value="{{ form.question_text.value|default_if_none:'' }}"
             placeholder="예: 당신이 좋아하는 음식은 무엇인가요?"
             required>
      {{ form.question_text.errors }}
    </p>

    <p>
      <label for="{{ form.pub_date.id_for_label }}">게시 날짜와 시간:</label><br>
      <input type="datetime-local"
             name="{{ form.pub_date.name }}"
             id="{{ form.pub_date.id_for_label }}"
             value="{{ form.pub_date.value|default_if_none:'' }}"
             placeholder="예: 2025-06-06T10:00"
             required>
      {{ form.pub_date.errors }}
    </p>

    <button type="submit" class="submit-btn">저장</button>
  </div>
</form>
{% endblock %}
```

`{{ form.as_p }}`코드를 풀어서 쓰면:
```html
<form method="post">
  {% csrf_token %}
  
  <p>
    <label for="{{ form.question_text.id_for_label }}">질문:</label>
    {{ form.question_text }}
    {{ form.question_text.errors }}
  </p>

  <p>
    <label for="{{ form.pub_date.id_for_label }}">날짜:</label>
    {{ form.pub_date }}
    {{ form.pub_date.errors }}
  </p>

  <button type="submit">저장</button>
</form>
```
---
`polls/templates/polls/question_form_delete.html`:기존코드
```html
{% extends "polls/base.html" %} 
{% block title %} 질문 삭제 {% endblock %}

{%block content %}
<h2>정말 삭제하시겠습니까?</h2>
<p>{{ object.question_text }}</p>

<form method="post">
  {% csrf_token %}
  <button type="submit">삭제</button>
  <a href="{% url 'polls:index' %}">취소</a>
</form>
{% endblock %}
```

`polls/templates/polls/question_form_delete.html`: `수정`
```html
{% extends "polls/base.html" %}
{% block title %}질문 삭제 확인{% endblock %}

{% block content %}
<div class="form-container" style="text-align: center;">
  <h2>정말 삭제하시겠습니까?</h2>
  <p><strong>{{ object.question_text }}</strong></p>

  <form method="post" style="margin-top: 20px;">
    {% csrf_token %}
    <button type="submit" class="btn-delete">삭제</button>
    <a href="{% url 'polls:index' %}" class="btn-cancel">취소</a>
  </form>
</div>
{% endblock %}
```
---
`polls/templates/polls/index.html` 기존코드
```html
{% extends "polls/base.html" %} {% load static %} 
{% block title %} 질문 목록{%endblock %} 

{% block content %}
<link rel="stylesheet" href="{% static 'polls/index.css' %}" />
<div class="container">
  <div class="header">
    <h2>설문 목록</h2>  
    <a href="{% url 'polls:question_create' %}" class="btn new-question-btn">새 질문</a>   
  </div>
  
  {% if latest_question_list %}
  <ul class="question-list">
    {% for question in latest_question_list %}
    <li class="question-item">
      <div class="question-text">
        <a href="{% url 'polls:detail' question.id %}">
        {{ question.question_text }}</a>
      </div>
      <div class="question-actions">
        <a href="{% url 'polls:question_update' question.id %}"
          class="btn edit-btn">수정</a>
        <a href="{% url 'polls:question_delete' question.id %}"
        class="btn delete-btn"> 삭제</a>
      </div>
    </li>
    {% endfor %}
  </ul>
  {% else %}
  <p>질문이 없습니다.</p>
  {% endif %}
</div>
{% endblock %}
```

`polls/templates/polls/index.html`: `수정`
```html
{% extends "polls/base.html" %} {% load static %}
{% block title %}<h1>최근 질문</h1>{% endblock %}

{% block content %}
{% if latest_question_list %}
<ul class="question-list">
  {% for question in latest_question_list %}
  <li class="question-item">
    <div class="question-text">
      <a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a>
    </div>
    <div class="question-actions">
      <a href="{% url 'polls:question_create' %}" class="button create">글생성</a>
      <a href="{% url 'polls:question_update' question.id %}" class="button update">수정</a>
      <a href="{% url 'polls:question_delete' question.id %}" class="button delete">삭제</a>
    </div>
  </li>
  {% endfor %}
</ul>
{% else %}
<p>No polls are available.</p>
{% endif %} {% endblock %}
```

index.css
```css
body {
  background-color: #f5f7fa;
  font-family: "Segoe UI", sans-serif;
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

.site-nav li {
  display: inline;
  background: none;
  box-shadow: none;
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

/* 질문 리스트 영역 */
.question-list {
  list-style: none;
  padding: 0;
  margin: 0 auto;
  max-width: 700px;
}

.question-item {
  background-color: white;
  border-radius: 12px;
  margin-bottom: 20px;
  padding: 20px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.08);
  text-align: left;
  transition: box-shadow 0.2s ease;
}

.question-item:hover {
  box-shadow: 0 6px 14px rgba(0, 0, 0, 0.12);
}

/*###-------index.html----------###*/
/* 질문 텍스트 */
.question-text {
  font-size: 18px;
  font-weight: 500;
  color: #2c2c2c;
  margin-bottom: 10px;
}
 
/* 버튼 영역 */
.question-actions {
  display: flex;
  gap: 8px;
}
 
/* 공통 버튼 스타일  */
.button {
  padding: 6px 12px;
  border-radius: 5px;
  font-size: 12px;
  font-weight: 500;
  color: white;
  text-decoration: none;
  transition: all 0.2s ease-in-out;
}

/* 각 버튼 색상*/
.button.create {
  background-color: #6c757d; /* 차분한 회색 */
}

.button.update {
  background-color: #495057; /* 짙은 회색 */
}

.button.delete {
  background-color: #adb5bd; /* 은은한 회색 */
}

/* 호버 시 강조 */
.button:hover {
  filter: brightness(1.15);
  transform: translateY(-1px);
}
 
/* 폼 전체 컨테이너 */
.form-container {
  max-width: 500px;
  margin: 40px auto;
  background-color: white;
  padding: 30px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.08);
}

/* 폼 필드 텍스트 */
.form-container label {
  font-weight: 500;
  color: #333;
  display: block;
  margin-bottom: 6px;
}

.form-container input {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ccc;
  border-radius: 6px;
  margin-bottom: 20px;
  font-size: 14px;
}

/* 저장 버튼 스타일 */
.submit-btn {
  background-color: #4a4e69;
  color: white;
  padding: 10px 20px;
  font-size: 14px;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.submit-btn:hover {
  background-color: #5c6085;
}

/*###-------question_form_delete.html----------###*/
/* 삭제 폼 버튼 스타일 */
.btn-delete {
  background-color: #e63946;
  color: white;
  border: none;
  padding: 10px 20px;
  margin-right: 10px;
  border-radius: 6px;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.btn-delete:hover {
  background-color: #c82333;
}
 
.btn-cancel {
  background-color: #dee2e6;
  color: #333;
  padding: 10px 20px;
  border-radius: 6px;
  text-decoration: none;
  font-size: 14px;
  transition: background-color 0.2s ease;
}

.btn-cancel:hover {
  background-color: #ced4da;
}

/*###-------results.html----------###*/
.results-container {
  max-width: 700px;
  margin: 50px auto;
  padding: 20px;
  text-align: center;
}

.question-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 30px;
  color: #2c2c2c;
}

.result-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.result-list li {
  background-color: white;
  border-radius: 10px;
  margin-bottom: 16px;
  padding: 14px 20px;
  box-shadow: 0 2px 6px rgba(0, 0, 0, 0.08);
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 16px;
  transition: background-color 0.2s ease;
}

.result-list li:hover {
  background-color: #f1f3f5;
}

.choice-text {
  font-weight: 500;
  color: #1d3557;
}

.vote-count {
  font-weight: 600;
  color: #495057;
}

/* 다시 투표 버튼 */
.vote-again {
  margin-top: 30px;
}

.vote-again-btn {
  display: inline-block;
  background-color: #4a4e69;
  color: white;
  padding: 10px 20px;
  border-radius: 8px;
  font-size: 14px;
  text-decoration: none;
  transition: background-color 0.2s ease;
}

.vote-again-btn:hover {
  background-color: #5f6483;
}

/* 별 이미지 */
.header {
  display: flex;
  align-items: center;
  justify-content: center;
  margin: 30px 0;
  gap: 10px;
  margin-bottom: 1em;
}
 
.heading-with-icon .icon {
  width: 50px;
  height: 50px;
}
```