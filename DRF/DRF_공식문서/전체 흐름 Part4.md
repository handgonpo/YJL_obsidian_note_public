모델에 사용자 연결
`Snippet` 모델 필드 추가 + `.save()` 오버라이드
`snippets/models.py`
```python
from django.contrib.auth.models import User
from pygments.lexers import get_lexer_by_name
from pygments.formatters.html import HtmlFormatter
from pygments import highlight

class Snippet(models.Model):
    # ... 기존 필드 생략 ...
    
    # 새로 추가된 필드
    owner = models.ForeignKey(User, related_name='snippets', on_delete=models.CASCADE)
    highlighted = models.TextField()

	# 코드 저장 시 자동으로 하이라이트 처리
    def save(self, *args, **kwargs):
        lexer = get_lexer_by_name(self.language)
        linenos = 'table' if self.linenos else False
        options = {'title': self.title} if self.title else {}
        formatter = HtmlFormatter(style=self.style, linenos=linenos, full=True, **options)
        self.highlighted = highlight(self.code, lexer, formatter)
        super().save(*args, **kwargs)
```

`마이그레이션 (초기화 방식으로)`
```bash
rm -f db.sqlite3
rm -r snippets/migrations
python manage.py makemigrations snippets
python manage.py migrate
python manage.py createsuperuser  # 테스트 사용자 생성
```
---
`owner` 필드 추가, `UserSerializer` 추가
`snippets/serializers.py`
```python
from rest_framework import serializers
from snippets.models import Snippet
from django.contrib.auth.models import User

class SnippetSerializer(serializers.ModelSerializer):
	# ... 기존 필드 생략 ...
    owner = serializers.ReadOnlyField(source='owner.username')
	highlighted = serializers.ReadOnlyField() # 두개 필드추가

    class Meta:
        model = Snippet
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style', 'owner', 'highlighted'] # owner, highlighted두개 필드 추가

# owner 사용자 API뷰 추가
class UserSerializer(serializers.ModelSerializer):
    snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())

    class Meta:
        model = User
        fields = ['id', 'username', 'snippets']
```

`SnippetSerializer`에 추가:
```python
owner = serializers.ReadOnlyField(source='owner.username')
highlighted = serializers.ReadOnlyField()

	model = Snippet
        fields = ['owner', 'highlighted']
```
---
`snippets/permissions.py` (신규 파일 생성)
```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
	    # 읽기 요청이면 (GET, HEAD, OPTIONS)
        if request.method in permissions.SAFE_METHODS:
            return True
        # 수정/삭제 요청이면: 객체의 소유자만 가능
        return obj.owner == request.user
```
- 사용자 A는 자신의 글만 수정/삭제하고
- 사용자 B는 A의 글을 읽기만 가능하게 하려면
- 이 권한 클래스가 필요합니다.

---
`views.py` – 권한, 사용자 뷰, perform_create 추가
```python
from rest_framework import generics, permissions
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer, UserSerializer
from snippets.permissions import IsOwnerOrReadOnly
from django.contrib.auth.models import User

class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    
    # 권한 추가
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)

class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    
    # 권한 추가
    permission_classes = [permissions.IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly]

# 사용자용 뷰 추가
class UserList(generics.ListAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer

class UserDetail(generics.RetrieveAPIView):
    queryset = User.objects.all()
    serializer_class = UserSerializer
```





---
`snippets/urls.py`
```python
from django.urls import path
from snippets import views

urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
    path('users/', views.UserList.as_view()),
    path('users/<int:pk>/', views.UserDetail.as_view()),
]
```

`프로젝트 루트 tutorial/urls.py`
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("", include("snippets.urls")),
    path("api-auth/", include("rest_framework.urls")),  # 추가
]
```

