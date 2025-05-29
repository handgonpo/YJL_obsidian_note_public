🔹 Django 튜토리얼 Part 2 – DB 모델 만들기
###### 📖 공식 문서 링크:  
🔗 https://docs.djangoproject.com/ko/5.2/intro/tutorial02/

목표
- 데이터베이스 설정과 생성
- 설문조사 모델(`Question`, `Choice`) 만들기
- 관리자(admin) 페이지에 설문 데이터 등록하기
- Django ORM을 활용하여 데이터를 직접 저장하고 불러오기

🔹 ORM이란?  
ORM은 "Object-Relational Mapping"의 약자입니다.  
한글로는 객체-관계 매핑이라고 합니다.  
즉, 데이터베이스 테이블과 파이썬 객체를 자동으로 연결해주는 기술입니다.

보통 데이터베이스는 SQL이라는 언어로 데이터를 다룹니다:

```python
SELECT * FROM question WHERE id=1;
```

그런데 파이썬은 객체 지향 언어입니다:

```python
question = Question.objects.get(id=1)
```

SQL을 직접 쓰는 대신, 파이썬 코드로 DB를 조작할 수 있게 해주는 게 ORM입니다! 즉 모델정의 테이블 정의가 ORM입니다.

---
📁 프로젝트 구조:
```
mysite/                ← 최상위 프로젝트 디렉토리
├── manage.py          ← Django 명령어 실행 파일
├── mysite/            ← 프로젝트 설정 디렉토리
│   ├── __init__.py
│   ├── settings.py    ← ==환경 설정 파일==
│   ├── urls.py        ← ==전체 URL 연결 파일==
│   ├── asgi.py
│   └── wsgi.py
├── polls/             ← polls 앱 디렉토리
│   ├── __init__.py
│   ├── admin.py       ← ==관리자 설정==
│   ├── apps.py        ← 앱 설정 정보
│   ├── migrations/    ← 마이그레이션 파일 저장 폴더
│   │   └── __init__.py
│   ├── models.py      ← ==데이터베이스 모델 정의==
│   ├── tests.py       ← 테스트 코드
│   ├── views.py       ← ==뷰(View) 함수 정의==
│   └── urls.py        ← ==polls 앱의 URL 연결 파일==

```
- `mysite/` (상단): 전체 프로젝트 디렉토리
- `mysite/` (하위): 프로젝트 설정 파일들 (settings, urls 등)
- `polls/`: 여러분이 만든 첫 번째 앱(설문조사 기능 담당)
- `migrations/`: 데이터베이스 구조 변경 이력 파일 저장소
- `manage.py`: Django 명령어 (`runserver`, `migrate` 등) 실행용

