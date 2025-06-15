- 실제 사용 가능한 REST API를 설계하고 구현하는 전 과정을 익힌다.
- Django로 만든 웹앱을 RESTful API 서버로 전환하는 과정을 단계적으로 체험한다.
- 보안(인증/권한), 구조(링크 기반), 자동화(ViewSet/Router)를 통해 실무 수준의 API 구조를 익힌다.

```python
# 디렉토리 생성
mkdir drf_tutorial 
cd drf_tutorial

# 가상환경 설정
python3 -m venv venv
source venv/bin/activate

# 패키지 설치
pip install django
pip install djangorestframework

# 서드파티 설치 Browsable API용
pip install markdown

# 새 프로젝트 생성
django-admin startproject tutorial .

# 새 앱 생성
python manage.py startapp snippets
```

 `tutorial/settings.py`
```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'snippets',
]
```

`pygments` 외부 패키지
```bash
pip install pygments
```

`snippets/models.py`
```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted([(item, item) for item in get_all_styles()])


class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ['created']

```

```python
# 마이그레이션
python manage.py makemigrations snippets
python manage.py migrate
```
---
`snippets/serializers.py` 기본형
```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES

class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        for field in ['title','code','linenos','language','style']:
            setattr(instance, field, validated_data.get(field, getattr(instance, field)))
        instance.save()
        return instance
```

🔷 `ModelSerializer` 버전
`snippets/serializers.py`
```python
from rest_framework import serializers
from snippets.models import Snippet


class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style']
```

`snippets/views.py`
```python
from django.http import HttpResponse, JsonResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer


@csrf_exempt
def snippet_list(request):
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)


@csrf_exempt
def snippet_detail(request, pk):
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```

`snippets/urls.py`
```python
from django.urls import path
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]
```

`tutorial/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
	path('admin/', admin.site.urls),
    path('', include('snippets.urls')),
]
```

`API 테스트 방법: 서버 실행`
```bash
python manage.py runserver
```

###### 🔹 Insomnia 사용
| 요청                       | URL            | 메서드              | 설명                      |
| ------------------------ | -------------- | ---------------- | ----------------------- |
| 전체 목록 조회                 | `/snippets/`   | `GET`            | 모든 Snippet을 JSON으로 받아옴  |
| 새 Snippet 생성             | `/snippets/`   | `POST`           | JSON 데이터로 새 Snippet 추가  |
| 특정 Snippet 조회            | `/snippets/1/` | `GET`            | ID=1인 Snippet의 상세 정보 조회 |
| 특정 Snippet 수정 .......... | `/snippets/1/` | `PUT` or `PATCH` | ID=1인 Snippet 수정        |
| 특정 Snippet 삭제            | `/snippets/1/` | `DELETE`         | ID=1인 Snippet 삭제        |
```json
{
  "title": "Hello World",
  "code": "print('Hello, World!')",
  "language": "python",
  "style": "friendly",
  "linenos": true
}
```

![[Pasted image 20250607232829.png]]

`httpie로 GET 테스트`
```python
pip install httpie

# 전체 스니펫 목록
http GET http://127.0.0.1:8000/snippets/

# 특정 스니펫 상세
http GET http://127.0.0.1:8000/snippets/1/
```


