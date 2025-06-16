🔹 DRF 튜토리얼 Part 2 _ Requests and Responsesponses

📖 공식 문서 링크:  
🔗 [https://www.django-rest-framework.org/tutorial/2-requests-and-responses/](https://www.django-rest-framework.org/tutorial/2-requests-and-responses/)

목표:
- `request.data 사용`	
	request.POST보다 더 유연하게 JSON, form, multipart 데이터를 처리
- `Response 객체 사용`	
	API 응답을 클라이언트가 요청한 타입(JSON, HTML 등)으로 자동 반환
- `status 코드 사용`	
	201, 404 같은 숫자 대신 status.HTTP_201_CREATED 같은 식별자 사용
- `@api_view 데코레이터`	
	함수 기반 뷰를 REST API 스타일로 변환 (CSRF 자동 처리 등 포함)
- `URL에 .json, .api 같은 포맷 접미사 지원`	
	다양한 응답 포맷에 대응하도록 URL 패턴 확장

---
표준 Django 기능에서 DRF 기능으로 전환하는 과정:
`JsonResponse` → `Response`
	`JsonResponse: `
	`Django 기본 기능으로, JSON 형식으로 응답을 보낼 때 사용하며 딱 JSON만 보내는 역할입니다.`
	
	Response (DRF):
	DRF에서 제공하는 응답 클래스이며, JSON은 물론, HTML, XML 등 다양한 
	포맷 지원이 가능합니다. 상태 코드도 더 쉽게 처리할 수 있어요.

왜 바꾸나?
	`Response는 더 강력하고 유연합니다. 예를 들어 `
	`Response(data, status=201)처럼 상태코드도 명확하게 넣을 수 있고,` 
	`나중에 HTML, XML로 포맷을 바꿔야 할 때도 자동 처리됩니다.`

---
보일러플레이트 코드란?
	`매번 반복해서 똑같이 써야 하는 코드. 재미없고 귀찮은 코드를 제거합니다.`

코드 예시 (기존 방식 - 보일러플레이트 많음):
```python
@csrf_exempt
def snippet_list(request):
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)
```

바꾼 코드 (DRF 방식 - 보일러플레이트 제거):
```python
@api_view(['GET'])
def snippet_list(request):
    snippets = Snippet.objects.all()
    serializer = SnippetSerializer(snippets, many=True)
    return Response(serializer.data)
```
여기서 `csrf_exempt`, `JsonResponse`, `JSONParser().parse()` 등을 다 없앴음.  
→ 똑같은 기능인데 코드는 더 짧고 보기 좋아졌습니다.

리팩토링이란?
	기능은 똑같이 유지하면서 코드를 더 보기 좋고 간단하게 정리하는 것

---
응답 클래스와 상태 코드 상수 사용:
	`예전 방식:` `return JsonResponse(data, status=400)`
	`개선된 방식 (DRF):`
	`return Response(data, status=status.HTTP_400_BAD_REQUEST) `
	
	숫자 400, 201 같은 건 HTTP 응답 코드입니다. 예전에는 숫자만 써서 
	의미를 외워야 했지만, 지금은 status.HTTP_400_BAD_REQUEST처럼 읽기 
	쉬운 이름으로 바꿨어요. 이렇게 하면 에러코드가 뭔지 한눈에 알 수 있어
	서 코드가 더 이해하기 쉬워요.
	
---
✅ `views.py` 수정
```python
from rest_framework import status
from rest_framework.decorators import api_view
from rest_framework.response import Response
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

# @csrf_exempt → @api_view(['GET', 'POST'])
@api_view(['GET', 'POST'])
def snippet_list(request, format=None):
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
---
`views.py` 코드를 변경한 핵심 이유
	`1. 더 명확하고 직관적인 API 코드 작성:`
		- `DRF는 API 전용 툴킷이라 API 작성에 최적화된 기능을 제공합니다.`
	    - `request.data`, `Response`, `status.HTTP_400_BAD_REQUEST` `등으로 의도가 명확해집니다.`
	    - ❌ `기존 방식: Django의 일반 뷰 + JSON 수동 처리`
	    - ✔️ `변경 방식: DRF의 REST 전용 기능 활용 (더 짧고, 안전하고, 
	        `확장성 있음)
	    
`수작업을 자동화한 것처럼 더 쉬워짐!`
```python
# 변경 전: JSON 직접 파싱하고 처리해야 함 (복잡함)
data = JSONParser().parse(request)

# 변경 후: DRF는 한 줄이면 끝!
data = request.data
```
	
	2. CSRF 처리 자동화 & 보안 향상: 보안 처리가 자동으로 됨 (CSRF)
		- ❌ 예전 방식은 @csrf_exempt를 써야 해서 보안이 뚫릴 수 있음
		- ✔️ DRF 방식은 API 요청만 자동으로 허용해 줘서 더 안전해요.
		
`보안 걱정을 덜어주는 자동문 같음!`
```python
# 변경 전: 수동으로 보안 해제 (위험)
@csrf_exempt

# 변경 후: DRF가 알아서 처리
@api_view(['GET', 'POST'])
```
	
	3. 응답 객체 JsonResponse → Response(응답 처리 도구가 더 똑똑함)
		- Response는 단순 JSON뿐만 아니라 XML 등도 자동으로 처리할 수 
		있어요. 여러 형식을 지원할 수 있어서 더 유연합니다.
```python
return Response(serializer.data)  # 다양한 포맷 자동 대응
```
	
	4. 상태 코드를 더 보기 쉽게 쓸 수 있음.
		-기존:status=201 → 변경: status=status.HTTP_201_CREATED
		-기존:status=400 → 변경: status=status.HTTP_400_BAD_REQUEST
		- 숫자만 보면 무슨 의미인지 헷갈릴 수 있지만 DRF는 이름으로 의미
		를 알려줍니다.
```python
# 보기 불편한 방식
return Response(data, status=400)

# 보기 쉬운 방식 (추천)
return Response(data, status=status.HTTP_400_BAD_REQUEST)
```

	5. 향후 확장성: CBV, 인증, 필터, 페이지네이션 등
		- DRF 방식은 나중에 APIView, GenericAPIView, ViewSet 등 고급 
		기능으로 점점 레벨업이 가능합니다.
		- 인증/권한, 필터링, 검색, 페이지네이션도 간편하게 추가 가능함.

---
`데코레이터 변경`
```python
- @csrf_exempt
+ @api_view(['GET', 'POST'])
```
- `@csrf_exempt`는 Django 기본 뷰에서 CSRF 체크만 비활성화함
- `@api_view([...])`는 DRF가 제공하는 함수형 API 뷰 데코레이터로,
    - CSRF 자동 비활성화
    - 요청/응답 자동 파싱
    - 브라우저에서 Browsable API 제공
---
`요청 데이터 처리 방식`
```python
- data = JSONParser().parse(request)
+ serializer = SnippetSerializer(data=request.data)
```
- `JSONParser().parse(request)`는 수동으로 JSON을 파싱해야 함
- `request.data`는 DRF가 자동으로 처리해줌 (더 안전하고 간결)
---
`응답 반환 방식`
```python
- return JsonResponse(serializer.data, status=201)
+ return Response(serializer.data, status=status.HTTP_201_CREATED)
```
- `JsonResponse`는 Django 내장 JSON 응답
- `Response`는 DRF의 응답 객체로:
    - 다양한 포맷 지원(JSON, HTML 등)
    - 렌더링 처리 내장
    - 자동 상태 코드 처리
---
`HTTP 상태 코드 처리`
```python
- status=201
+ status=status.HTTP_201_CREATED
```
- 정수 값 대신 DRF에서 정의한 명확한 상수 이름 사용
- 가독성과 유지보수성이 뛰어남
---
✅ `urls.py` 수정 (`format_suffix_patterns` 추가)
```python
from django.urls import path
from rest_framework.urlpatterns import format_suffix_patterns
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]

urlpatterns = format_suffix_patterns(urlpatterns) <- 추가
```

`format_suffix_patterns()` 이란?
	URL 뒤에 `.json`, `.html` 같은 확장자를 붙이면,  DRF가 자동으로 그에 맞는 응답 형식으로 바꿔주는 기능이에요.
	
❓  왜 쓰는가?
	브라우저 테스트 시 유용함 : 브라우저는 기본적으로 이렇게 요청해요
```
Accept: text/html
```

그래서 `/snippets/`를 요청하면 브라우저가 보기 좋은 HTML 인터페이스(DRF Browsable API) 가 떠요.
하지만 내가 진짜 JSON 결과만 보고 싶을 때는 어떻게 할까요?
👉 URL 뒤에 `.json`만 붙이면 됩니다.
```
http://127.0.0.1:8000/snippets.json
```
이렇게 하면 DRF가 무조건 JSON만 보여줘요.

```python
GET /snippets/           → 기본: JSON
GET /snippets.json       → JSON 
GET /snippets.api        → JSON (alias)
GET /snippets.html       → HTML browsable API
```

실제로 URL 끝에 `.json`을 붙이는 경우:
- 브라우저에서 직접 테스트할 때
```bash
http://127.0.0.1:8000/snippets.json
```
- 브라우저는 기본적으로 `Accept: text/html`을 보냅니다.
- `.json` 확장자를 붙이면 DRF가 강제로 JSON으로 렌더링하게 함 → 브라우저에서 구조 확인 가능

`urlpatterns = format_suffix_patterns(urlpatterns)` 없을때
![[Pasted image 20250608173152.png]]

`urlpatterns = format_suffix_patterns(urlpatterns)` 있을때
![[Pasted image 20250608173245.png]]

`format_suffix_patterns()`는 Django REST framework(DRF)에서 URL에 `.json`, `.api`, `.html` 등의 포맷 접미사(suffix)를 붙여서 다양한 응답 포맷을 요청할 수 있게 해주는 기능입니다.

![[Pasted image 20250615225353.png]]
이 코드를 추가하면, 다음처럼 URL 끝에 포맷을 명시할 수 있습니다:

Insomnia에는 기본 설정되어 있으나 삭제가능 합니다.
![[Pasted image 20250608173357.png]]

API 요청 테스트 (GET, POST, PUT, DELETE 등)
```
GET     http://localhost:8000/snippets/
POST    http://localhost:8000/snippets/
GET     http://localhost:8000/snippets/3/
PUT     http://localhost:8000/snippets/3/
DELETE  http://localhost:8000/snippets/3/
```

포맷 명시 요청 가능
```
GET http://localhost:8000/snippets.json
GET http://localhost:8000/snippets.api
```
→ `.json`, `.api` 붙여도 잘 동작하는지 확인 가능  
→ 브라우저에서도 직접 주소창에 입력해 확인 가능

REST API에서 사용자가 응답 포맷을 명시하는 URL 접미사(suffix)
`.json` 
- 클라이언트가 **JSON 형식**의 응답을 원한다는 뜻
- 서버는 `Content-Type: application/json`으로 응답
`GET /todos.json`뜻
- 이 요청은 `"나는 JSON으로 응답받고 싶어"`라고 말하는 것과 같음

`.api`
- `.api`는 특별한 포맷이 아니라 DRF에서 기본적으로 JSON으로 처리되도록 매핑된 형식
- 실질적으로 `.json`과 동일하게 작동합니다

즉, 이 접미사는 서버에 요청하는 방법 중 하나입니다. 서버는 이를 보고 내부적으로 
적절한 렌더러(`JSONRenderer`, `HTMLRenderer` 등)를 선택합니다.

✨ 활용예시:
실거래가 데이터를 Django 백엔드 + DRF + 프론트엔드로 활용하는 전체 흐름

✅ 1단계. 먼저 부동산 거래 데이터를 파일로 준비합니다.
	- `real_estate.csv` (엑셀처럼 생긴 표 파일)
	- `real_estate.json` (JSON 형식)
	- `국토부_데이터.xlsx` (엑셀파일)
```
지역       | 가격        | 계약일
-----------|----------- |---------
서울 강남구 | 15억        | 2024-01-15
```


✅ 2단계. Django 서버에 데이터 저장
	`방법 1: 관리자 페이지 or 커맨드로 직접 넣기`
```bash
python manage.py import_real_estate_data real_estate.csv
```
- 서버 개발자가 직접 파일을 읽어서 데이터베이스에 넣는 명령어 실행

	`방법 2: API로 파일 업로드`
```
POST /api/real_estates/upload/  ← Insomnia로 테스트
```
	- Insomnia나 프론트에서 파일 업로드
	- 서버가 이 파일을 받아서 DB에 저장함
	
✅ 3단계. 저장된 데이터를 API로 제공 (DRF)
이제 데이터를 저장했으니,  외부에서 조회할 수 있도록 API를 열어줘야 하죠.
```
GET /api/real_estates/  → 전체 거래 목록
GET /api/real_estates/2024/  → 연도별 거래
GET /api/real_estates/서울/  → 지역별 거래
```

이 주소들에서 `.json`, `.api`를 붙이면:
```
GET /api/real_estates.json
GET /api/real_estates/2024.api
```

서버는 JSON으로 응답합니다:
```json
[
  {
    "id": 1,
    "region": "서울 강남구",
    "price": 1500000000,
    "contract_date": "2024-01-15",
    ...
  }
]
```
브라우저는 기본적으로 HTML을 요청하려 하기 때문에,  
`.json`을 붙이면 DRF가 "JSON으로 줘야겠군요!" 하고 응답을 바꿔줍니다.

✅ 4단계. 프론트엔드에서 요청하기
```js
axios.get("/api/real_estates.json")
  .then(res => {
    const data = res.data;
    drawChart(data);  // 대시보드 차트로 사용
  });
```
이 요청으로 서버에서 JSON을 받아서 대시보드에 그래프/표로 보여주는 거예요

✅ 5단계.  `.json`, `.api`는 왜 유용한가?
브라우저에서 직접 URL 확인할때 : `.json` 붙이면 무조건 JSON 응답 확인 가능
프론트에서 데이터 가져올 때: 응답 포맷이 예측 가능해서 버그 줄어듦
테스트할 때 (Insomnia 등): 확장자를 붙이면 응답 포맷을 확실히 고정시킬 수 있음
문서화할 때: 주소만 봐도 어떤 형식으로 응답이 오는지 알 수 있음

```
CSV/JSON 파일 → Django DB 저장 → DRF API로 공개 → 
프론트가 요청해서 차트로 표시
```
그리고 `.json` 같은 확장자를 붙이면 서버가 정확한 형식으로 응답해서  
브라우저, Insomnia, 프론트엔드 모두 예측 가능하고 안전하게 데이터를 쓸 수 있어요!

JSON은 기본적으로 "키-값" 쌍을 가진 딕셔너리 구조입니다.  
→ 자바스크립트에서 다루기 아주 쉬운 구조예요.
```json
[
  {
    "region": "서울 강남구",
    "price": 1500000000,
    "contract_date": "2024-01-15"
  },
  {
    "region": "서울 마포구",
    "price": 980000000,
    "contract_date": "2024-01-10"
  }
]
```

</>예시  이 데이터를 자바스크립트로 가져오면 바로 이렇게 쓸 수 있어요:
```js
fetch("/api/real_estates.json")
  .then(res => res.json())
  .then(data => {
    console.log(data[0].region);  // "서울 강남구"
    console.log(data[1].price);   // 980000000
  });
```

</>예시 차트 그리기 (Chart.js)
```js
const labels = data.map(item => item.contract_date);
const values = data.map(item => item.price);

new Chart(ctx, {
  type: 'line',
  data: {
    labels: labels,
    datasets: [{
      label: '실거래가',
      data: values
    }]
  }
});
```

</>예시 테이블 그리기;
```js
data.forEach(item => {
  const row = `
    <tr>
      <td>${item.region}</td>
      <td>${item.price}</td>
      <td>${item.contract_date}</td>
    </tr>
  `;
  document.querySelector("#dataTable").innerHTML += row;
});
```

`settings.py` 수정
```python
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.AllowAny',
    ]
}
```

`'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'`
`'rest_framework.permissions.AllowAny'` 의 차이점

| 항목                                | `AllowAny`                | `DjangoModelPermissionsOrAnonReadOnly`               |
| --------------------------------- | ------------------------- | ---------------------------------------------------- |
| 기본 의미                             | 모든 사용자 허용                 | 인증된 사용자는 Django 모델 <br>권한에 따라 허용, 익명 사용자는 <br>읽기만 허용 |
| 인증 필요 여부                          | ❌ 없음                      | ✔️ 인증 필요 (쓰기 시)                                      |
| 익명 유저 <br>(비로그인)                . | ✔️ 모든 요청 가능 (GET, POST 등) | ✔️ 읽기만 가능 (GET), <br>❌ POST/PUT/DELETE 불가            |
| 권한 검사                             | ❌ 하지 않음                   | ✔️ `user.has_perm(...)` 체크                           |
| 적합한 <br>상황                        | 테스트용, <br>공개 API          | 관리자 기능, 실제 사용자 관리가 <br>필요한 API                       |

- `AllowAny`
	아무나 접근 가능. 로그인 안 해도 되고, 권한 없어도 됨.
	개발 중에 간단히 API를 테스트할 때 유용
    실전에서는 보안상 위험할 수 있음

- `DjangoModelPermissionsOrAnonReadOnly`
	로그인한 사용자: Django 권한(permission)에 따라 허용  
	로그인 안 한 사용자: 읽기(GET)만 허용, 쓰기 금지
	
---
✅ 테스트 방법: 
브라우저
- [http://127.0.0.1:8000/snippets/](http://127.0.0.1:8000/snippets/) → HTML 브라우저 전용 API 렌더링
- [http://127.0.0.1:8000/snippets.json](http://127.0.0.1:8000/snippets.json) → JSON으로 응답

httpie 명령어
```bash
# 목록 조회
http GET http://127.0.0.1:8000/snippets/

