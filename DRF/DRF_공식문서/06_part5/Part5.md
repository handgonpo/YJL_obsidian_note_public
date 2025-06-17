🔹 DRF 튜토리얼 Part5 _ Relationships & Hyperlinked APIs

📖 공식 문서 링크:  
🔗 [https://www.django-rest-framework.org/tutorial/5-relationships-and-hyperlinked-apis/](https://www.django-rest-framework.org/tutorial/5-relationships-and-hyperlinked-apis/)

목표:
- `id 대신 url로 관계 표현`	
	PK 대신 URL 하이퍼링크 사용으로 관계 가독성 향상
- `API 루트 엔드포인트 생성`	
	모든 API 입구를 하나의 /에서 보여줌
- `하이라이트된 HTML 제공`	
	코드 스니펫을 보기 좋게 HTML로 표시
- `URL에 이름(name=) 지정`	
	DRF가 내부적으로 URL 역참조할 수 있도록
- `결과 목록에 페이지네이션` 
	적용	한 페이지에 10개씩 제한하여 효율적인 조회 지원

---
✅ API 루트 만들기 (`/`)
`snippets/views.py`에 추가
```python
from rest_framework.decorators import api_view
from rest_framework.response import Response
from rest_framework.reverse import reverse
from rest_framework import renderers

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
→ API의 홈(/)에 접속하면 `users`, `snippets` 링크 제공

---
✅ 하이퍼링크 기반 Serializer로 변경
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
---
✅ URL 패턴 이름 지정 및 하이라이트 경로 추가
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
각 URL에 `name='...'`을 추가해야 **reverse()나 HyperlinkedSerializer**가 제대로 작동합니다.

---
✅ 페이지네이션 적용
`settings.py`
```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10,
    'DEFAULT_PERMISSION_CLASSES': [
    'rest_framework.permissions.AllowAny',
  ]
}
```
→ `/snippets/`, `/users/`에서 페이지가 나뉘어 표시됩니다.

---
- `HyperlinkedModelSerializer`	
	id 대신 url로 관계를 표현하는 serializer
- `reverse()`	
	URL name으로 실제 경로를 생성해주는 도구
- `highlight 필드`	
	HTML 코드 하이라이트를 위한 전용 링크
- `url, highlight, snippets`	
	모두 링크로 관계를 명시함 (직관적이고 RESTful)
- `format_suffix_patterns()`	
	.json, .api, .html 같은 포맷 지정 가능
- `페이지네이션`	
	API 호출 시 결과를 나눠서 보여주는 방식
