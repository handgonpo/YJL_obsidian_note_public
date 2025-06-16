🔹 DRF 튜토리얼 Part 3 _ Class-based Views

📖 공식 문서 링크:  
🔗 [https://www.django-rest-framework.org/tutorial/3-class-based-views/](https://www.django-rest-framework.org/tutorial/3-class-based-views/)

목표:
- `FBV → CBV 전환` 	
	함수형 뷰에서 클래스형 뷰로 전환
- `APIView 사용`	
	DRF에서 제공하는 기본 클래스 기반 뷰
- `Mixin 사용`	
	공통 동작(list, create, retrieve, ...)을 재사용
- `Generic View 사용`	
	DRF가 제공하는 조합된 클래스 기반 뷰로 코드 최소화
---
FBV (Function-Based View): 
	- 우리가 흔히 쓰던 `def` 함수로 만든 뷰
	- 모든 요청(GET, POST 등)을 `if request.method == 'GET'`처럼 조건문으로 나눠서 처리
```python
@api_view(['GET', 'POST'])
def snippet_list(request):
    if request.method == 'GET':
        # 목록 조회
    elif request.method == 'POST':
        # 데이터 생성
```
- ✔️ 장점: 단순하고 초보자가 이해하기 쉬움
- ❌ 단점: 기능이 많아질수록 `if`, `elif`가 많아지고, 코드가 길어짐


CBV (`APIView`) Class-Based View: 
	- `class`로 만든 뷰. 메서드(get, post, put 등)로 HTTP 메서드를 나눔
	- 코드 구조가 명확하고 재사용성이 좋음
```python
class SnippetList(APIView):
    def get(self, request):  # 목록 조회
    def post(self, request):  # 생성
```
- ✔️ 장점: 복잡한 기능을 구조적으로 관리하기 쉬움
- ❌ 단점: 클래스, 메서드 개념에 익숙하지 않으면 처음엔 낯설 수 있음
---
✅ `APIView`를 사용한 기본 클래스형 뷰: `views.py`
```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework import status
from rest_framework.response import Response

from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status
from django.http import Http404

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

🔄 FBV → CBV로 바꾸는 과정 (리팩토링 단계별)
1단계: `APIView` 사용
	가장 기본적인 클래스형 뷰입니다.  
	FBV의 기능을 똑같이 하되, `class`와 `메서드`로 나눠서 처리합니다.
```python
class SnippetList(APIView):
    def get(self, request):
        # 목록 조회
    def post(self, request):
        # 데이터 생성
```
- `APIView`는 우리가 직접 `get`, `post`, `put`, `delete` 메서드를 작성해야 함
- 제어는 자유롭지만 반복 코드가 많아질 수 있음
- 예외 처리도 수동으로 해줘야 함
---
✅ Mixin을 활용한 코드 재사용: 리팩토링(변경) `views.py`
```python
from rest_framework import mixins, generics

class SnippetList(mixins.ListModelMixin,
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

2단계: `GenericAPIView + mixins` 사용
	- 반복되는 작업을 `Mixin`이 대신해줍니다!
	
`Mixin`은 어떤 역할이냐면:
`ListModelMixin` : 전체 목록 조회 (GET /snippets/)
`CreateModelMixin` : 새 데이터 생성 (POST /snippets/)
`RetrieveModelMixin` : 단일 항목 조회 (GET /snippets/1/)
`UpdateModelMixin` : 데이터 수정 (PUT /snippets/1/)
`DestroyModelMixin` : 삭제 (DELETE /snippets/1/)

```python
class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

    def get(self, request):
        return self.list(request)  # Mixin이 대신 실행
```
✔️ 중복 줄이고, 코드도 간결  
❗ 하지만 여전히 `get()`, `post()` 메서드를 수동으로 만들어야 함

---
✅ 완성형 `GenericView`로 최적화
```python
from rest_framework import generics

class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer

class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

3단계: 완성형 `Generic CBV` 사용
	이 단계에선 `DRF`가 미리 조합해 놓은 클래스를 그대로 가져다 씁니다.
```python
class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```
- `ListCreateAPIView`: 목록 보기 + 생성
- `RetrieveUpdateDestroyAPIView`: 단일 조회 + 수정 + 삭제
➡ 코드도 짧고, 거의 모든 기능이 자동으로 처리됨

---
- 1단계: `APIView`	
	모든 메서드를 직접 정의 (가장 상세하게 제어 가능)
- 2단계: `GenericAPIView + Mixins`	
	중복 제거 + 공통 동작은 Mixin에서 가져다 씀
- 3단계: `ListCreateAPIView, RetrieveUpdateDestroyAPIView`	
	가장 간결한 형태. 모든 기능을 자동 처리

1️⃣ 전체 목록 조회 (`GET /snippets/`)
2️⃣ 새 Snippet 생성 (`POST /snippets/`)
3️⃣ 개별 Snippet 조회 (`GET /snippets/<pk>/`)
4️⃣ 개별 Snippet 수정 (`PUT /snippets/<pk>/`)
5️⃣ 개별 Snippet 삭제 (`DELETE /snippets/<pk>/`)
	
이 5개 기능이 점차적으로 간결한 구조로 리팩토링 됩니다.

---
🔹 `1단계: APIView 기반`:   (가장 기본, 수동 구현) 

`전체 목록 조회`
```python
class SnippetList(APIView):
	def get(self, request, format=None):
	    snippets = Snippet.objects.all()
	    serializer = SnippetSerializer(snippets, many=True)
	    return Response(serializer.data)
```
- 직접 DB에서 데이터를 조회하고,
- 직접 직렬화하고,
- 직접 `Response`로 반환

---
`새 Snippet 생성`
```python
def post(self, request, format=None):
	serializer = SnippetSerializer(data=request.data)
	if serializer.is_valid():
		serializer.save()
		return Response(serializer.data,status=status.HTTP_201_CREATED)
	return Response(serializer.errors,status=status.HTTP_400_BAD_REQUEST)
```
- 데이터 검증부터 저장까지 모두 수동
- 예외처리도 직접 작성

`개별 Snippet 조회`
```python
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
```

`수정`
```python
def put(self, request, pk, format=None):
	snippet = self.get_object(pk)
	serializer = SnippetSerializer(snippet, data=request.data)
	if serializer.is_valid():
		serializer.save()
		return Response(serializer.data)
	return Response(serializer.errors,status=status.HTTP_400_BAD_REQUEST)
```

`삭제`
```python
def delete(self, request, pk, format=None):
	snippet = self.get_object(pk)
	snippet.delete()
	return Response(status=status.HTTP_204_NO_CONTENT)
```
---
🔹 `2단계:GenericAPIView + mixins`
	중복 로직 제거 / 기능별 메서드(`list`, `create`, `retrieve`, `update`, `destroy`)를 mixin이 제공함

`list, create`
```python
class SnippetList(mixins.ListModelMixin,
                  mixins.CreateModelMixin,
                  generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

`전체 목록 조회`
```python
def get(self, request, *args, **kwargs):
    return self.list(request, *args, **kwargs)
```

`새 Snippet 생성`
```python
def post(self, request, *args, **kwargs):
    return self.create(request, *args, **kwargs)
```
---
`retrieve, update, destroy`
```python
class SnippetDetail(mixins.RetrieveModelMixin,
                    mixins.UpdateModelMixin,
                    mixins.DestroyModelMixin,
                    generics.GenericAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

`개별 Snippet 조회`
```python
def get(self, request, *args, **kwargs):
    return self.retrieve(request, *args, **kwargs)
```

`수정`
```python
def put(self, request, *args, **kwargs):
    return self.update(request, *args, **kwargs)
```

`삭제`
```python
def delete(self, request, *args, **kwargs):
    return self.destroy(request, *args, **kwargs)
```

---
🔹 `3단계:Generic CBV (가장 간단한 버전)`

`전체 목록 조회 + 생성`
```python
class SnippetList(generics.ListCreateAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

`단일 조회 + 수정 + 삭제`
```python
class SnippetDetail(generics.RetrieveUpdateDestroyAPIView):
    queryset = Snippet.objects.all()
    serializer_class = SnippetSerializer
```

---
`CBV(Class-Based View)` : 
	기존에는 함수를 사용해서 `if request.method == 'GET'` 처럼 처리했다면,
	이제는 클래스로 뷰를 만들고, 그 안에 `get()`, `post()` 같은 메서드를 써서  
    요청 종류에 따라 따로 처리할 수 있어요.
    
`APIView`	: 
	DRF에서 제공하는 가장 기본적인 클래스 기반 뷰예요.
	직접 `get()`, `post()` 같은 메서드를 작성해서 처리할 수 있어요.
	함수형 뷰보다 구조가 더 깔끔해지고 확장도 쉬워요.
	
`Mixin`	: 
	`목록 조회`, `생성`, `수정`, `삭제` 같은 기능을 미리 만들어진 코드 조각으로 제공해줘요. 이걸 클래스에 상속해서 갖다 쓰면, 직접 구현하지 않아도 돼요.
	
`GenericView`	: 
	mixin 기능이 이미 조합된 클래스예요.
	거의 모든 CRUD 기능이 자동으로 들어가 있고,  
    2~3줄이면 기본 API를 만들 수 있어요.

✅ URL 연결: `snippets/urls.py`
```python
from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

# 기존
urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]

# 변경
urlpatterns = [
    path('snippets/', views.SnippetList.as_view()),
    path('snippets/<int:pk>/', views.SnippetDetail.as_view()),
]

urlpatterns = format_suffix_patterns(urlpatterns)
```

