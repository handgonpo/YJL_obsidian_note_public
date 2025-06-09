🔹 Django 튜토리얼 Part8 – 재사용 가능한 앱 만들기

###### 📖 공식 문서 링크:  
🔗 [https://docs.djangoproject.com/ko/stable/intro/reusable-apps/](https://docs.djangoproject.com/ko/stable/intro/reusable-apps/)

- `polls` 앱을 패키지로 만들어 `pip install`로 설치 가능하게 하기
- 다른 프로젝트에서도 import해서 쓸 수 있도록 구성

✅ 디렉토리 구조 만들기
기존 `polls` 앱만 분리해서 재사용할 것이므로  
프로젝트 폴더 외부에 패키지용 디렉토리를 따로 만듭니다:
```bash
mkdir django_polls # 폴더를 만든다
# polls앱을 복사하여 django_polls에 넣는다
# polls앱의 이름을 django_polls로 변경한다.
```

📁 이제 디렉토리 구조는 다음과 같습니다:
```
project_root/
├── mysite/         # 프로젝트 설정 (settings.py, urls.py 등)
├── accounts/       # accounts 앱
├── polls/          # polls 앱 (→ 패키지화 시 삭제 가능)
├── static/         # 공통 static(css, js, images)
├── templates/      # 공통 base.html, admin 커스터마이징 등
├── django-polls/   # polls 앱을 패키지화한 프로젝트 폴더
│   └── django_polls/  # 복사된 앱 (polls → django_polls)
│   │    ├── __init__.py
│   │    ├── apps.py  
│   │    ├── urls.py
│   │    ├── models.py
│   │    ├── views.py
│   │    ├── templates/
│   │    └── static/
│   ├── README.rst      # 여기에 작성
│   ├── pyproject.toml  # 여기에 작성
│   ├── MANIFEST.in     # 여기에 작성
├── manage.py
```

✅ apps.py 수정
`django_polls/apps.py` 파일에서 `name`과 `label` 수정:
```python
from django.apps import AppConfig

class PollsConfig(AppConfig):
    default_auto_field = "django.db.models.BigAutoField"
    name = "django_polls"
    label = "polls"
```
polls 앱을 `django_polls` 폴더로 복사해서, 그 안에서 `django_polls` 라는 이름으로 리네이밍한 후 수정하는 `apps.py` 파일을 말합니다.


✅ README.rst 작성
```
============
django-polls
============

django-polls is a Django app to conduct web-based polls.

Quick start
-----------

1. Add "django_polls" to INSTALLED_APPS
2. Include the polls URLs in urls.py
3. Run migrations
4. Visit /polls/ to participate
```

✅ LICENSE 파일 작성 (예: BSD 또는 MIT)
`django_polls/LICENSE` 파일 생성: 예: MIT License Template

✅ pyproject.toml 작성
`django_polls/pyproject.toml` 생성:
```
[build-system]
requires = ["setuptools>=69.3"]
build-backend = "setuptools.build_meta"

[project]
name = "django-polls"
version = "0.1"
dependencies = ["django>=5.0"]
description = "A Django app to conduct web-based polls."
readme = "README.rst"
requires-python = ">= 3.10"
authors = [{name = "Your Name", email = "you@example.com"}]
classifiers = [
    "Framework :: Django",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3.10"
]

[project.urls]
Homepage = "https://example.com/"
```

✅ MANIFEST.in 작성
`django_polls/MANIFEST.in` 파일 생성:
```recursive-include django_polls/static *
recursive-include django_polls/templates *
```

✅ (선택) 문서 디렉토리 추가
```
mkdir django_polls/docs
```
	django_polls폴더 안에 docs라는 폴더를 추가한다.

✅ 패키지 빌드
```bash
python -m pip install build  # build 모듈 없으면 설치
```

✅ 빌드할 폴더로 이동
```bash
cd django_polls
```

✅ 패키지 빌드
```bash
python -m build  # dist/ 폴더 생성됨
```

빌드 성공시:
```bash
Successfully built django-polls
Successfully created 'dist/django_polls-0.1.tar.gz'
Successfully created 'dist/django_polls-0.1-py3-none-any.whl'
```

`django_polls/dist/` 폴더 안에 다음이 생깁니다:
```
dist/
├── django_polls-0.1.tar.gz
└── django_polls-0.1-py3-none-any.whl
```

✅ 설치 테스트 (가상환경 권장)
```bash
python -m pip install dist/django_polls-0.1.tar.gz
```

- 단순 빌드 테스트:
```bash
(venv) youjung@DESKTOP-PJCRMMU:~/Django_first_for/django-polls$ python -m pip install dist/django_polls-0.1.tar.gz
```
- 패키지화된 django_polls가 정상적으로 설치되는지 확인
- 설치 이후 `INSTALLED_APPS = ["django_polls.apps.PollsConfig"]`에서 인식되는지 확인
- 즉, 개발 중인 가상환경에서 설치 테스트하는 방법입니다.

- 완전한 검증: 새로운 가상환경 만들어서 테스트:
```bash
cd ~/Desktop/test_env/
python -m venv venv
source venv/bin/activate
pip install /path/to/django-polls/dist/django_polls-0.1.tar.gz
```
- 현재 개발 환경의 설정이나 `polls` 폴더와 충돌하지 않는 깨끗한 상태에서 설치 확인 가능
- 실제 외부에서 사용하는 것과 같은 환경 재현 가능

✅ settings.py 추가:
```python
INSTALLED_APPS = [
    "django_polls.apps.PollsConfig",
    ...
]
```

✅ urls.py 추가: `mysite/urls.py 또는 프로젝트 루트 urls.py`
```python
urlpatterns = [
    path("polls/", include("django_polls.urls")),
    ...
]
```

✅ 기존 경로로 변경
```bash
cd ..
(venv) youjung@DESKTOP-PJCRMMU:~/Django_first_for
```

✅ VSCode터미널에서 아래 명령어로 설치:
```bash
python -m pip install django_polls/dist/django_polls-0.1.tar.gz
```

✅ 설치가 잘 되었는지 확인:
```bash
pip show django_polls
```

✅ `mysite/urls.py`
```python
urlpatterns = [
    path("admin/", admin.site.urls),
    path("", views.index, name="index"),
    path("accounts/", include("accounts.urls")),
    path("accounts/", include("django.contrib.auth.urls")),
    path("polls/", include("django_polls.urls")),# 이것사용
    # path("polls/", include("polls.urls")), 반드시 주석처리
]
```

✅ `mysite/settings.py`
```python
INSTALLED_APPS = [
    # polls 반드시 주석처리
    "django_polls.apps.PollsConfig", # 이것으로 사용
]
```

✅ 실행 확인
```bash
python manage.py runserver
```
---
🔹 PyPI란?
	PyPI (Python Package Index) 는  
	`pip install 패키지명` 명령어로 설치되는 모든 Python 패키지의 저장소입니다.
- 공식 주소: [https://pypi.org/](https://pypi.org/)
- 예:
    - `pip install django` → PyPI에서 Django를 받아오는 것
    - `pip install requests` → PyPI에서 requests 받아오기

```bash
pip install twine
```
- 업로드 도구인 `twine`을 설치합니다.
- `setup.py`나 `pyproject.toml`로 만든 패키지를 PyPI에 안전하게 업로드하는 툴입니다.

```bash
python -m twine upload dist/*
```
- `dist/` 폴더에 있는 `.tar.gz`와 `.whl` 파일을 PyPI 서버로 업로드합니다.
- 즉, `pip install django-polls`처럼 다른 사람들이 설치할 수 있게 됩니다.

✅ 업로드 전에 계정 필요
- PyPI 계정 생성:  
    → [https://pypi.org/account/register/](https://pypi.org/account/register/)
    
- API 토큰 생성 (추천)  
    → [https://pypi.org/manage/account/](https://pypi.org/manage/account/)
    
- 업로드 시 아래와 같이 로그인 요청이 나옵니다:
```bash
username: __token__
password: <API token>
```

실제 배포 전에 테스트용 서버에서 연습해볼 수 있어요:
```bash
python -m twine upload --repository testpypi dist/*
```
- 주소: [https://test.pypi.org/](https://test.pypi.org/)
- 설치도 가능:
```bash
pip install --index-url https://test.pypi.org/simple/ django-polls
```
