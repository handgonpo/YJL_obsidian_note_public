API 루트 View 추가, 하이라이트 View 추가
`snippets/views.py`
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework.reverse import reverse
from rest_framework import renderers

# ... 기존 필드 생략 ...

# API의 홈(/)에 접속하면 users, snippets 링크 제공
@api_view(['GET'])
def api_root(request, format=None):
    return Response({
        'users': reverse('user-list', request=request, format=format),
        'snippets': reverse('snippet-list', request=request, format=format)
    })

# HTML 하이라이트 뷰 만들기
class SnippetHighlight(generics.GenericAPIView):
    queryset = Snippet.objects.all()
    renderer_classes = [renderers.StaticHTMLRenderer]

    def get(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)
```

`api_root` 함수
`/` (API 홈 주소)에 접속하면,  사용자 목록(`/users/`)과 코드 목록(`/snippets/`) 링크를 보여줍니다.

왜 필요하냐면?  
DRF에서 브라우저로 API를 탐색할 때,  
처음 화면에 링크를 보여줘야 어디로 가야 할지 알 수 있기 때문입니다.  
→ API의 홈 메뉴판 같은 역할이에요.

---
`SnippetHighlight` 클래스
코드 조각을 HTML로 예쁘게 하이라이팅된 형태로 보여주는 API입니다.
왜 필요하냐면?  
일반 JSON으로 코드를 보는 것보다,  
하이라이팅된 HTML로 보여주면 가독성이 훨씬 좋아서 사용자 경험이 좋아집니다.
![[Pasted image 20250617215037.png|400]]

---
`HyperlinkedModelSerializer`로 변경, 하이라이트 필드 및 URL 필드 추가
`snippets/serializers.py`
```python
from rest_framework import serializers
from snippets.models import Snippet
from django.contrib.auth.models import User

class SnippetSerializer(serializers.HyperlinkedModelSerializer):
    owner = serializers.ReadOnlyField(source='owner.username')
    highlight = serializers.HyperlinkedIdentityField(view_name='snippet-highlight', format='html')

    class Meta:
        model = Snippet
        fields = ['url', 'id', 'highlight', 'owner', 'title', 'code', 'linenos', 'language', 'style']


class UserSerializer(serializers.HyperlinkedModelSerializer):
    snippets = serializers.HyperlinkedRelatedField(
        many=True,
        view_name='snippet-detail',
        read_only=True
    )

    class Meta:
        model = User
        fields = ['url', 'id', 'username', 'snippets']
```
```python
class SnippetSerializer(serializers.HyperlinkedModelSerializer):
```

	이 시리얼라이저는 `ModelSerializer`의 확장 버전인
	HyperlinkedModelSerializer 
	일반 필드뿐 아니라, 링크(URL) 도 포함해서 보여줍니다.

![[Pasted image 20250617215323.png|400]]

```python
    owner = serializers.ReadOnlyField(source='owner.username')
```
	owner 필드는 읽기 전용입니다.
	Snippet 객체에 연결된 User 객체의 username만 표시됨
	사용자가 직접 owner를 입력하지 않아도 됨

```python
    highlight = serializers.HyperlinkedIdentityField(
        view_name='snippet-highlight',
        format='html'
    )
```
	이 필드는 하이라이트된 코드 보기 링크
	/snippets/<id>/highlight/ 와 같은 URL로 연결됨
	view_name='snippet-highlight' 뷰를 호출함

![[Pasted image 20250617220550.png|400]]

```python
    class Meta:
        model = Snippet
        fields = ['url', 'id', 'highlight', 'owner', 'title', 'code', 'linenos', 'language', 'style']
```
	어떤 모델을 직렬화할지 지정 (`Snippet`)  
	어떤 필드들을 포함할지 나열  
	url 필드가 자동 생성되어 각 Snippet의 상세 링크 제공됨  
	highlight는 HTML 보기 전용 링크  
	owner는 작성자 username  
	나머지는 코드의 실제 내용들 (제목, 내용 등)

---
```python
class UserSerializer(serializers.HyperlinkedModelSerializer):
```
	사용자(`User`) 객체를 직렬화할 때 링크를 포함해서 보여주는 시리얼라이저

```python
    snippets = serializers.HyperlinkedRelatedField(
        many=True,
        view_name='snippet-detail',
        read_only=True
    )
```
	사용자가 작성한 Snippet 목록  
	여러 개(many=True)  
	각 Snippet은 링크 형태(/snippets/<id>/)로 제공  
	read_only=True → 사용자 정보 입력 시 여기에 직접 값을 넣지 않음

```python
    class Meta:
        model = User
        fields = ['url', 'id', 'username', 'snippets']
```
	User : 모델을 직렬화
	url : 이 사용자의 상세 URL
	id : 고유 번호
	username : 사용자 이름
	snippets : 이 사용자가 만든 Snippet들의 링크 목록

----
모든 URL에 이름(`name=...`) 추가, API 루트 및 하이라이트 경로 추가
`snippets/urls.py`
```python
from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = format_suffix_patterns([
    path('snippets/', views.SnippetList.as_view(), name='snippet-list'),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view(), name='snippet-detail'),
    path('users/', views.UserList.as_view(), name='user-list'),
    path('users/<int:pk>/', views.UserDetail.as_view(), name='user-detail'),
	
	path('', views.api_root),
    path('snippets/<int:pk>/highlight/', views.SnippetHighlight.as_view(), name='snippet-highlight'),
])
```

---
`tutorial/settings.py` 페이지네이션 설정 추가
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    'DEFAULT_PERMISSION_CLASSES': [
    'rest_framework.permissions.AllowAny',
  ]
}
```
	part6을 위한 페이지 네이션 미리 설정