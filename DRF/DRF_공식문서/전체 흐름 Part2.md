Tutorial 2: Requests and Responses에서는 기존의 `views.py`와 `urls.py`가 다음과 같이 변경됩니다.

기존에는 django.http, JSONParser 사용했지만 이제는 DRF 모듈 사용
`snippets/views.py` : 
```python
from rest_framework import status

# 변경된 데코레이터
from rest_framework.decorators import api_view

# 변경된 응답 객체
from rest_framework.response import Response 
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer


# @csrf_exempt → @api_view(['GET', 'POST'])
@api_view(['GET', 'POST']) 
def snippet_list(request, format=None): # format 파라미터 추가 가능
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data) # JsonResponse → Response

    elif request.method == 'POST':
	    # JSONParser().parse(request) → request.data
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED) # 상태 코드도 DRF 방식
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)


# @csrf_exempt → @api_view([...])
@api_view(['GET', 'PUT', 'DELETE'])
def snippet_detail(request, pk, format=None): 
# format 파라미터 추가 가능

    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return Response(status=status.HTTP_404_NOT_FOUND)
        # HttpResponse → Response

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return Response(serializer.data)

    elif request.method == 'PUT':
		
		# JSONParser().parse(request) → request.data
        serializer = SnippetSerializer(snippet, data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    elif request.method == 'DELETE':
        snippet.delete()
        return Response(status=status.HTTP_204_NO_CONTENT)
        # HttpResponse → Response
```

`snippets/urls.py` (`format_suffix_patterns` 추가)
```python
from django.urls import path
from snippets import views
from rest_framework.urlpatterns import format_suffix_patterns

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

`Imsomnia`
```bash
# 목록 조회
GET http://127.0.0.1:8000/snippets/

# 개별 조회
GET http://127.0.0.1:8000/snippets/1/

# JSON으로 POST
POST http://127.0.0.1:8000/snippets/ code="print('hello')"

# 오류 테스트 POST 요청 보내기 (에러 테스트)
POST http://127.0.0.1:8000/snippets/ code=""
```

###### 필드별 분석
| 필드         | 필수 여부 | 설명                         |
| ---------- | ----- | -------------------------- |
| `title`    | ❌ 선택  | `blank=True`, `default=''` |
| `code`     | ✔️ 필수 | 아무 설정 없으므로 필수              |
| `linenos`  | ❌ 선택  | `default=False`            |
| `language` | ❌ 선택  | `default='python'`         |
| `style`    | ❌ 선택  | `default='friendly'`       |