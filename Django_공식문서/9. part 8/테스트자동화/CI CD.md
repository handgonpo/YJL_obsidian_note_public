`CI (Continuous Integration)` 지속적 통합
`CD(Continuous Delivery / Deployment)` 지속적 전달/배포
	코드 변경이 생기면, 자동으로 테스트하고, 자동으로 배포까지 진행되게 만드는 개발 파이프라인입니다.

`CI (지속적 통합)`: 
	개발자들이 자주 코드를 commit/push하면, 이를 자동으로 테스트 및 병합하는 과정
`CD (지속적 배포/전달)`: 
	테스트까지 통과한 코드를 자동으로 서버에 배포하거나, 배포 가능한 상태로 만드는 과정
```
1. 개발자가 코드를 수정함
2. GitHub에 push 함
3. GitHub Actions가 자동으로 실행됨
    - 1) 테스트 코드 실행
    - 2) 코드 오류 검증
    - 3) 성공 시 Docker 이미지 생성
    - 4) (선택) 서버에 자동 배포
4. 성공/실패 여부를 PR에서 확인 가능
```


🔹 Django 프로젝트에서 CI/CD 구축 순서:

- `tests.py 작성`: 
	개발 중 기능별 자동 테스트 코드를 작성합니다.  
	예: `test_model`, `test_view`, `test_form` 등
	
- VSCode 터미널에서 tests.py가 잘 실행되는지 확인:
```bash
python manage.py test
```
	
- `requirements.txt 생성`:
	`pip freeze > requirements.txt`  
	→ 가상환경에 설치된 패키지 목록을 기록합니다.  
	→ GitHub Actions에서 동일한 환경을 구성하기 위해 필요
- `.github/workflows/django.yml 작성`:
	CI 자동화 설정 파일 작성  
	→ `push`, `PR` 때 테스트 자동 실행
- `git add .` / `git commit` / `git push`:
	작성한 코드, 테스트, 설정파일을 깃허브에 올립니다
- `GitHub Actions에서 테스트 실행 확인`:
	`[Actions]` 탭에서 초록색 체크가 뜨면 성공입니다.  
	→ 실패하면 로그 보고 `tests.py`, 마이그레이션, 경로 등을 수정합니다

![[Pasted image 20250607144702.png]]

---
📁 루트 디렉토리:
```
mysite/                  
├── manage.py            
├── mysite/              
│   ├── settings.py      
│   └── urls.py          
├── polls/               ← Django 앱 폴더
│   ├── models.py        
│   ├── views.py         
│   ├── tests.py         ← 여기에 테스트 클래스 작성
├── .github/             ← GitHub Actions 관련 파일
│   └── workflows/       ← GitHub Actions 워크플로우 파일
│       └── django.yml   ← GitHub Actions 설정 파일
├── requirements.txt     ← 의존성 파일
```
- 경로를 설정하면 자동으로 VSCode에서 githubAction을 설치하라고 뜬다. 
- 요청대로 설치를 진행한다.
---
🔹yml파일이란?

YAML은 데이터 표현 언어입니다. 마크업 언어는 사람이 읽는 텍스트에 특별한 태그(tag)를 추가하여 구조를 설명하는 언어이며  마크업 언어(XML, HTML)와는 달리 데이터 구조를 표현하기 위한 간단한 포맷입니다.
- `.yml` 또는 `.yaml` 확장자를 사용합니다 (둘 다 동일하게 동작).

즉, 야물은기계가 따라야 할 명세서(설명서)입니다.

