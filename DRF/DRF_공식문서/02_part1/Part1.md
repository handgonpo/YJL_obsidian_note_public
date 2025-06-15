🔹 DRF 튜토리얼 Part 1 _ Serialization

📖 공식 문서 링크:  
🔗 [https://www.django-rest-framework.org/tutorial/1-serialization/](https://www.django-rest-framework.org/tutorial/1-serialization/)

목표:
- `직렬화(Serialization)`	
	Django 모델 데이터를 JSON 형식으로 변환
- `역직렬화(Deserialization)`	
	JSON을 Django 모델로 저장
- `Serializer`
	Django Form과 비슷한 역할(입력 검증, 필드변환)
- `ModelSerializer` 
	forms.py와 유사한 개념으로 모델기반 자동생성
- `실제 API를 위한 기초 뷰 함수 작성`	
	API에서 GET/POST/PUT/DELETE 처리
- `웹 브라우저 또는 httpie로 테스트`	
	만든 API를 직접 호출하여 확인

`기존 Django 방식:`
```css
사용자가 폼(form) 작성 → 사용자가 입력 → 서버에 POST → 
Form.is_valid()유효성검사 → 서버에 저장
```

`DRF + axios 방식:`
```css
사용자가 프론트에서 입력 → axios로 JSON 전송 → 
serializer.is_valid()유효성검사 → 서버에 저장

단지 <form>을 사용하지 않습니다. 여전히 input, textarea,button같은 HTML 입력 요소는 사용합니다.
```

`<form>` 태그: 
	DFW: HTML 폼을 서버로 전송 submit
	DRF: 사용하지 않고 axios로 대체
`<input>`, `<textarea>`, `<select>`
	똑같이 사용
`.is_valid()유효성검사`
	DFW: 서버에서 검사
	DRF: Serializer가 처리
`전송방식`
	DFW: 사용자가 버튼 클릭 시 브라우저가 자동으로 `<form>` 데이터를 
	HTTP POST로 서버에 보냄
	DRF: `axios.post()`는 자바스크립트가 직접 서버에 JSON을 전송
	
---
🔹 Django Form 기반 CBV (DFW 방식)

`froms.py`
```python
class TodoForm(forms.ModelForm):
    class Meta:
        model = Todo
        fields = ['title', 'completed'...]
```

```python
class TodoCreateView(generics.CreateView):
    model = Todo
    form_class = TodoForm
    template_name = 'todo/create.html'  
    success_url = reverse_lazy('todo:list')  
```
---
🔹 DRF + ModelSerializer 기반 CBV (API 방식)

`ModelSerializer` (API용)
```python
class TodoSerializer(serializers.ModelSerializer):
    class Meta:
        model = Todo
        fields = ['id', 'title', 'completed'...]
```

`views.py (CBV 예시)`
```python
class TodoListCreateAPIView(generics.ListCreateAPIView):
    queryset = Todo.objects.all()
    serializer_class = TodoSerializer
```
ModelSerializer 는 ModelForm의 API 버전 이라고 기억하면 편해요.

---
Django REST Framework(DRF)의 직렬화(Serialization)는 파이썬 객체(특히 Django 모델 인스턴스)를 JSON과 같은 네트워크 전송 가능한 형식으로 변환하거나, 그 반대로 JSON 데이터를 파이썬 객체로 역직렬화(Deserialization)하는 과정입니다.

📖 용어설명: 
- `직렬화 (Serialization)`	
	Python 객체 → JSON 등으로 변환
- `역직렬화 (Deserialization)`	
	JSON → Python 객체 (모델 인스턴스)로 복원
- `Serializer 클래스`	
	폼(Form)과 유사하게 필드 정의 및 검증 제공
- `ModelSerializer`	
	Django 모델과 자동 연결되는 Serializer (직렬화/역직렬화 자동 처리)

❓ 이런 과정이 왜 필요한가요?
- 웹 API는 데이터를 JSON 형식으로 주고받습니다.
- Django는 내부적으로 Python 객체를 사용합니다.
- 이 둘을 연결하기 위해 데이터를 변환해야 합니다.
즉,
- 클라이언트 → 서버: JSON 데이터를 받아 Django 모델 객체로 저장하려면 역직렬화 필요
- 서버 → 클라이언트: 모델 데이터를 JSON으로 보내려면 직렬화 필요

---
◽ 전체 튜토리얼 목적:
	코드 스니펫을 생성, 조회, 수정, 삭제할 수 있는 RESTful API를 직접 구현하면서  DRF의 핵심 기능(Serializer, View, URL, 권한, 관계 등)을 하나씩 학습하도록 구성된 실습 프로젝트입니다.

`pygments` 외부 패키지 설치
```bash
pip install pygments
```

✅ `snippets/models.py` 작성
```python
from django.db import models
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted([(item, item) for item in get_all_styles()])

class Snippet(models.Model):
    created = models.DateTimeField(auto_now_add=True)
    title = models.CharField(max_length=100, blank=True, default='')
    code = models.TextField()
    linenos = models.BooleanField(default=False)
    language = models.CharField(choices=LANGUAGE_CHOICES, default='python', max_length=100)
    style = models.CharField(choices=STYLE_CHOICES, default='friendly', max_length=100)

    class Meta:
        ordering = ['created']
```
---
```python
# 마이그레이션
python manage.py makemigrations snippets
python manage.py migrate
```
---
###### 🔹 위의 모델을 테이블로 정의:
| 필드명        | 타입              | 설명                     |
| ---------- | --------------- | ---------------------- |
| `id`       | AutoField (PK)  | 자동 생성되는 기본 키           |
| `created`  | DateTimeField   | 생성 시각 자동 저장            |
| `title`    | CharField (100) | 코드 스니펫 제목 (옵션, 기본값 '') |
| `code`     | TextField       | 코드 내용                  |
| `linenos`  | BooleanField    | 줄 번호 표시 여부             |
| `language` | CharField (100) | 코드의 프로그래밍 언어 (선택지 있음)  |
| `style`    | CharField (100) | 문법 강조 스타일 (선택지 있음)     |
	`Meta.ordering = ['created']`는 Snippet 객체가 생성일 기준으로 정렬되도록 설정합니다.

---
`기본소스`
```python
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

LEXERS = [item for item in get_all_lexers() if item[1]]
```
이 함수들은 내부적으로 다음과 같은 튜플 중첩 리스트를 반환합니다:

`get_all_lexers()`란?
`pygments` 안의 모든 프로그래밍 언어 렉서 정보를 가져옵니다. (수백 개)
```python
[
  ('Python', ['python', 'py'], ['*.py'], ['text/x-python']),
  ('JavaScript', ['js'], ['*.js'], ['application/javascript']),
  ...
]
```

`pygments 데이터 확인`
```python
from pygments.lexers import get_all_lexers
from pygments.styles import get_all_styles

# 언어 종류 출력
print(" 모든 렉서(프로그래밍 언어):")
for lexer in get_all_lexers():
    print(lexer)

print("\n 지원되는 스타일 목록:")
for style in get_all_styles():
    print(style)
```
- 터미널에서 python test.py로 확인

`get_all_lexers()`는 다음과 같은 구조의 튜플을 여러 개 반환합니다:
```python
('Python', ['python', 'py'], ['*.py'], ['text/x-python']),

item = (
    item[0],        # 이름 (예: 'Python')
    item[1],        # 알리아스 목록 (예: ['python', 'py'])
    item[2],        # 확장자 목록 (예: ['*.py'])
    item[3],        # MIME 타입 (예: ['text/x-python'])
)
```
알리아스란? 언어 이름을 가리키는 다른 이름들(`'python'`, `'py'`, `'python3'`)

</>예시: 튜플데이터 한묶음
```python
item = ('Python', ['python', 'py'], ['*.py'], ['text/x-python'])

item[1]         # ['python', 'py'] ← 알리아스 목록 (리스트)
item[1][0]      # 'python'         ← 그 목록 중 첫 번째 항목
item[0]         # 'Python'         ← 튜플에서 첫 번째 요소 (언어 이름)
```

여기서 우리는:
```python
LANGUAGE_CHOICES = [(item[1][0], item[0]) for item in LEXERS]
```
→ 최종적으로 `('python', 'Python')` 형태의 튜플 리스트를 만들어 `choices`로 사용합니다.

`pygments`는 코드 하이라이팅(문법 강조) 라이브러리입니다. 직접 설치를 해야 사용할수 있습니다.
```bash
pip install pygments
```
`requirements.txt`에 추가할 수도 있습니다:
```
pygments>=2.0
```
---
아래 코드는 전역변수로 설정한 상수이자, 리스트 컴프리헨션(리스트 내포)을 사용해서 간결하게 작성된 코드입니다.
```python
LEXERS = [item for item in get_all_lexers() if item[1]]
LANGUAGE_CHOICES = sorted([(item[1][0], item[0]) for item in LEXERS])
STYLE_CHOICES = sorted([(item, item) for item in get_all_styles()])
```

`LEXERS` 일반 for문으로 풀기
```python
LEXERS = []
for item in get_all_lexers():
    if item[1]:  # item[1]은 별칭 목록, 예: ['python', 'py']
        LEXERS.append(item)
```

`LANGUAGE_CHOICES` 일반 for문으로 풀기
```python
LANGUAGE_CHOICES = []
for item in LEXERS:
    alias = item[1][0]     # 예: 'python'
    display_name = item[0] # 예: 'Python'
    LANGUAGE_CHOICES.append((alias, display_name))

LANGUAGE_CHOICES.sort()  # 알파벳 순으로 정렬
```

`STYLE_CHOICES` 일반 for문으로 풀기
```python
STYLE_CHOICES = []
for style in get_all_styles():
    STYLE_CHOICES.append((style, style))

STYLE_CHOICES.sort()
```
---
`get_all_lexers()`와 `get_all_styles()`는  Pygments 내부에 하드코딩된 데이터를 객체처럼 불러오는 함수이며,  우리는 for문으로 그 중 필요한 정보를 가공해서 템플릿/API에 보여주는 것입니다.

---
✅ 마이그레이션
```bash
python manage.py makemigrations snippets
python manage.py migrate
```
---
###### 🔹 시리얼라이즈(serialize)란?
	파이썬 객체를 JSON처럼 전송 가능한 데이터로 바꾸는 것입니다. 그리고 
	그 반대로도 가능해야 합니다. (역직렬화)

</>예시코드: 파이썬 객체
```python
snippet = Snippet(title="Hello", code="print('Hi')")
```

사용자가 프론트엔드에서 이렇게 요청하면:
```json
{
  "id": 1,
  "title": "인사",
  "code": "print('안녕하세요.')",
  "linenos": false,
  "language": "python",
  "style": "friendly"
}
```

서버는 이렇게 이해합니다:
```python
<Snippet: title="인사", code="print('안녕하세요.')", language="python">
```
이 중간에서 데이터 형식, 유효성 검사를 담당하는 것이 **Serializer**입니다!

---
🔷 시리얼라이저의 핵심 기능 2가지
`직렬화 (Serialize)` : 모델 인스턴스를 → JSON으로 변환
`역직렬화 (Deserialize)` : JSON을 → 파이썬 객체(모델 인스턴스)로 복원

시리얼라이저는  모델 필드들을 직렬화하는 과정입니다.

---
✅ `snippets/serializers.py` 작성: 기본형 Serializer 클래스
```python
from rest_framework import serializers
from snippets.models import Snippet, LANGUAGE_CHOICES, STYLE_CHOICES

class SnippetSerializer(serializers.Serializer):
    id = serializers.IntegerField(read_only=True)
    title = serializers.CharField(required=False, allow_blank=True, max_length=100)
    code = serializers.CharField(style={'base_template': 'textarea.html'})
    linenos = serializers.BooleanField(required=False)
    language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
    style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')

    def create(self, validated_data):
        return Snippet.objects.create(**validated_data)

    def update(self, instance, validated_data):
        for field in ['title','code','linenos','language','style']:
            setattr(instance, field, validated_data.get(field, getattr(instance, field)))
        instance.save()
        return instance
```
---
🔹 코드 해석: `serializers.py`

```python
class SnippetSerializer(serializers.Serializer):
```
	Serializer는 데이터를 JSON으로 바꾸고, JSON을 다시 객체로 바꾸는 역할
	Serializer는 DRF에서 정해진 상속 클래스명임
	SnippetSerializer는 개발자가 자유롭게 정하는 이름
---
Serializer와 Model 간에 필드명이 반드시 일치해야 정상적으로 직렬화/역직렬화가 됩니다.

`필드 정의`
```python
id = serializers.IntegerField(read_only=True)
```
	id는 읽기 전용으로 자동 생성됨
	read_only=True:입력값을 받지 않고 출력용으로만 사용됨

`id`는 모델에서 자동 생성하는데 왜 시리얼 라이저에서 써줘야 하는가?
	Serializer는 입출력 명세서 역할을 하므로, 자동 생성 여부와 관계없이 출력 필드를 직접 정의해야합니다.
`read_only=True`를 써야 하는 이유는?
	사용자가 입력할 수 없도록 제한하기 위함 (안 쓰면 API 요청에서 잘못된 값이 들어올 수 있음)

---
```python
title = serializers.CharField(required=False, allow_blank=True, max_length=100)
```
- `required=False`:필수 입력이 아님
	사용자가 제목을 안 보내도 된다는 뜻으로 폼으로 치면 name="title" 없이 제출해도 통과 된다는 뜻입니다.
- `allow_blank=True`:빈 문자열 허용한다는 뜻
	`required=False`가 필드를 생략 가능하게 해주는 것이라면,  
	`allow_blank=True`는 보내긴 보내되 내용은 없어도 OK라는 뜻입니다
- `요약정리:` 
	제목을 생략해도 되고(`required=False`),  
	빈칸이어도 괜찮고(`allow_blank=True`),  
	100자까지만 입력할 수 있어요.(`max_length=100`)

---
```python
code = serializers.CharField(style={'base_template': 'textarea.html'})
```
- `serializers.CharField(...)`
	Django REST Framework에서 문자열 필드를 나타내는 필드 타입입니다.
	HTML `<input>` 또는 `<textarea>`로 표현됩니다.
	이 필드는 `"코드(code)" 입력란`을 위한 것입니다.
- `style={...}`
	이 옵션은 DRF의 Browsable API에서 필드가 어떻게 보일지 UI를 조정하는 역할을 합니다.
	즉, DRF가 제공하는 테스트용 웹페이지에서 폼의 입력창 모양을 바꾸는 용도입니다.
	
==`Browsable API란?`==
	Django REST Framework(DRF)가 자동으로 제공하는 웹 기반 인터페이스로,  웹 브라우저에서 API를 직접 보고, 테스트해볼 수 있는 UI 페이지입니다.
		- HTML 폼 기반 인터페이스를 통해 `GET`, `POST`, `PUT`, `DELETE` 요청을 직접 보낼 수 있음
		- 개발자 입장에서 Postman 또는 Imsomnia 없이 API를 빠르게 확인할 수 있음
		- 마크다운 설명, 권한, 인증 등도 자동 렌더링됨

- `base_template: 'textarea.html'`
	기본 입력 UI 템플릿을 `'textarea.html'`로 지정
    결과적으로: 이 필드는 한 줄 input이 아니라 여러 줄이 가능한 `<textarea>` 형태로 아래 이미지와 같이 보여집니다.

![[Pasted image 20250614191947.png]]

---
```python
linenos = serializers.BooleanField(required=False)
```
	linenos는 “line numbers”의 약자입니다.
	줄 번호를 표시할지 여부 (True/False)
	required=False: 사용자가 이 필드를 생략해도 오류가 나지 않음

화면에 코드가 렌더링(rendering) 되어 줄 번호가 포함된 코드로 표현된 HTML 예시
```html
<pre>
1   print("Hello")
2   print("World")
</pre>
```

---
```python
language = serializers.ChoiceField(choices=LANGUAGE_CHOICES, default='python')
style = serializers.ChoiceField(choices=STYLE_CHOICES, default='friendly')
```
	언어와 스타일은 미리 정해진 선택지 중 고름

---
`데이터를 저장할 때 쓰는 메서드`
```python
def create(self, validated_data):
    return Snippet.objects.create(**validated_data)
```
	POST로 새 Snippet 만들 때 호출됨

---
```python
def update(self, instance, validated_data):
    for field in ['title', 'code', 'linenos', 'language', 'style']:
        setattr(instance, field, validated_data.get(field, getattr(instance, field)))
    instance.save()
    return instance
```
- `PUT`으로 기존 Snippet 수정할 때 호출됨
- 하나씩 새 값이 있으면 바꿔주고 저장함

---
🔷 `ModelSerializer` 버전
```python
class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style']
```
그러나 현업에서는 `ModelSerializer`를 사용합니다.

---
🔷 모델 시리얼 라이저에서 오버라이드가 필요할때:
- `save() 시 추가 로직이 필요할 때`	
	예: 작성자 자동 추가, 로그 저장, 특정 값 가공
- `외부 API 호출과 함께 저장할 때`	
	예: 결제 요청 → 저장
- `특정 필드를 조건에 따라 조작할 때`	
	예: title이 비어 있으면 기본값 채우기

</> 예시코드: 제목이 비어 있을 때 자동으로 "Untitled"로 저장되도록 수정
`create()` 오버라이드
```python
from rest_framework import serializers
from snippets.models import Snippet

class SnippetSerializer(serializers.ModelSerializer):
    class Meta:
        model = Snippet
        fields = ['id', 'title', 'code', 'linenos', 'language', 'style']

    def create(self, validated_data):
        if not validated_data.get('title'):
            validated_data['title'] = 'Untitled'
        return super().create(validated_data)
```
- `ModelSerializer`의 `create()`는 원래 자동 생성됨
- `super().create()`를 사용하여 기본 동작을 재사용
- 그 전에 우리가 원하는 전처리/로직 삽입이 가능

</> 예시코드: 코드를 수정할 때, 코드 내용에 금지 단어가 있으면 에러 발생시키기

`update()` 오버라이드
```python
    def update(self, instance, validated_data):
        code = validated_data.get('code', instance.code)
        if 'hack' in code:
            raise serializers.ValidationError({"code": "금지된 단어가 포함되어 있습니다."})
        return super().update(instance, validated_data)
```
- `update()` 메서드는 PUT/PATCH 요청 시 호출됩니다
- 특정 값 검증을 직접 추가할 수 있음 (`serializer.is_valid()`에서 오류 발생)

---
✅ Django shell에서 테스트: 
```bash
python manage.py shell
```

```python
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer
from rest_framework.renderers import JSONRenderer
from rest_framework.parsers import JSONParser
import io

# 모델 인스턴스 생성 및 저장
snippet = Snippet(code='print("hello, world")')
snippet.save()

# 직렬화: 모델 인스턴스를 JSON 데이터로 변환
serializer = SnippetSerializer(snippet)
json_data = JSONRenderer().render(serializer.data)

# 역직렬화: JSON 데이터를 다시 Python 객체로 읽고, 저장
stream = io.BytesIO(json_data)
data = JSONParser().parse(stream)

serializer = SnippetSerializer(data=data)
serializer.is_valid()  # True
serializer.save()  # Snippet 인스턴스로 저장
```
---
==`모델 인스턴스 생성 및 저장`==
```python
snippet = Snippet(code='print("hello, world")')
snippet.save()
```
- `Snippet` 모델에 `"print("hello, world")"`라는 값을 넣고 저장합니다.
- 여기서 `code`는 모델의 필드 중 하나입니다.
- `Snippet`은 `models.Model`을 상속받은 Django 모델 클래스입니다.
- 위 코드는 `Snippet` 클래스의 인스턴스를 만들며,
- `code='print("hello, world")'` 라는 필드값을 전달하고 있습니다.
- `.save()`는 모델 인스턴스를 실제 데이터베이스 테이블에 저장합니다.
---
==`직렬화: 모델 인스턴스를 JSON 데이터로 변환`==
```python
serializer = SnippetSerializer(snippet)
json_data = JSONRenderer().render(serializer.data)
```

`serializer = SnippetSerializer(snippet)`
- `SnippetSerializer`는 DRF의 직렬화 클래스입니다.
- `snippet`은 앞에서 만든 `Snippet` 모델 인스턴스를 인자값으로 받습니다.
- `snippet` 객체를 직렬화 가능한 형태(파이썬 딕셔너리)로 준비합니다.

`serializer.data`를 하면 아래처럼 딕셔너리 형태로 변환됩니다:
```python
{
    'id': 1,
    'title': '',
    'code': 'print("hello, world")',
    'linenos': False,
    'language': 'python',
    'style': 'friendly'
}
```
- 필드 구성은 `SnippetSerializer` 안에 정의된 필드들에 따릅니다.

`json_data = JSONRenderer().render(serializer.data)`
- `.render()`는 딕셔너리 데이터를 JSON 문자열 (bytes 타입)로 변환합니다.
- bytes 타입은 브라우저나 API 클라이언트(Postman 등)에 응답으로 보내는 형식입니다.
```python
b'{"id":1,"title":"","code":"print(\\"hello, world\\")","linenos":false,"language":"python","style":"friendly"}'
```
- 여기서 `b'...'` 는 bytes 객체라는 뜻이고,
- 실제로 API에서 응답으로 내려보낼 수 있는 JSON 문자열입니다.
---
==`역직렬화: JSON 데이터를 다시 Python 객체로 읽고, 저장`==
```python
stream = io.BytesIO(json_data)
data = JSONParser().parse(stream)
```

`stream = io.BytesIO(json_data)`
- `json_data`는 `b'{...}'` 형태의 바이트 타입 JSON 문자열입니다.
- `BytesIO()`는 메모리 안에 있는 파일처럼 작동하는 스트림(stream)을 만듭니다.

`data = JSONParser().parse(stream)`
- DRF의 `JSONParser`는 스트림을 읽어서 Python 딕셔너리로 파싱합니다.
- 즉, JSON 문자열을 Python 딕셔너리로 변환합니다.
- 파싱이란 복잡한 문자열 데이터를 사람이 또는 프로그램이 이해할 수 있는 구조로 바꾸는 작업을 말합니다.
```python
# stream에 들어있는 JSON이:
b'{"code": "print(\\"hello\\")", "language": "python"}'

# parse 결과:
{
  "code": 'print("hello")',
  "language": "python"
}
```
---
◽  직렬화 공식 (Serialization)
`모델 인스턴스 → Python dict → JSON 바이트 문자열`
```python
from rest_framework.renderers import JSONRenderer

# 1. 모델 인스턴스를 시리얼라이저에 전달
serializer = SnippetSerializer(snippet)  # snippet: Snippet 모델 객체

# 2. serializer.data → Python dict
# 3. JSONRenderer로 JSON 문자열(bytes)로 변환
json_data = JSONRenderer().render(serializer.data)
```

◽  역직렬화 공식 (Deserialization)
`JSON 바이트 문자열 → Python dict → 모델 인스턴스 생성 및 저장`
```python
from rest_framework.parsers import JSONParser
import io

# 1. JSON 문자열을 메모리 스트림으로 변환
stream = io.BytesIO(json_data)

# 2. JSONParser로 Python dict로 파싱
data = JSONParser().parse(stream)

# 3. 시리얼라이저에 dict 전달
serializer = SnippetSerializer(data=data)

# 4. 유효성 검사 후 저장
if serializer.is_valid():
    serializer.save()
```
---
✅ 뷰 작성 (`snippets/views.py`)
```python
from django.http import JsonResponse, HttpResponse
from django.views.decorators.csrf import csrf_exempt
from rest_framework.parsers import JSONParser
from snippets.models import Snippet
from snippets.serializers import SnippetSerializer

@csrf_exempt
def snippet_list(request):
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return JsonResponse(serializer.data, safe=False)

    elif request.method == 'POST':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data, status=201)
        return JsonResponse(serializer.errors, status=400)

@csrf_exempt
def snippet_detail(request, pk):
    try:
        snippet = Snippet.objects.get(pk=pk)
    except Snippet.DoesNotExist:
        return HttpResponse(status=404)

    if request.method == 'GET':
        serializer = SnippetSerializer(snippet)
        return JsonResponse(serializer.data)

    elif request.method == 'PUT':
        data = JSONParser().parse(request)
        serializer = SnippetSerializer(snippet, data=data)
        if serializer.is_valid():
            serializer.save()
            return JsonResponse(serializer.data)
        return JsonResponse(serializer.errors, status=400)

    elif request.method == 'DELETE':
        snippet.delete()
        return HttpResponse(status=204)
```

`@csrf_exempt`
- Django는 기본적으로 POST/PUT/DELETE 요청 시 CSRF 토큰을 검사합니다.
- 그러나 API 요청은 브라우저 기반이 아닌 경우가 많고 (예: Postman, axios, 모바일 앱 등), 이 경우 CSRF 토큰이 없기 때문에 오류가 발생합니다.
- 따라서 `@csrf_exempt`를 붙이면 CSRF 보안 검사를 우회하여 요청을 허용할 수 있습니다.
- 단, `@csrf_exempt`는 임시방편 또는 개발/테스트용으로만 사용하는 것이 좋습니다.
- 실제 운영 환경에서는 CSRF 대신 토큰 기반 인증(JWT, SimpleJWT 등)을 사용하는 것이 더 안전하고 일반적입니다.

==`첫번째 함수`==
`def snippet_list(request):`
	`if request.method == 'GET':` 데이터 불러오기
```python
snippets = Snippet.objects.all()
serializer = SnippetSerializer(snippets, many=True)
return JsonResponse(serializer.data, safe=False)
```
- `snippets = Snippet.objects.all()`: 모든 `Snippet` 객체를 가져옵니다.
- `serializer`: `many=True`로 여러 개를 직렬화합니다.
- `JsonResponse`: JSON 형태로 반환 (리스트 형태)합니다.
	  `serializer.data`의 값 자체가 리스트형태입니다.
- `safe=False`: 리스트를 JSON으로 응답하기 위해 필요
	-`safe=True` (기본값) : 딕셔너리만 허용 `{"message": "ok"}`
	-safe=False: 리스트 등 비딕셔너리도 허용`[{"id":1}, {"id":2}]`
---
	`elif request.method == 'POST':` 데이터 생성
```python
data = JSONParser().parse(request)
serializer = SnippetSerializer(data=data)
if serializer.is_valid():
   serializer.save()
   return JsonResponse(serializer.data, status=201)
return JsonResponse(serializer.errors, status=400)
```
- `JSONParser` : JSON 데이터를 파싱(분석 및 변환)하는 클래스로 
- `parse()`메서드를 이용해 요청 데이터를 JSON → Python dict로 변환합니다.
- `serializer`변수 : 역직렬화용 `SnippetSerializer` 인스턴스를 생성하는 과정입니다.
- `serializer`변수의 유효성 검사 (`is_valid()`) 후 `.save()`로 DB에 저장
- 저장 성공 시 직렬화된 데이터를 반환 (`201 Created`)
- 유효하지 않으면 오류 정보를 포함해 `400 Bad Request` 응답

- HTTP 메서드별 기능 요약:
	`GET 조회:`	목록 조회 또는 상세 조회 (읽기)
	`POST 생성:` 새 데이터 저장 (등록)
	`PUT 수정:`	기존 데이터 전체 수정 (덮어쓰기)
	`PATCH 부분 수정:`	기존 데이터 일부 수정
	`DELETE 삭제:`	 기존 데이터 삭제

---
==`두번째 함수`==
`def snippet_detail(request, pk):`
```python
try:
    snippet = Snippet.objects.get(pk=pk)
except Snippet.DoesNotExist:
    return HttpResponse(status=404)
```
- 먼저 `pk`로 객체 조회합니다.
- 없으면 `404 Not Found` 반환합니다.

◽ 예외 발생 상황:
	DB에 해당 `pk` 값을 가진 Snippet 객체가 존재하지 않을 때
	예) 요청 URL이 `/snippets/999/`인데 - `id=999`인 Snippet 객체가 DB에 없을 때

