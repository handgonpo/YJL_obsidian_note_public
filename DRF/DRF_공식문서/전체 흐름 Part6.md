Tutorial 6: ViewSets & Routers에서는 기존의 뷰 클래스들을 `ViewSet`으로 통합하고, URL 구성을 `DefaultRouter`로 자동화하는 방식으로 변경되었습니다.


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


`[test]`
`http://127.0.0.1:8000/snippets/`
`http://127.0.0.1:8000/snippets/1/`
`http://127.0.0.1:8000/users/`
`http://127.0.0.1:8000/users/1/`
`http://127.0.0.1:8000/`
`http://127.0.0.1:8000/snippets/1/highlight/`