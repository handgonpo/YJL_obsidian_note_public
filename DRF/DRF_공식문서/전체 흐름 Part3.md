Tutorial 3: Class-based Views에서는 `views.py`와 `urls.py`만 변경됩니다.

1단계: Function-Based Views (FBV)
```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

@api_view(['GET', 'POST'])
def snippet_list(request, format=None):
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk, format=None):
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```

2단계: Class-Based Views using `APIView`
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.http import Http404
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

class SnippetList(APIView): 
    def get(self, request, format=None): 
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    def post(self, request, format=None):
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

class SnippetDetail(APIView):
    def get_object(self, pk):
        try:
            return Snippet.objects.get(pk=pk)
        except Snippet.DoesNotExist:
            raise Http404

    def get(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    def put(self, request, pk, format=None):
        snippet = self.get_object(pk)
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def delete(self, request, pk, format=None):
        snippet = self.get_object(pk)
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
```
---
3단계: Mixin + GenericAPIView 조합
```python
from rest_framework import mixins, generics
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

class SnippetList(mixins.ListModelMixin, #←여러 개의 기능 클래스 조합
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)

    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)
```

수정된 부분:
- `APIView` → `GenericAPIView`로 변경
- `mixin`을 상속받아 `list()`, `create()` 동작을 분리
- `get()` 안에서 `self.list()` 호출 → 재사용 구조로 개선됨
- 장점: 중복 제거됨. `list()` 등은 DRF가 내부 구현
- 단점: Mixin이 무엇인지 몰라서 ‘자동으로 뭘 하는지’가 안 보임

---
`snippets/views.py`
최종 버전 (Generic Class-Based Views 사용)
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import generics

# 전체 목록 조회 + 생성
class SnippetList(generics.ListCreateAPIView): # ← 단일 클래스로 축약
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

# 단일 조회 + 수정 + 삭제
class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

수정된 부분
- `get()`, `post()` 같은 메서드도 생략됨
- DRF가 미리 만든 `ListCreateAPIView`, `RetrieveUpdateDestroyAPIView` 사용
- 2줄로 CRUD 기능 완성

---
`snippets/urls.py`
```python
from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```
- `views.py`: FBV → APIView → Mixin → Generic CBV 순으로 리팩토링되어 최종적으로 `ListCreateAPIView`, `RetrieveUpdateDestroyAPIView` 사용
- `urls.py`: `.as_view()`를 사용하도록 변경