---
`if request.method == 'GET':` GET 요청 (조회)
```python
serializer = SnippetSerializer(snippet)
return JsonResponse(serializer.data)
```
- `serializer`: 단일 객체 직렬화합니다.
- `JsonResponse`: JSON 형태로 반환 (리스트 형태)합니다.
	  `serializer.data`의 값 자체가 리스트형태입니다.
---
`elif request.method == 'PUT':` PUT 요청 (수정)
```python
data = JSONParser().parse(request)
serializer = SnippetSerializer(snippet, data=data)
if serializer.is_valid():
    serializer.save()
    return JsonResponse(serializer.data)
return JsonResponse(serializer.errors, status=400)
```
- `JSONParser` : JSON 데이터를 파싱(분석 및 변환)하는 클래스로 
- `parse()`메서드를 이용해 요청 데이터를 JSON → Python dict로 변환합니다.
- `serializer변수에` 기존 객체(`snippet`) + 새 데이터(`data`)를 함께 대입
- `if` 조건문은 DRF의 유효성 검증을 통과한 경우만 저장(save)하도록 처리
	예: 필수 필드 누락, 잘못된 값, 형식 오류 등이 없는지 확인
- 누락등 오류발생시 400에러값으로 반환합니다.

---
`elif request.method == 'DELETE':` DELETE 요청 (삭제)
```python
snippet.delete()
return HttpResponse(status=204)
```
- 해당 인스턴스를 삭제하고
- 응답 코드 204는 "요청은 성공했지만, 보낼 내용은 없다"는 뜻입니다.
- 요청(예: 삭제 요청)은 정상적으로 처리되었으나 서버는 보내줄 데이터가 없기 때문에 응답 본문은 비어있습니다.
결론적으로 "데이터는 잘 삭제했어요. 그런데 보여줄 건 없어요." 라는 의미예요.