---
✅ 1단계: 데이터베이스 설정 확인  [DB설정 공식문서](https://docs.djangoproject.com/en/4.1/ref/settings/#databases)

📂 `mysite/settings.py` 파일을 엽니다.
- 기본 DB는 `SQLite`입니다.
- 설정은 다음과 같습니다 (기본 그대로 사용해도 됩니다):

```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
```

`시간대를 한국 기준으로 변경`
```python
LANGUAGE_CODE = "ko-kr"
TIME_ZONE = "Asia/Seoul"     # 한국 시간으로 수정
USE_I18N = True
USE_TZ = True                # 반드시 True로 유지
```
---
✅ 2단계: 데이터베이스 테이블 생성

다음 명령어를 입력해 Django 내부 앱들의 기본 테이블을 생성합니다.
```python
python manage.py migrate
```
- 이 명령어는 `INSTALLED_APPS`에 있는 앱들의 테이블을 생성해줍니다.
---
✅ 3단계: 모델 만들기

`polls/models.py` : 전체코드
```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")

    def __str__(self):
        return self.question_text

    def was_published_recently(self):
        from django.utils import timezone
        import datetime
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete = models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
```
---
`Question` 모델만 정의
```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")
```
`DateTimeField` : 실제 기능은 날짜와 시간을 저장하는 필드

```python
python manage.py makemigrations 
# 모델 변경사항을 감지해 마이그레이션 파일 생성 

python manage.py migrate
# 생성된 마이그레이션 파일을 DB에 적용
```

`VSCode Extension`
![[Pasted image 20250530071326.png]]
생성한 테이블을 시각적으로 확인할수 있습니다.
![[Pasted image 20250530071633.png|200]]
db.sqlite3 파일로 VSCode내에서 생성한 데이터 테이블을 확인할수 있습니다.

---
`Choice` 모델 추가
```python
class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete=models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)
```

```python
python manage.py makemigrations
python manage.py migrate
```
모델이 변경이 되었으니 다시 마이그레이션, 마이그레이트를 합니다.

---
`관리자 계정 생성`
```python
python manage.py createsuperuser
```

```python
python manage.py runserver
```

`polls/admin.py` 관리자 페이지로 데이터 확인
```python
from django.contrib import admin
from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```

```python
class Question(models.Model):

    def __str__(self):
        return self.question_text
    
class Choice(models.Model): 
    def __str__(self):
        return self.choice_text
```
`__str__` 메서드를 추가하는 것은 데이터베이스 스키마(테이블 구조)를 변경하는 것이 아니기 때문에 `makemigrations`나 `migrate`는 필요 없습니다.

```python
    def was_published_recently(self):
        from django.utils import timezone
        import datetime
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)
```
이 함수는 기존 `pub_date`가 최근 1일 이내인지 판별해주는 "판단용" 메서드입니다.

---
✅ 4단계: 마이그레이션 만들기

모델을 작성했으면, DB에 적용하기 위한 중간 파일을 생성합니다:
```python
python manage.py makemigrations polls
```
- 이 명령은 `polls/migrations/0001_initial.py` 파일을 만들어줍니다.  
- 이 파일은 "설계도"입니다.
---
✅ 5단계: 마이그레이션 실제 반영

DB에 테이블을 실제로 만들려면 아래 명령을 실행합니다:
```python
python manage.py migrate
```
- 이 명령으로 실제 데이터베이스에 `polls_question`, `polls_choice` 테이블이 생성됩니다.

DB 반영뒤 SQlite 뷰어에서 VSCode를 통해 테이블을 확인합니다.
![[Pasted image 20250530074355.png]]

---
◽ 모델 롤백 (Rollback the Model)
	모델을 변경한 후, 이전 테이블 구조로 되돌리고 싶을 때 사용하는 방법입니다.

💡 `롤백 명령어`
```python
# 형식
python manage.py migrate <앱 이름> <되돌릴 마이그레이션 파일 이름>

# 예시
python manage.py migrate polls 0001_initial
```

🔍 해설:
- `polls`는 롤백하려는 Django 앱 이름입니다.
- `0001_initial`은 롤백하고자 하는 마이그레이션 버전입니다.  
    → 이 명령은 현재 상태에서 `0001_initial` 시점으로 테이블 구조를 되돌립니다.
    
🛑 주의: 롤백은 데이터 손실이 발생할 수 있으니 신중하게 실행해야 합니다!

`models.py`에 임시 필드 하나 추가:
```python
class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")
    
    # 연습용 필드 추가
    test_field = models.CharField(max_length=50, default='임시')

    def __str__(self):
        return self.question_text
```

```python
python manage.py makemigrations polls  # → 0002 생성됨
python manage.py migrate polls         # → DB 반영됨
```
	테이블이 생성되었는지 SQliet 뷰어에서 확인합니다.

`그 다음 롤백:`
```python
python manage.py migrate polls 0001
```
	롤백후 SQLite 뷰어에서 테이블이 삭제되었는지 확인할수 있습니다.

---
✅ 6단계: Django Shell에서 ORM 실습하기
```python
python manage.py shell
```

Django shell 환경에서 사용하는 파이썬 객체 조작 코드 문법
```python
from polls.models import Question, Choice
from django.utils import timezone

# CREATE (생성)
q = Question(question_text="What's new?", pub_date=timezone.now())
q.save()

# READ (조회)
q.id # 1
q.question_text # "What's new?"
q.pub_date 

# UPDATE (수정)
q.question_text = "What's up?"
q.save()

# READ ALL (전체 조회)
Question.objects.all() 

# DELETE 
q = Question.objects.get(pk=3)  # 먼저 정확한 객체를 조회함
q.delete()   # 해당 객체 삭제

# READ ALL AGAIN (삭제 후 확인)
print(Question.objects.all())  
```

`__str__()` 메서드를 추가했기 때문에 이제 객체 이름이 보기 좋게 출력됩니다:
```python
<Question: What's up?>
```

✅ 7단계: Choice 연결 예제 (1:N 관계)
```bash
q = Question.objects.get(pk=1)
q.choice_set.create(choice_text="Not much", votes=0)
q.choice_set.create(choice_text="The sky", votes=0)
q.choice_set.create(choice_text="Just hacking again", votes=0)

# 연결된 Choice 보기
q.choice_set.all()
```
- `Question`과 `Choice`가 외래키(ForeignKey)로 잘 연결되었는지 확인 

✅ 8단계: 관리자 페이지 등록

`polls/admin.py` 열고 아래처럼 수정:
```python
from django.contrib import admin
from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```

✅ 9단계: 관리자 계정 만들기
```bash
python manage.py createsuperuser
```
- 사용자명, 이메일, 비밀번호 입력

✅ 10단계: 관리자 페이지 접속
```bash
python manage.py runserver
```

웹 브라우저에서 접속:
```
http://127.0.0.1:8000/admin/
```
- 아까 만든 슈퍼유저 계정으로 로그인
- "Questions" 메뉴가 보이면 성공
---
✨ 코드리뷰

1️⃣ 📁 `mysite/settings.py`
- 기본 데이터베이스는 `SQLite`로 설정되어 있습니다.
	Django는 기본적으로 SQLite를 기본 데이터베이스로 사용하도록 설정되어 있습니다.  개발 및 테스트 환경에서는 설정이 간단하고 별도 설치가 
	필요 없는 SQLite를 사용하며,  운영(배포) 환경에서는 보다 안정적이고
	확장성이 뛰어난 PostgreSQL 등으로 데이터베이스를 전환하는 것이 일반적입니다. [SQLite 다운로드](https://sqlitebrowser.org/)
```python
DATABASES = {
    "default": {
        "ENGINE": "django.db.backends.sqlite3",
        "NAME": BASE_DIR / "db.sqlite3",
    }
}
```

`TIME_ZONE` 설정을 내 시간대로 바꾸세요.
```python
TIME_ZONE = 'Asia/Seoul'  # 예: 한국 시간
```

`INSTALLED_APPS` 목록에는 Django의 기본 기능들이 들어 있음:
```python
INSTALLED_APPS = [
    "django.contrib.admin",  # 관리자 페이지
    "django.contrib.auth",   # 로그인/인증
    ...
]
```
---
2️⃣ 데이터베이스 테이블 만들기
아래 명령어를 실행하면, Django가 기본 테이블을 생성합니다:
```python
python manage.py migrate
```
- 위 명령은 `INSTALLED_APPS`에 있는 앱들이 필요한 데이터베이스 테이블을 만들어줍니다.
- Django 내부에서 사용하는 테이블들이 생성됩니다.
---
3️⃣ 모델 만들기 (데이터 구조 정의)
📁 `polls/models.py` 파일을 다음처럼 작성하세요:
```python
from django.db import models

class Question(models.Model):
    question_text = models.CharField(max_length=200)
    pub_date = models.DateTimeField("date published")

    def __str__(self):
        return self.question_text

    def was_published_recently(self):
        from django.utils import timezone
        import datetime
        return self.pub_date >= timezone.now() - datetime.timedelta(days=1)

class Choice(models.Model):
    question = models.ForeignKey(Question, on_delete = models.CASCADE)
    choice_text = models.CharField(max_length=200)
    votes = models.IntegerField(default=0)

    def __str__(self):
        return self.choice_text
```

`class Question(models.Model)`: 
	질문(Question)을 표현하는 Django 모델 클래스 정의입니다.
- Django에서 모델 클래스를 정의할 때 사용하는 상속 구문으로,
- `models.Model`은 Django가 제공하는 부모 클래스입니다.
- 이 클래스를 상속하면 Django는 이 클래스를 데이터베이스 테이블과 연결된 모델로 인식합니다.
- 즉, `Question`은 Django ORM 시스템에 의해 관리되는 모델 클래스가 됩니다.
---
`question_text = models.CharField(max_length=200)` : 
	속성생성 또는 필드정의이며 테이블의 열(Column) 또는 필드(Field)가 됨.
	질문 내용을 저장하는 문자 필드이며, 최대 200자까지 입력 가능합니다.
	 `max_length=200` 반드시 문자의 길이를 지정해줘야 하며 지정하지 않으면 오류가 납니다.

---
`pub_date = models.DateTimeField("date published")`
	질문이 게시된 날짜와 시간을 저장하는 필드이며, 라벨로 "date published"를 지정했습니다.
![[Pasted image 20250525190905.png]]

---
`def __str__(self):`
	`return self.question_text
	
- `__str__()`은 파이썬 내장 특수 메서드입니다.
- 이 메서드를 정의하면, 해당 객체를 `print()`하거나 관리자(admin) 페이지, Django Shell, 템플릿 등에서 사용할 때 사람이 읽기 쉬운 형태로 표시됩니다.

`__str__()`이 없는 경우:
```bash
>>> q = Question.objects.first()
>>> print(q)
<Question: Question object (1)>
```

`__str__()`이 있는 경우:
```python
def __str__(self):
    return self.question_text
```

```bash
>>> print(q)
What's up?
```

✨ 추가팁:
Django 개발 중에 VSCode나 터미널에서 `python manage.py shell` 명령으로 셸에 진입할 때마다, 모듈을 다시 import 해야 합니다.

```bash
python manage.py shell
```

```bash
from polls.models import Question
Question.objects.all()
exit()
```

셸을 종료한 후, 다시 열면:
```bash
python manage.py shell
```

다시 import 해야 함
```bash
from polls.models import Question
Question.objects.all()
```

🔁 반복되는 import가 귀찮을 때는?
`shell_plus` 사용 (추천)
```bash
pip install django-extensions
```

`settings.py`
```python
INSTALLED_APPS = [
    ...
    'django_extensions',
]
```

그 다음에는:
```bash
python manage.py shell_plus
```
- 자동으로 대부분의 모델이 import 되어 있습니다.  
- `from polls.models import Question` 이런 거 안 해도 됩니다.
---
`def was_published_recently(self):`
        `from django.utils import timezone`
        `import datetime`
        `return self.pub_date >= timezone.now() -` `datetime.timedelta(days=1)`
        
- 이 함수는 "이 질문(Question)이 최근 1일(24시간) 이내에 등록되었는가?" 를 판단해서  `True` 또는 `False` 값을 리턴합니다.
`self.pub_date` : 이 질문(Question)이 등록된 날짜와 시간
`timezone.now()` : 지금 현재 시간 (시간대 포함)
`datetime.timedelta(days=1)` : "하루 전" 시간 계산 도구
`>=` : 최근이면 True, 오래됐으면 False

---
`class Choice(models.Model)`
	선택지(Choice)를 표현하는 Django 모델 클래스 정의입니다.

---
`question = models.ForeignKey(Question, on_delete=models.CASCADE)`
	각 선택지가 어떤 질문에 속하는지를 나타내는 외래 키 필드입니다. 질문이 삭제되면 선택지도 함께 삭제됩니다.
	`ForeignKey`:다른 모델(테이블)과 1:N 관계를 정의하는 필드
	`Question`:연결할 다른 모델 클래스 (즉, 참조 대상 테이블)
	`on_delete=models.CASCADE`: 참조 대상(Question)이 삭제될 때 이 모델의 처리 방법을 정의

`on_delete = models.CASCADE`
	`on_delete`는 필수 매개변수이며, 외래키로 연결된 객체가 삭제될 때의 동작을 정합니다.

| 옵션            | 동작 설명                             |
| ------------- | --------------------------------- |
| `CASCADE`     | 게시글이 삭제되면 그 글에 달린 댓글도 모두 삭제       |
| `PROTECT`     | 주문 내역이 남아 있으면 상품 삭제를 막고 싶을 때      |
| `SET_NULL`    | 사용자가 탈퇴하면 작성자 정보만 비워두고 글은 남길 때    |
| `SET_DEFAULT` | 기본 사용자로 전환하거나, 기본 카테고리로 바꿀 때      |
| `DO_NOTHING`  | 아무 일도 하지 않음 (주의: DB 오류가 발생할 수 있음) |
| `SET(...)`    | 특정 값이나 함수로 설정                     |

---
`choice_text = models.CharField(max_length=200)`
	선택지 내용을 저장하는 문자 필드이며, 최대 200자까지 입력 가능합니다.
	
---
`votes = models.IntegerField(default=0)`: 정수형 필드 정의
	각 선택지가 받은 투표 수를 저장하는 정수형 필드이며, 기본값은 0입니다.
	`default=0`: 입력이 없을 때 자동으로 0을 저장합니다.
	
---
4️⃣ 모델을 마이그레이션(Migrate)하기
1단계: 마이그레이션 파일 생성
```python
python manage.py makemigrations polls
```
◽  마이그레이션(migration)이란?
- 장고에서는 모델 클래스(`models.py`)로 테이블 구조를 설계합니다.
- 하지만, 이 설계 내용을 실제로 데이터베이스에 적용하려면  
    먼저 장고가 이해할 수 있는 변환 파일(= 마이그레이션 파일)로 만들어야 해요.
- 이걸 자동으로 해주는 게 바로 `makemigrations` 명령어입니다.

◽  마이그레이션(migration) 명령어가 하는일
- `polls` 앱 안의 `models.py`에 있는 모델 변경 사항을 감지해서
- `polls/migrations/` 폴더 안에 새로운 마이그레이션 파일을 만들어줘요.
- 이 파일은 이런 상태로 생성됩니다:
```python
0001_initial.py
```
- 이 파일 안에는? 테이블을 만들기 위한 설계 정보 (CreateModel 명령)이 들어 있어요
---
`생성된 SQL 문 확인 (선택 사항)`
```bash
python manage.py sqlmigrate polls 0001
```

Django에서 실제 SQL이 어떻게 생성되는지 확인하려면 다음 명령어를 실행
`실제 SQL 확인 방법`
```python
python manage.py sqlmigrate 앱이름 마이그레이션번호
```

`아래와 같이 입력하세요.`
```python
python manage.py sqlmigrate polls 0001
```
그러면 Django가 생성할 SQL 코드가 터미널에 출력됩니다.

그러나 직접 SQL을 작성하지 않고도 장고에서 모델을 클래스로 짜면 자동으로 SQL문이 생성됩니다. 
`Question 테이블의 SQL`
```python
CREATE TABLE myapp_question (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    question_text VARCHAR(200) NOT NULL,
    pub_date DATETIME NOT NULL
);
```

`Choice 테이블의 SQL`
```python
CREATE TABLE myapp_choice (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    question_id INTEGER NOT NULL,
    choice_text VARCHAR(200) NOT NULL,
    votes INTEGER NOT NULL DEFAULT 0,
    FOREIGN KEY(question_id) REFERENCES myapp_question(id) ON DELETE CASCADE
);
```
위 SQL에서 `myapp_`은 앱 이름에 따라 접두사가 바뀔 수 있으며, 실제 실행 시점에는 마이그레이션 파일을 통해 자동 생성됩니다.

---
5️⃣ 마이그레이션 실제 반영

polls/migrations/0001_initial.py 파일이 자동 생성됩니다.
```python
python manage.py migrate
```

◽ migrate 란?
- 이전 단계에서 `makemigrations`로 만든 마이그레이션 파일(`0001_initial.py`)은  
    "이렇게 테이블을 만들어 주세요"라는 설계서(지시서)입니다.
    
- 이제 `migrate`를 실행하면 장고가 이 설계서를 보고  
    실제로 데이터베이스에 테이블을 만들어주는 작업을 해요.

◽ migrate 명령어가 하는 일:
- `db.sqlite3` 파일(기본 데이터베이스 파일)을 열어서
- `polls_question`, `polls_choice` 같은 진짜 테이블을 실제로 생성합니다.
- 장고가 자동으로 만들어주는 기본 앱들(admin, auth 등)도 이때 같이 적용됩니다.
---
6️⃣ 모델 테스트: Django Shell

VSCode 프로젝트 폴더 안(`manage.py`가 있는 폴더)에서 열어야 합니다.
```python
python manage.py shell
```
데이터베이스에 실제 데이터를 넣고 확인하기 위해 하는 모델 테스트 과정입니다. 장고의 모델이 제대로 작동하는지, 데이터베이스에 잘 저장되는지 확인하기 위해 쉘에서 직접 테스트하는 단계입니다.

makemigrations과 migrate로 테이블 구조를 만들었으니 이제는 데이터를 넣어보는 테스트를 진행합니다.

실행하면 아래처럼 Python 프롬프트로 전환됩니다:
```python
Python 3.x.x (default, ...) on ...
Type "help", "copyright", "credits" or "license" for more information.
(InteractiveConsole)
>>>
```

Django의 ORM을 사용해서 CRUD 동작 테스트:
```python
from polls.models import Question, Choice
from django.utils import timezone

# 새 Question 객체 생성 및 저장(데이터를 직접 만들어서 DB에 저장해보기)
# CREATE (생성)
q = Question(question_text="What's new?", pub_date=timezone.now())
q.save()

# 저장된 객체 확인
# READ (조회)
q.id # 1
q.question_text # "What's new?"
q.pub_date 
# datetime.datetime(2025, 5, 14, 11, 46, 58, 717824, tzinfo=datetime.timezone.utc)

# 필드 값 수정 후 저장
# UPDATE (수정)
q.question_text = "What's up?"
q.save()

# 모든 Question 객체 조회(DB에서 데이터가 잘 들어갔는지 확인하기) 
# READ ALL (전체 조회)
Question.objects.all() 
# <QuerySet [<Question: Question object (1)>]>
# 객체 스타일로 데이터가 나온다.

# DELETE (삭제)할때는 조회먼저 한다.
q = Question.objects.get(pk=3)  # 먼저 정확한 객체를 조회함
q.delete()   # 해당 객체 삭제

# READ ALL AGAIN (삭제 후 확인)
print(Question.objects.all())  
# <QuerySet []> ← 삭제되어 빈 결과가 출력됨
```

1️⃣ Create (생성)
```python
q = Question(question_text="What's new?", pub_date=timezone.now())
q.save()
```
- `Question` 모델 인스턴스를 생성하고, `.save()`로 데이터베이스에 저장
- 결과적으로 DB에 새로운 행(row)이 추가됨  → C(Create)

2️⃣ Read (조회)
```python
Question.objects.all()
```
- 데이터베이스에 있는 모든 `Question` 객체를 조회
- 결과: `<QuerySet [<Question: Question object (1)>]>`  → R(Read)

3️⃣ Update (수정)
```python
q.question_text = "What's up?"
q.save()
```
- 기존 객체의 속성을 수정한 뒤, 다시 `.save()` 호출하여 변경사항을 DB에 반영  → U(Update)

4️⃣ Delete (삭제)
```python
q.delete()
```

쉘 종료 방법: 둘중 하나로 종료
```python
exit()
quit()
```

`__str__()`메서드
Django 쉘이나 관리자(admin) 페이지에서 `Question` 객체를 보면 아래와 같이 출력됩니다.
![[Pasted image 20250526215446.png|650]]
```bash
<Question: Question object (1)>
```
	객체원형으로 출력되어 읽기가 불편합니다.

해결 방법: `__str__()` 메서드 추가
```python
def __str__(self):
    return self.question_text
```
	이걸 모델에 추가하면 Django는 그 객체를 문자열로 보여줄 때
	question_text 값으로 보여줍니다.

`polls/models.py`에 다음을 추가:
```python
class Question(models.Model):
    ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    ...
    def __str__(self):
        return self.choice_text
```
	이걸 하면 관리자 페이지와 쉘에서 객체 이름이 예쁘게 나옴.

![[Pasted image 20250526215922.png]]

```python
python manage.py shell
```

```python
>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>
```
	이렇게 "What's up?"처럼 실제 질문 내용이 보이게 됩니다!

7️⃣ Choice 연결 (1:N 관계)
	Django ORM을 사용해서 `Choice` 데이터를 생성하고, `Question`과 연결하는 방법으로 설문 질문(Question) 1번에 3개의 선택지(Choice)를 등록하고, 연결된 선택지를 확인하는 것입니다.

`질문 1번 가져오기`
```bash
q = Question.objects.get(pk=1)
```
- `Question` 모델에서 기본키(primary key, id) 가 `1`인 질문을 가져옵니다.
- 예: `q`는 `"What's up?"` 질문일 수 있어요.

`선택지 추가하기 (1:N 관계)`
```bash
q.choice_set.create(choice_text="Not much", votes=0)
```
- `q`라는 질문(1번)에 "Not much" 라는 선택지를 추가합니다.
- `votes=0` → 투표 수를 처음에는 0으로 설정합니다.
- `choice_set.create(...)`  
    → 이건 Django가 자동으로 만들어주는 연결 메서드입니다.
	    이 질문에 연결된 선택지를 추가한다는 뜻입니다.

`선택지 3개 모두 추가`
```bash
q.choice_set.create(choice_text="The sky", votes=0)
q.choice_set.create(choice_text="Just hacking again", votes=0)
```
- 같은 방식으로 추가 선택지 2개도 등록합니다.

`연결된 선택지 전체 조회`
```bash
q.choice_set.all()
```
- 이건 해당 질문(`q`)과 연결된 모든 선택지를 보여줍니다.
- 결과는 `QuerySet` 형태로 나옵니다:
```bash
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
```

❓ 왜 `choice_set`이라고 부르나요?
Django는 `ForeignKey`로 연결된 반대편 객체들을 자동으로 접근 가능한 세트(set) 로 만들어줍니다.
- 예: `Choice` 모델에서 `question = ForeignKey(Question)` 라고 선언하면,
- Django는 `question.choice_set` 이라는 방식으로 연결된 데이터를 조회할 수 있도록 만들어줍니다.

이름을 바꾸고 싶다면 `related_name` 속성을 사용할 수도 있어요.  
```bash
question = models.ForeignKey(Question, on_delete=models.CASCADE, related_name="choices")
```
- 그러면 `q.choices.all()` 로 접근할 수 있습니다.
---
8️⃣ 관리자 페이지 등록

`polls/admin.py` 열고 아래처럼 수정:
```python
from django.contrib import admin
from .models import Question, Choice

admin.site.register(Question)
admin.site.register(Choice)
```
- 관리자 페이지에서 Question을 보고, 생성/수정/삭제할 수 있습니다.

💡 관리자 페이지에서 확인하기
- `/admin/`에 접속 → 로그인 → Questions 메뉴 보임
- 질문 등록, 수정, 삭제 가능
- 질문 등록 시 달력/시간 입력 위젯 자동으로 제공

###### 🔹 Git Commit Push & Pull
[[Django_basic/Django_공식문서-깃허브업뎃용/2. Part1/Part 1#🔹 Git Commit Push & Pull]]

----
✅ 회원가입(User) 관련 모델 패턴
```python
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    nickname = models.CharField(max_length=30, unique=True)
    phone_number = models.CharField(max_length=20, blank=True)
    profile_image = models.ImageField(upload_to='profiles/', null=True, blank=True)
```

✅ 쇼핑몰 관련 모델 패턴
```python
class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    stock = models.PositiveIntegerField()
    description = models.TextField()

class Order(models.Model):
    user = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

class OrderItem(models.Model):
    order = models.ForeignKey(Order, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    quantity = models.PositiveIntegerField()
```

✅ 게시판(Post/Reply) 모델 패턴
```python
class Post(models.Model):
    title = models.CharField(max_length=100)
    content = models.TextField()
    author = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)

class Comment(models.Model):
    post = models.ForeignKey(Post, on_delete=models.CASCADE)
    content = models.TextField()
    author = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
```

✅ 이벤트/예약 시스템 모델 패턴
```python
class Event(models.Model):
    title = models.CharField(max_length=100)
    location = models.CharField(max_length=200)
    event_date = models.DateTimeField()

class Reservation(models.Model):
    event = models.ForeignKey(Event, on_delete=models.CASCADE)
    user = models.ForeignKey(CustomUser, on_delete=models.CASCADE)
    reserved_at = models.DateTimeField(auto_now_add=True)
```
