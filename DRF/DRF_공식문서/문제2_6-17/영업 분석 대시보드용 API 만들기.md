배경  
당신은 한 온라인 쇼핑몰의 백엔드 개발자입니다.  
마케팅팀이 다음과 같은 요청을 해왔습니다:

✅ "월별 매출 추이"와 ✅ "카테고리별 매출 합계"를 대시보드에 표시해 주세요.

실제로는 이런 데이터를 CSV나 JSON 파일로 받아와 저장하고,  
이를 Django API로 제공하면 됩니다.

제공된 JSON 원시 데이터 (`sales_data.json`)
```json
[
  {
    "order_id": "A001",
    "category": "의류",
    "price": 30000,
    "quantity": 2,
    "order_date": "2024-05-10"
  },
  {
    "order_id": "A002",
    "category": "전자제품",
    "price": 150000,
    "quantity": 1,
    "order_date": "2024-05-11"
  },
  {
    "order_id": "A003",
    "category": "의류",
    "price": 45000,
    "quantity": 1,
    "order_date": "2024-06-01"
  }
]
```

문제: **sales/models.py** 파일을 만들고, 아래 괄호`(____)` 부분을 채워보세요.
```python
# sales/models.py
from django.db import models

class SalesRecord(models.Model):
    # 주문 번호: 최대 (____) 글자의 문자열
    order_id = models.CharField(max_length=____)

    # 상품 카테고리: 최대 (____) 글자의 문자열
    category = models.CharField(max_length=____)

    # 단가(가격): (___________) 타입. 마이너스 안됨
    price = models.__________________________()

    # 수량: 음수가 되면 안됨 → (___________) 사용
    quantity = models.__________________________()

    # 주문 날짜: (__________) 타입을 사용
    order_date = models.____________________()

    # 문자열로 보여줄 때, (__________)를 대표값으로 사용
    def __str__(self):
        return self.___________

```

✅ 정답 코드:
```python
from django.db import models

class SalesRecord(models.Model):
    order_id = models.CharField(max_length=20)
    category = models.CharField(max_length=50)
    price = models.PositiveIntegerField()
    quantity = models.PositiveIntegerField()
    order_date = models.DateField()

    def __str__(self):
        return self.order_id
```

집계 API 만들기 (views.py)
두 가지 API를 만듭니다  
① 월별 매출 합계 ② 카테고리별 매출 합계

문제:
```python
# sales/views.py
from django.http import JsonResponse
from .models import SalesRecord
from django.db.models import Sum
from django.db.models.functions import TruncMonth

def monthly_sales_summary(request):
    # ① 월별 총 매출 집계
    result = (SalesRecord.objects
                .annotate(month=TruncMonth(______)) # 날짜 필드는?
                .annotate(sales=_______)  # 매출은 price * quantity
                .values("month")
                .annotate(total_sales=Sum(_______)) # 매출 합계?
                .order_by("month"))
    return JsonResponse(list(result), safe=False)

def category_sales_summary(request):
    # ② 카테고리별 총 매출 집계
    result = (SalesRecord.objects
                .annotate(sales=_______) # 매출 계산?
                .values("category")
                .annotate(total_sales=Sum(_______))) # 매출 합산?
    return JsonResponse(list(result), safe=False)

```

|용어|설명|
|---|---|
|`F()`|모델 필드 간 연산 가능하게 해줌. (`F("price") * F("quantity")`)|
|`TruncMonth`|날짜에서 '월'만 추출함. (`2024-05-10` → `2024-05-01`)|
|`annotate()`|임시 필드를 추가하거나 집계값을 붙일 때 사용|
|`Sum()`|특정 필드의 총합을 계산|

✅ 정답 코드 (views.py)
```python
from django.http import JsonResponse
from .models import SalesRecord
from django.db.models import Sum, F
from django.db.models.functions import TruncMonth

def monthly_sales_summary(request):
    result = (SalesRecord.objects
                .annotate(month=TruncMonth("order_date"))
                .annotate(sales=F("price") * F("quantity"))
                .values("month")
                .annotate(total_sales=Sum("sales"))
                .order_by("month"))
    return JsonResponse(list(result), safe=False)

def category_sales_summary(request):
    result = (SalesRecord.objects
                .annotate(sales=F("price") * F("quantity"))
                .values("category")
                .annotate(total_sales=Sum("sales")))
    return JsonResponse(list(result), safe=False)
```

URL 연결 sales/urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path("sales/monthly/", views.monthly_sales_summary),
    path("sales/category/", views.category_sales_summary),
]
```

config/urls.py
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path("admin/", admin.site.urls),
    path("api/", include("sales.urls")),
]
```

Insomnia 테스트
```
GET http://127.0.0.1:8000/api/sales/monthly/
GET http://127.0.0.1:8000/api/sales/category/
```

테스트 결과:
`Insomnia`로 아래의 두 API를 테스트하면 다음과 같은 결과가 나옵니다. 이 예시는 `views.py`에 있는 두 개의 뷰 함수:
- `monthly_sales_summary`
- `category_sales_summary` 을 기준으로 합니다.

✅ 1. `GET /api/sales/monthly/`
목적: 월별 매출 총합 집계

요청:
```
GET http://127.0.0.1:8000/api/sales/monthly/
```

✅ 응답 (예시) 월별 매출
```json
[
  {
    "month": "2024-05-01",
    "total_sales": 210000
  },
  {
    "month": "2024-06-01",
    "total_sales": 45000
  }
]
```

- `month`: 해당 월(1일로 자동 변환됨)
- `total_sales`: `price * quantity`의 합

✅ 2. `GET /api/sales/category/`
목적: 카테고리별 매출 총합 집계

요청:
```
GET http://127.0.0.1:8000/api/sales/category/
```

✅ 응답 (예시) 카테고리별 매출
```python
[
  {
    "category": "의류",
    "total_sales": 105000
  },
  {
    "category": "전자제품",
    "total_sales": 150000
  }
]
```

- `category`: 상품 카테고리명
- `total_sales`: 카테고리별 매출합 (`price * quantity`)