---
✅ URL 설정: `snippets/urls.py` 생성
```python
from django.urls import path
from snippets import views

urlpatterns = [
    path('snippets/', views.snippet_list),
    path('snippets/<int:pk>/', views.snippet_detail),
]
```

`tutorial/urls.py`에 포함시키기
```python
from django.urls import path, include

urlpatterns = [
    path('', include('snippets.urls')),
]
```
---
✅ API 테스트 방법: 서버 실행
```bash
python manage.py runserver
```

---
🔗 [https://insomnia.rest/download](https://insomnia.rest/download)
	회원가입후 설치합니다.
	
###### 🔹 Insomnia 사용
| 요청                       | URL            | 메서드              | 설명                      |
| ------------------------ | -------------- | ---------------- | ----------------------- |
| 전체 목록 조회                 | `/snippets/`   | `GET`            | 모든 Snippet을 JSON으로 받아옴  |
| 새 Snippet 생성             | `/snippets/`   | `POST`           | JSON 데이터로 새 Snippet 추가  |
| 특정 Snippet 조회            | `/snippets/1/` | `GET`            | ID=1인 Snippet의 상세 정보 조회 |
| 특정 Snippet 수정 .......... | `/snippets/1/` | `PUT` or `PATCH` | ID=1인 Snippet 수정        |
| 특정 Snippet 삭제            | `/snippets/1/` | `DELETE`         | ID=1인 Snippet 삭제        |

```json
{
  "title": "Hello World",
  "code": "print('Hello, World!')",
  "language": "python",
  "style": "friendly",
  "linenos": true
}
```

![[Pasted image 20250607232829.png]]

---
Insomnia(또는 Postman)으로 API 테스트를 하고 JSON 응답이 잘 나왔다면,
이 주소는 실제로 Axios 등 프론트엔드 코드에서 데이터를 불러올 수 있는 주소가 됩니다.

Insomnia로 확인한 주소는 “API 엔드포인트(Endpoint)”입니다.

| 용도            | 설명                                                                         |
| ------------- | -------------------------------------------------------------------------- |
| 프론트 <br>엔드 연결 | JS (Axios, Fetch)로 데이터 요청 (`axios.get('http://localhost:8000/snippets/')`) |
| 모바일 앱 연결      | React Native, Flutter 등에서도 해당 API 주소 사용                                    |
| 다른 서버 간 통신    | 예: 마이크로서비스 구조에서 백엔드 간 통신                                                   |
| 자동화 <br>스크립트  | Python, Shell 등으로 해당 API를 호출해 자동화                                          |
| 외부 공개 API     | 인증 후 외부 개발자도 호출 가능하게 (예: Open API)                                         |

사용예시
```js
axios.get("http://localhost:8000/snippets/")
  .then(response => {
    console.log(response.data);  // → JSON 데이터 출력
  });
```

---
httpie로 GET 테스트
```bash
pip install httpie
http GET http://127.0.0.1:8000/snippets/
http GET http://127.0.0.1:8000/snippets/1/
```

POST 테스트
```bash
http POST http://127.0.0.1:8000/snippets/ code="print('hello')"
```



