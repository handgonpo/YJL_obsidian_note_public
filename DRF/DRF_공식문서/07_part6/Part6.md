🔹 DRF 튜토리얼 Part6 _ ViewSets & Routers

📖 공식 문서 링크:  
🔗 [https://www.django-rest-framework.org/tutorial/6-viewsets-and-routers/](https://www.django-rest-framework.org/tutorial/6-viewsets-and-routers/)

목표:
- `ViewSet 사용`	
	CRUD 기능을 하나의 클래스에 통합
- `@action 사용`	
	커스텀 API(예: highlight) 만들기
- `Router로 URL 자동화`	
	urlpatterns 수동 작성 없이 자동 처리
- `API Root 자동 생성`	
	/ 경로를 자동으로 만들어 줌
- `코드량 최소화`	
	클래스 수와 URL 작성 수를 줄이고 API 구조를 단순화
---
`UserViewSet`, `SnippetViewSet` 추가 / 기존 뷰들 제거 (`SnippetList`, `SnippetDetail`, `SnippetHighlight`, `UserList`, `UserDetail`, `api_root`)

`snippets/views.py`
```python
from rest_framework import viewsets, permissions, renderers
from rest_framework.decorators import action
from rest_framework.response import Response
from django.contrib.auth.models import User
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer, UserSerializer
from snippets.permissions import IsOwnerOrReadOnly


class UserViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = User.objects.all()
    serializer_class = UserSerializer


class SnippetViewSet(viewsets.ModelViewSet):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly, IsOwnerOrReadOnly]

    @action(detail=True, renderer_classes=[renderers.StaticHTMLRenderer])
    def highlight(self, request, *args, **kwargs):
        snippet = self.get_object()
        return Response(snippet.highlighted)

    def perform_create(self, serializer):
        serializer.save(owner=self.request.user)
```
---
기존 뷰 분기 방식 제거 → Router 기반 URL 자동 구성으로 변경

`snippets/urls.py`
```python
from snippets import views

from django.urls import path, include
from rest_framework.routers import DefaultRouter

router = DefaultRouter()
router.register(r'snippets', views.SnippetViewSet, basename='snippet')
router.register(r'users', views.UserViewSet, basename='user')

urlpatterns = [
    path('', include(router.urls)),
]
```
기존에 수동으로 정의했던 `path('snippets/...')`, `path('users/...')`, `api_root` 등은 모두 제거됨

`ViewSet`은 일반 뷰 클래스처럼 직접 `path()`로 URL을 지정하지 않습니다.
대신 `router`가 자동으로 GET, POST, PUT/PATCH, DELETE를 만들어 줍니다.

