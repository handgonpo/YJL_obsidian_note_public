배경  
당신은 한 온라인 쇼핑몰의 백엔드 개발자입니다.  
마케팅팀이 다음과 같은 요청을 해왔습니다:

✅ "월별 매출 추이"와 ✅ "카테고리별 매출 합계"를 대시보드에 표시해 주세요.

실제로는 이런 데이터를 CSV나 JSON 파일로 받아와 저장하고,  
이를 Django API로 제공하면 됩니다.

제공된 JSON 원시 데이터 (`sales_data.json`)
```json
// sales_data.json
[
  { "order_id": "A001", "category": "의류",     "price": 30000,  "quantity": 2, "order_date": "2024-05-10" },
  { "order_id": "A002", "category": "전자제품", "price": 150000, "quantity": 1, "order_date": "2024-05-11" },
  { "order_id": "A003", "category": "의류",     "price": 45000,  "quantity": 1, "order_date": "2024-06-01" }
]
```

문제: `sales/utils.py` – JSON 로드 함수 완성
```python
import os
import json
from django.conf import settings

def load_sales_data():
    """프로젝트 루트의 sales_data.json 파일을 읽어와 Python 객체로 반환"""
    path = os.path.join(settings.______, 'sales_data.json')
    with open(path, encoding='utf-8') as f:
        data = json.______(f)
    return data
```
- `settings.______`
- `json.______`

---
`sales/views.py` – 두 개의 API 뷰 완성
```python
from django.http import JsonResponse
from .utils import load_sales_data

def monthly_sales(request):
    """
    월별 매출 추이 집계
    결과 예시: [
      { "month": "2024-05", "total": 210000 },
      { "month": "2024-06", "total":  45000 }
    ]
    """
    data = load_sales_data()
    result = {}   # {"2024-05": 0, "2024-06": 0, …}

    for item in data:
        # 1) order_date에서 앞 7글자(YYYY-MM)만 잘라 key로 사용
        month = item["____"][:7]

        # 2) 매출 = price * quantity
        sales = item["____"] * item["____"]

        # 3) 누적 계산
        if month in result:
            result[month] += sales
        else:
            result[month] = sales

    # 4) 최종 리스트로 변환
    response = []
    for m, total in sorted(result.items()):
        response.append({"month": m, "total": total})

    return JsonResponse(response, safe=____)


def category_sales(request):
    """
    카테고리별 매출 합계 집계
    결과 예시: [
      { "category": "의류",     "total": 105000 },
      { "category": "전자제품", "total": 150000 }
    ]
    """
    data = load_sales_data()
    result = {}   # {"의류": 0, "전자제품": 0, …}

    for item in data:
        # 1) category
        cat = item["____"]

        # 2) 매출 = price * quantity
        sales = item["____"] * item["____"]

        # 3) 누적 계산
        if cat in result:
            result[cat] += sales
        else:
            result[cat] = sales

    # 4) 최종 리스트로 변환
    response = []
    for c, total in sorted(result.items()):
        response.append({"category": c, "total": total})

    return JsonResponse(response, safe=____)
```
---
URL 연결
```python
from django.urls import path
from . import views

urlpatterns = [
    path('sales/monthly/',  views.monthly_sales),
    path('sales/category/', views.category_sales),
]
```
- `config/urls.py` 에 `path('api/', include('sales.urls'))` 설정을 추가하세요.

---
✅ 정답 코드:
```python
# sales/utils.py
import os
import json
from django.conf import settings

def load_sales_data():
    """프로젝트 루트의 sales_data.json 파일을 읽어와 Python 객체로 반환"""
    path = os.path.join(settings.BASE_DIR, 'sales_data.json')
    with open(path, encoding='utf-8') as f:
        data = json.load(f)
    return data
```

✔️ 의사코드:
```python
함수 load_sales_data():
    1. 설정 객체(settings)에서 프로젝트 루트 디렉터리(BASE_DIR) 경로를 가져온다.
    2. 파일 이름 'sales_data.json'을 프로젝트 루트 경로와 결합하여 전체 파일 경로(path)를 생성한다.
    3. 해당 파일 경로(path)에 있는 파일을 UTF-8 인코딩으로 연다.
    4. 열린 파일 객체(f)를 json.load() 함수에 전달하여 JSON 문자열을 Python 객체(data)로 파싱한다.
    5. 파싱된 Python 객체(data)를 호출자에게 반환(return)한다.

```