📖 공식 문서 링크:  
🔗 GitHub Actions 공식 문서 [https://docs.github.com/ko/actions/writing-workflows/quickstart](https://docs.github.com/ko/actions/writing-workflows/quickstart)

🔗 YAML 문법 가이드 [https://yaml.org/spec/1.2.2/](https://yaml.org/spec/1.2.2/)

🔗 YAML 튜토리얼 [https://learnxinyminutes.com/yaml/](https://learnxinyminutes.com/yaml/)

| 문법 요소         | 설명                         | 예시                                |
| ------------- | -------------------------- | --------------------------------- |
| key: value  . | 딕셔너리(사전)처럼 표현              | `name: Django CI`                 |
| 들여쓰기          | 계층 구조 표현 (공백 2칸 또는 4칸)     | `steps:`  <br>  `- name: Install` |
| 리스트           | `-`로 표현                    | `- python3`  <br>`- pip install`  |
| 주석            | `#`로 시작                    | `# 여기는 설명`                        |
| 문자열           | 보통 따옴표 없어도 됨, 필요하면 `"..."` | `python-version: "3.12.3"`        |
- 들여쓰기로 계층 구조를 표현합니다. (스페이스 2칸 또는 4칸)
- 중괄호 `{}` 나 대괄호 `[]` 없이 깔끔한 구문을 사용합니다.
- 리스트는 `-` 기호로 작성합니다.
```yml
fruits:
  - apple
  - banana
  - orange
```
---
❓ 어디에 사용되나요?
	
`CI/CD 도구 (예: GitHub Actions)`	
	`.github/workflows/*.yml` 파일을 사용하여 자동 테스트, 빌드, 배포 등을 설정
`Docker Compose`	
	여러 컨테이너의 설정을 docker-compose.yml에 정의
`Kubernetes`	
	배포 설정, 서비스 정의 등 모든 리소스를 YAML로 설정
`Ansible`	
	서버 자동화 설정을 YAML 파일로 관리
`Python 라이브러리 설정`	
	일부 프레임워크에서 config 파일로 사용 (예: mkdocs.yml)
`API 문서 작성(OpenAPI/Swagger)`	
	API 스펙을 정의하는 데 사용

---
GitHub Actions용 YAML 기본 구조
```yaml
name: 워크플로우이름

on: [push, pull_request]  # 언제 실행?

jobs:
  작업이름:
    runs-on: 실행환경
    steps:
      - name: 단계이름
        run: 실행할 명령어
```

---
`github yml`
```yml
name: Django CI

on:
  push:
    branches: [ "Eunice" ]
  pull_request:
    branches: [ "Eunice" ]

jobs:
  build:

    runs-on: ubuntu-latest
    strategy:
      max-parallel: 4
      matrix:
        python-version: [3.10, 3.11, 3.12]

    steps:
    - uses: actions/checkout@v4
    - name: Set up Python ${{ matrix.python-version }}
      uses: actions/setup-python@v3
      with:
        python-version: ${{ matrix.python-version }}
    - name: Install Dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Run Tests
      run: |
        python manage.py test

```


`django.yml`
```yml
name: Django CI                       # 이 워크플로우의 이름

on:
  push:
    branches: [ "Eunice" ]
  pull_request:
    branches: [ "Eunice" ]             # 이 워크플로우의 이름

jobs:                                 # 실행할 작업(Job) 정의 시작
  test:                               # job 이름: test
    runs-on: ubuntu-latest            # 가상 환경: 최신 Ubuntu

    steps:                            # 단계별로 무엇을 할지
      - uses: actions/checkout@v3

      - uses: actions/setup-python@v4
        with:
          python-version: '3.12.3'

      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt

      - name: Make migrations
        run: python manage.py makemigrations

      - name: Migrate database
        run: python manage.py migrate

      - name: Run tests
        run: python manage.py test
```

🔹 yml파일 코드리뷰

✅ 1. 코드 내려받기
```yaml
- uses: actions/checkout@v3
```

- GitHub 저장소에 있는 코드를 runner(가상머신)로 복사합니다.
- 모든 워크플로우에서 거의 필수입니다.

---
✅ 2. 파이썬 실행 환경 설정
```yaml
- uses: actions/setup-python@v4
  with:
    python-version: '3.12.3'
```

- GitHub Actions 서버에 Python 3.12.3 설치
- 우리가 사용하는 버전과 일치시켜야 합니다

---
✅ 3. 프로젝트 의존성 설치
```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
```
- 현재 사용 중인 Python 환경의 pip을 최신 버전으로 업그레이드해라
- `requirements.txt`에 명시된 Django, pytest 등 필요한 패키지를 설치
- `pip install -r requirements.txt`는 내 환경에 맞춰서 자동설치

---
✅ 4. 마이그레이션 파일 생성 (DB 스키마 준비)
```yaml
- name: Make migrations   
  run: python manage.py makemigrations
```

- `models.py`의 변경사항을 기반으로 DB에 적용할 준비 파일을 생성합니다.
- 일반적으로 로컬에서 실행하면 GitHub에는 이미 파일이 있지만, CI 환경은 새롭게 시작하기 때문에 필요합니다.

---
✅ 5. DB 테이블 실제 생성 (마이그레이션 실행)
```yaml
- name: Migrate database   run: python manage.py migrate
```

- 위에서 만든 마이그레이션 파일들을 실행해서 DB 테이블을 실제로 생성함.

---
✅ 6. 테스트 실행
```yaml
- name: Run tests   run: python manage.py test
```

- `polls/tests.py` 등에서 작성한 테스트 코드를 실행합니다.
- 통과하지 못하면 이 워크플로우는 실패 처리됩니다 ❌

---
전체 흐름
```
[push or PR 발생]
      ↓
[코드 체크아웃]
      ↓
[Python 3.12 환경 세팅]
      ↓
[필요한 패키지 설치]
      ↓
[DB 마이그레이션 파일 생성]
      ↓
[DB 테이블 생성]
      ↓
[테스트 실행]
```
