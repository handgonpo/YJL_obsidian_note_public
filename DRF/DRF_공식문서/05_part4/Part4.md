🔹 DRF 튜토리얼 Part4 _ Authentication & Permissions

📖 공식 문서 링크:  
🔗 [https://www.django-rest-framework.org/tutorial/4-authentication-and-permissions/](https://www.django-rest-framework.org/tutorial/4-authentication-and-permissions/)

목표:
- `사용자와 코드 스니펫 연결`	
	Snippet을 작성한 사용자를 owner로 지정
- `인증된 사용자만 쓰기 가능`	
	로그인한 사용자만 Snippet 생성/수정/삭제 가능
- `익명 사용자는 읽기만 가능`	
	로그인하지 않은 사용자는 읽기 전용(read-only)
- `작성자만 수정/삭제 가능`	
	Snippet을 만든 사용자만 해당 Snippet 수정/삭제 가능
---
✅ 모델에 사용자 연결
`Snippet` 모델에 `owner`와 `highlighted` 필드 추가: `snippets/models.py`
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

`owner = serializers.ReadOnlyField(source='owner.username')`는
권한 부여를 위해 꼭 필요한 필드입니다.
이글을 누가 썼는지 표시해주는 필드이며 나중에 작성자만 수정.삭제 할수 있습니다.
즉, Snippet을 만든 사용자를 추적할 수 있게 하는 작성자(owner) 필드를 만든 것입니다.

---
`highlighted = models.TextField()` (선택사항)
	이 필드는 렌더링 용도입니다.  
	`Insomnia`나 `API JSON`에서 보면 길고 복잡한 HTML처럼 보이지만,  
	웹 페이지에 붙이면 예쁘게 문법 강조된 코드가 보이게 됩니다.
	`highlighted` 필드는 하이라이트(문법 강조)를 위한 HTML 태그들이 미리 적용된 HTML 코드 덩어리를 저장하는 겁니다.

만약 `code = "result = 5 - 3"`을 저장하면,  
`highlighted`에는 이런 HTML이 생성됩니다:
```html
<div class="highlight">
<pre>
<span class="n">result</span> <span class="o">=</span> <span class="mi">5</span> <span class="o">-</span> <span class="mi">3</span>
</pre>
</div>
```
바로 이 출력은 Django에서 Pygments 라이브러리를 사용해 문법 강조(highlighting)한 결과입니다.  
즉, `highlighted` 필드에 저장된 내용이 바로 이 HTML이며, 그걸 그대로 출력했기 때문에 저렇게 보이는 거예요. `highlighted` 필드는 선택적인 기능입니다.

- 이 필드는 코드를 웹에서 예쁘게 보여주고 싶을 때만 사용하는 용도입니다.
- 만약  파이썬 코드 처리, 백엔드 연산, API 기능 중심의 개발만 한다면,  
    `highlighted` 필드는 필요 없습니다.

---
✅ 마이그레이션 (초기화 방식으로)
```bash
rm -f db.sqlite3
rm -r snippets/migrations
python manage.py makemigrations snippets
python manage.py migrate
python manage.py createsuperuser  # 테스트 사용자 생성
```
초기화를 꼭 해야 하는 이유는 `owner`가 외래키이자 필수 필드이기 때문입니다.  따라서 연습이나 튜토리얼에서는 DB 초기화로 깔끔하게 해결하는 것이 일반적입니다.

---
✅ 사용자 API 뷰 추가: `snippets/serializers.py`
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
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style', 'owner', 'highlighted'] 
    # owner, highlighted두개 필드 추가

# owner 사용자 API뷰 추가
class UserSerializer(serializers.ModelSerializer):
    snippets = serializers.PrimaryKeyRelatedField(many=True, queryset=Snippet.objects.all())

    class Meta:
        model = User
        fields = ['id', 'username', 'snippets']
```
---
✅ 사용자만 본인 소유 Snippet 수정/삭제 가능
`snippets/permissions.py` 생성
```python
from rest_framework import permissions

class IsOwnerOrReadOnly(permissions.BasePermission):
    def has_object_permission(self, request, view, obj):
        if request.method in permissions.SAFE_METHODS:
            return True
        return obj.owner == request.user
```
- 사용자 A는 자신의 글만 수정/삭제하고
- 사용자 B는 A의 글을 읽기만 가능하게 하려면
- 이 권한 클래스가 필요합니다.

---
✅ `views.py`
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
Django REST Framework(DRF)에서 사용하는 뷰별 접근 권한 설정입니다.

---
```python
permission_classes = [permissions.IsAuthenticatedOrReadOnly]
```
- 로그인하지 않은 사용자는 읽기(GET)만 가능
- 로그인한 사용자는 읽기/쓰기/삭제 등 모든 요청 가능

```python
def perform_create(self, serializer):
    serializer.save(owner=self.request.user)
```
- 이 메서드는 DRF에서 `POST` 요청으로 객체를 생성할 때 자동으로 호출됩니다.
- `serializer.save(owner=...)` 을 통해 현재 로그인한 사용자를 Snippet의 `owner` 필드에 저장합니다.

기본적으로 serializer는 JSON에서 입력된 값만 저장합니다. 즉, 이런 요청이 들어면:
```json
{
  "title": "My code",
  "code": "print('hello')"
}
```
저장됩니다.

그런데 `owner = models.ForeignKey(User, ...)` 이것도 필수필드인데 사용자가 json으로 owner를 보내지 않으면 저장할수 없습니다. 

```python
def perform_create(self, serializer):
	serializer.save(owner=self.request.user)
```
이 메서드는 DRF에서 `POST` 요청으로 객체를 생성할 때 자동 호출되며
한 줄 덕분에 사용자가 JSON 요청에 `owner` 값을 입력하지 않아도 현재 로그인한 
사용자가 자동으로 `owner` 필드에 저장됩니다.

```python
permission_classes = [
    permissions.IsAuthenticatedOrReadOnly,IsOwnerOrReadOnly
]
```
기본 동작은 위와 같지만, 작성자(owner) 외에는 수정·삭제가 불가능합니다.

---
✅ `snippets/urls.py`
```python
from django.urls import path
from snippets import views

urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
    
    path('users/', views.UserList.as_view()), # 추가
    path('users/<int:pk>/', views.UserDetail.as_view()),#추가
]
```
---
✅ 사용자 로그인 기능 추가 (Browsable API용)
`tutorial/urls.py` (프로젝트 루트 URLconf)
```python
from django.urls import path, include

urlpatterns = [
    # ...
    path('api-auth/', include('rest_framework.urls')),  
    # 로그인/로그아웃 뷰
]
```

`http://127.0.0.1:8000/api-auth/login/`을 하면 DRF 로그인 창고 연동

![[Pasted image 20250617202542.png]]
로그인과 연결하지 않으면 글 접근 권한이 없습니다.

---
`owner`	
	Snippet 작성자와 연결
`ReadOnlyField`	
	읽기 전용 필드 (request.user.username)
`perform_create()`	
	저장 전 사용자 지정
`IsAuthenticatedOrReadOnly`	
	로그인한 사용자만 쓰기 가능
`IsOwnerOrReadOnly`	
	작성자만 수정/삭제 가능
`/api-auth/login/`	
	로그인 후 브라우저에서도 쓰기 가능