# 개별 조회
http GET http://127.0.0.1:8000/snippets/1/

# JSON으로 POST
http POST http://127.0.0.1:8000/snippets/ code="print('hello')"

# 오류 테스트
http POST http://127.0.0.1:8000/snippets/ code=""
```

`POST 요청 보내기(정상 데이터)`
```json
{
  "title": "비어 있는 코드 예제",
  "code": "",  // ← 공백 허용 안함으로 오류 발생
  "linenos": true,
  "language": "python",
  "style": "friendly"
}

{
  "title": "Hello World",
  "code": "print('Hello, world!')",
  "linenos": true,
  "language": "python",
  "style": "friendly"
}

{
  "title": "JS Example",
  "code": "console.log('Hello JS');",
  "linenos": false,
  "language": "javascript",
  "style": "monokai"
}

```
![[Pasted image 20250608165731.png]]
`title`, `language`, `style`, `linenos` 필드들이 모델에서 필수값(required)이 아니도록 설정되어 있기 때문에,  `code` 하나만 보내도 `.is_valid()`가 `True`가 되고 `.save()`가 성공하게 됩니다.

`모델 필드 정의`
```python
class Snippet(models.Model):
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)
```

###### 필드별 분석
| 필드         | 필수 여부 | 설명                         |
| ---------- | ----- | -------------------------- |
| `title`    | ❌ 선택  | `blank=True`, `default=''` |
| `code`     | ✔️ 필수 | 아무 설정 없으므로 필수              |
| `linenos`  | ❌ 선택  | `default=False`            |
| `language` | ❌ 선택  | `default='python'`         |
| `style`    | ❌ 선택  | `default='friendly'`       |

---
POST 요청 보내기 (에러 테스트)
JSON을 이렇게 비워보세요:
```json
{
  "code": ""
}
```
![[Pasted image 20250608165955.png]]
- 이때 `code` 필드는 `required=True` 또는 `blank=False`이면 400 Bad Request 오류가 발생합니다.
- 예: `"이 필드는 blank일 수 없습니다"` 또는 `"이 필드는 필수입니다"` 등의 응답이 나옵니다.