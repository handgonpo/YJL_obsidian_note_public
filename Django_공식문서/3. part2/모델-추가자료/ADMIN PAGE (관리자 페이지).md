◽ 관리자 페이지란?
- Django의 가장 강력한 기능 중 하나는 자동으로 생성되는 관리자(admin) 인터페이스입니다.
- 이 관리자 페이지는 모델에서 정의한 메타데이터(구조 정보)를 읽어와서  
    → 빠르고 쉽게 데이터를 관리할 수 있는 인터페이스를 만들어줍니다.
- 사이트의 신뢰된 사용자(관리자)만 이 페이지에서 콘텐츠를 관리할 수 있습니다.
- 내부 관리 도구로 사용하는 것이 추천됩니다. (즉, 일반 사용자용이 아님)

◽ 관리자 페이지 사용을 위한 설정 방법

1️⃣ 관리자 기능 활성화 (앱 등록)
`settings.py`의 `INSTALLED_APPS`에 다음을 포함해야 합니다:
```python
'django.contrib.admin',
```
![[Pasted image 20250525193103.png|370]]

추가 가능한 Django 내장 앱 예시
`django.contrib.sites`	
	여러 도메인을 하나의 Django 프로젝트에서 관리할 때 사용
`django.contrib.flatpages`	
	DB에서 관리되는 정적 페이지 시스템
`django.contrib.humanize`	
	숫자/날짜를 사람 친화적으로 포맷 (예: 1000 → 1,000)`
`django.contrib.sitemaps`	
	자동 sitemap.xml 생성
`django.contrib.redirects`	
	URL 리디렉션 관리
`django.contrib.admindocs`	
	관리자 페이지에 문서 표시

`polls`	:앱 디렉토리 이름으로 앱을 등록	
	기본 설정(PollsConfig)을 자동 사용
`polls.apps.PollsConfig`	:앱 구성 클래스를 명시적으로 등록	
	고급 설정(앱 이름, 레이블 등) 가능

2️⃣ URL 연결하기 (admin 주소 지정)
`urls.py`에 다음과 같이 작성합니다:
```python
from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),  # 이 줄이 admin 페이지를 연결함
]
```
![[Pasted image 20250525193207.png|400]]

3️⃣ 관리자(admin) 사용자 생성: 사용자명, 이메일, 비밀번호 입력
```bash
python manage.py createsuperuser
```
Django의 관리자 페이지(`/admin`)에 로그인할 수 있는 ‘최고 권한 관리자 계정’을 만드는 명령어입니다.
![[Pasted image 20250525193412.png|400]]

4️⃣ 관리자 페이지가 생성됨:
![[Pasted image 20250525190905.png]]

