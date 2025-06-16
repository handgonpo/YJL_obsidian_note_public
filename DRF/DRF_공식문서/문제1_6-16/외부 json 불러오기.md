두개의 외부 json파일을 imsomnia로 불러서 잘 불러지는지 테스트 하세요.

sample_data.json
```json
{"address": {"building": "1007", "coord": [-73.856077, 40.848447], "street": "Morris Park Ave", "zipcode": "10462"}, "borough": "Bronx", "cuisine": "Bakery", "grades": [{"date": {"$date": 1393804800000}, "grade": "A", "score": 2}, {"date": {"$date": 1378857600000}, "grade": "A", "score": 6}, {"date": {"$date": 1358985600000}, "grade": "A", "score": 10}, {"date": {"$date": 1322006400000}, "grade": "A", "score": 9}, {"date": {"$date": 1299715200000}, "grade": "B", "score": 14}], "name": "Morris Park Bake Shop", "restaurant_id": "30075445"}
```

test.json
```json
{ "users": [ { "userId": 1, "firstName": "AAAAA", "lastName": "as23", "phoneNumber": "123456", "emailAddress": "AAAAA@test.com", "homepage": "https://amogg.tistory.com/1" }, { "userId": 2, "firstName": "BBBB", "lastName": "h5jdd", "phoneNumber": "123456", "homepage": "https://amogg.tistory.com/2" }, { "userId": 3, "firstName": "CCCCC", "lastName": "2dhbs", "phoneNumber": "33333333", "homepage": "https://amogg.tistory.com/3" }, { "userId": 4, "firstName": "DDDDD", "lastName": "bacasd", "phoneNumber": "222222222", "homepage": "https://amogg.tistory.com/4" }, { "userId": 5, "firstName": "EEEEE", "lastName": "asdfasdf", "phoneNumber": "111111111", "homepage": "https://amogg.tistory.com/5" } ] }
```

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path('', include('sample.urls')),
]
```

```python
from django.urls import path
from . import views

urlpatterns = [
    path('sample/', views.get_sample_data),
]
```

```python
import os
import json
from django.http import JsonResponse
from django.conf import settings

def get_sample_data(request):
    file_path = os.path.join(settings.BASE_DIR, 'sample_data.json')  

    try:
        with open(file_path, encoding='utf-8') as f:
            data = json.load(f)
        return JsonResponse(data, safe=False)  # 리스트 형태 응답 가능하도록 safe=False
    except FileNotFoundError:
        return JsonResponse({'error': '파일을 찾을 수 없습니다.'}, status=404)
```

```python
REST_FRAMEWORK = {
    'DEFAULT_RENDERER_CLASSES': [
       'rest_framework.renderers.JSONRenderer',
    ]
}

INSTALLED_APPS = [
    "yourapp"
]
```