```python
# sales/views.py
from django.http import JsonResponse
from .utils import load_sales_data

def monthly_sales(request):
    """
    월별 매출 추이 집계
    결과 예시: [
      { "month": "2024-05", "total": 210000 },
      { "month": "2024-06", "total":  45000 }
    ]
    """
    data = load_sales_data()
    result = {}   # {"2024-05": 0, "2024-06": 0, …}

    for item in data:
        # 1) order_date에서 앞 7글자(YYYY-MM)만 잘라 key로 사용
        month = item["order_date"][:7]

        # 2) 매출 = price * quantity
        sales = item["price"] * item["quantity"]

        # 3) 누적 계산
        if month in result:
            result[month] += sales
        else:
            result[month] = sales

    # 4) 최종 리스트로 변환
    response = []
    for m, total in sorted(result.items()):
        response.append({"month": m, "total": total})

    return JsonResponse(response, safe=False)


def category_sales(request):
    """
    카테고리별 매출 합계 집계
    결과 예시: [
      { "category": "의류",     "total": 105000 },
      { "category": "전자제품", "total": 150000 }
    ]
    """
    data = load_sales_data()
    result = {}   # {"의류": 0, "전자제품": 0, …}

    for item in data:
        # 1) category
        cat = item["category"]

        # 2) 매출 = price * quantity
        sales = item["price"] * item["quantity"]

        # 3) 누적 계산
        if cat in result:
            result[cat] += sales
        else:
            result[cat] = sales

    # 4) 최종 리스트로 변환
    response = []
    for c, total in sorted(result.items()):
        response.append({"category": c, "total": total})

    return JsonResponse(response, safe=False)
```

✔️의사코드:
```python
# monthly_sales 함수 의사코드 (문장 설명)

1. 브라우저에서 요청이 들어오면, load_sales_data() 함수를 호출하여 sales_data.json 파일의 모든 주문 데이터를 Python 리스트 형태로 불러옵니다.
    
2. 월별 매출을 저장할 빈 딕셔너리 result 를 만듭니다.
    
3. 불러온 데이터 리스트를 하나씩 순회(for item in data:)하면서:
    
    - order_date 문자열에서 앞의 7글자(`YYYY-MM`)를 잘라 month 변수에 담습니다.
        
    - price 값에 quantity 값을 곱해 이 주문의 매출액(sales)을 계산합니다.
        
    - 만약 month 키가 이미 result 딕셔너리에 있으면, 그 값에 방금 계산한 sales 를 더해주고,  
        그렇지 않으면 month 키를 새로 만들어 sales 를 할당합니다.
        
4. 월별로 누적된 매출이 담긴 result 딕셔너리를 키(월) 순서대로 정렬하여,
    
    - 빈 리스트 response 에 각 (month, total) 쌍을 { "month": month, "total": total } 형태의 딕셔너리로 하나씩 추가합니다.
        
5. 완성된 response 리스트를 JsonResponse 에 담아 safe=False 옵션과 함께 반환합니다.
    
---
# category_sales 함수 의사코드 (문장 설명)

1. 요청이 들어오면 역시 load_sales_data() 함수를 통해 모든 주문 데이터를 불러옵니다.
    
2. 상품별 매출을 저장할 빈 딕셔너리 result 를 만듭니다.
    
3. 주문 하나하나를 순회하면서:
    
    - category 값을 꺼내 cat 변수에 저장합니다.
        
    - price * quantity 연산으로 해당 주문의 매출액 sales 를 계산합니다.
        
    - cat 키가 이미 result 에 있으면 그 값에 sales 를 더하고,  
        없으면 cat 키를 새로 만들어 sales 값을 할당합니다.
        
4. 상품명 순으로 정렬된 result 아이템을 순회하며,
    
    - 빈 리스트 response 에 { "category": c, "total": total } 형태로 하나씩 추가합니다.
        
5. 완성된 response 리스트를 JsonResponse(response, safe=False) 로 반환합니다.
```

```python
# sales/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('sales/monthly/',  views.monthly_sales),
    path('sales/category/', views.category_sales),
]
```

`config/urls.py` (프로젝트 최상위)
```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include('sales.urls')),
]
```

Insomnia 테스트
```
GET http://127.0.0.1:8000/api/sales/monthly/
GET http://127.0.0.1:8000/api/sales/category/
```

✅ 응답 월별 매출
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

✅ 응답  카테고리별 매출
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
