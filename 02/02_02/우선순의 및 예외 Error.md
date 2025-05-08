### 🔹 연산자의 우선순위 
	하나의 계산식에 여러 연산자가 섞여 있을 때,  
	어떤 연산을 먼저 계산할지 정해진 규칙을 말합니다.

🔍 해석: 
	수학에서도 "괄호 먼저", "곱셈 먼저" 같은 계산 순서가 있듯이,  
	파이썬도 똑같이 연산자마다 우선 계산해야 할 순서가 정해져 있습니다.

###### ‼️ 우선순위 순서 (중요한 순서부터)
| 순위  | 연산자                              | 의미       | 설명          |
| --- | -------------------------------- | -------- | ----------- |
| 1   | 괄호 `()`                          | 그룹 묶기    | 가장 먼저 계산    |
| 2   | 거듭제곱 `**`                        | 제곱       | 곱하기보다 먼저 계산 |
| 3   | 곱셈/나눗셈/몫/나머지 `*`, `/`, `//`, `%` | 곱하거나 나누기 | 덧셈/뺄셈보다 우선  |
| 4   | 덧셈/뺄셈 `+`, `-`                   | 더하고 빼기   | 가장 마지막 계산   |
✅ 외우기 쉽게!!  괄호( ) → 거듭제곱 `**` → 곱나몫나 `*,/,//,%` → 덧뺄 `+ -`

---
</>예시코드:  아래 코드를 연산하면?
```python
result = 2 + 3 * 4
print(result) # 14
```

</>예시코드:  그러나 괄호를 사용하면?
```python
result = (2 + 3) * 4
print(result) # 20
```

</>예시코드: 괄호 없이 복합 연산
```python
result = 2 + 3 * 4 - 1 ** 2 
# 1 * 1 = 1
# 3 * 4 =12 
# 2 + 12 -1 = 13

print(resul) # 13
```

</>예시코드: 괄호를 이용해 우선순위 변경
```python
result = (2 + 3) * (4 - 1) ** 2

# (2+3)
# 4-1
# 5* 9 = 45

print(resul) # 45
```

---
###### ◽ 에러 메시지 종류 
| 코드                         | 에러 종류               | 설명                              |
| -------------------------- | ------------------- | ------------------------------- |
| `print(1 / 0)`             | `ZeroDivisionError` | 0으로 나누면 안 돼요! (수학에서도 정의되지 않음)   |
| `print("hi" + 3)`          | `TypeError`         | 문자열과 숫자는 서로 더할 수 없어요 (타입이 안 맞음) |
| `int("abc")`               | `ValueError`        | 문자 `"abc"`는 숫자로 바꿀 수 없어요        |
| `print(x)`                 | `NameError`         | `x`라는 변수를 만들지 않았는데 쓰려고 해서 오류예요  |
| `lst = [1]; print(lst[5])` | `IndexError`        | 리스트에 5번째 값이 없어요 (범위 밖)          |
| `d = {}; print(d["key"])`  | `KeyError`          | 딕셔너리에 `"key"`라는 항목이 없어요         |

---
### 🔹 TypeError 예외 

✅ 예외 처리란?
	프로그램을 실행할 때 예상치 못한 에러(오류)가 발생할 수 있는데,  그때 프로그램이 멈추지 않고 안전하게 처리할 수 있도록 만드는 기능입니다.

◽ `TypeError` 예외

</>예시코드: 숫자와 문자열 연산 에러
``` python
	# 숫자와 문자열은 연산할수 없어요.
	print("hi" + 3)  # TypeError
	
	# 예외처리로 에러 해결
	try:
		print("hi" + 3)
	except TypeError:
		print("숫자와 문자열은 연산할수 없어요.")
```

🖨️ 출력결과:
```python
TypeError: can only concatenate str (not "int") to str
```

💡 기억하기
	에러를 미리 대비해서, 사용자가 잘못 입력해도 프로그램이 멈추지 않고  
	친절한 메시지를 출력합니다.

---
◽ ZeroDivisionError 예외

</>예시코드:  0으로 나누기
```python
print(10 / 0)
```

❌ 🖨️ 출력결과: (Error발생)
```python
ZeroDivisionError: division by zero
```

🔍 해설:
	10을 0으로 나누려고 하면 수학적으로 정의되지 않아서 
	**ZeroDivisionError** 에러가 발생합니다.

⭕ 안전하게 처리하려면? 예외처리를 해줍니다.

</>예시코드:  0으로 나누기 안전하게 처리하기
```python
	try: # 이 안의 코드를 실행해본다(시도할때)
	    print(10 / 0) # ❌ 여기서 오류 발생 (ZeroDivisionError)
	except ZeroDivisionError: #예외일때
	    print("0으로 나눌 수 없습니다.") # 예외 메시지를 출력
```

🖨️ 출력결과:
```python
0으로 나눌 수 없습니다.
```

🔍 해설:
	`try` 블록에서 에러가 발생하면,
	`except` 블록으로 넘어가서
    에러 메시지를 출력하고 프로그램은 멈추지 않습니다!

---
◽ 디버깅: 에러를 찾고 고치는 과정에서 자주 사용하는 방법

</> 예시코드: `Exception as e` 사용
```python
	try:
	    print("hi" + 3)
	except Exception as e:
	    print("에러 종류:", type(e).__name__)
	    print("에러 메시지:", e)
```

🖨️ 출력결과:
```python
	에러 종류: TypeError
	에러 메시지: can only concatenate str (not "int") to str
```

💡 이렇게 하면 어떤 에러든 에러 이름과 메시지를 자동으로 확인할 수 있어요!

---
</> 예시코드:
```python
try:
	data = {
		"user": {
			"name": "Tom",
			"score": 100
		}
	}
	average = data.get("user").get("total") / 0
except Exception as e:
	print("에러종류:", type(e).__name__ )
	print("에러 메시지:", e)
```

🖨️ 출력 결과:
```python
에러 종류: TypeError 
에러 메시지: unsupported operand type(s) for /: 'NoneType' and 'int'
```

💡 None / 0 에러발생:
	total를 찾을수가 없어서 None
	None/0  타입이 다르다

💡 왜 `ZeroDivisionError`가 아니고 `TypeError`인가요?
	0으로 나누면 `ZeroDivisionError`가 납니다.
	그런데 여기서는 먼저 `"total"` 키가 없어서 `None`이 나오고, 그걸 나누려고 했기 때문에,  "0으로 나누기 전에" 먼저 `None / 0`이라는 잘못된 연산이 발생합니다. 그래서 먼저 발생한 `TypeError`가 발생된 겁니다.

---
조건문(`if`)과 예외처리(`try-except`)는 목적이 다릅니다.  
파이썬에서는 "예상할 수 있는 상황은 `if`, 예상할 수 없는 에러는 `try-except`"로 구분해서 사용하는 것이 가장 바람직합니다.

---
1️⃣ 조건문으로 처리 (예방 중심)
```python
user = {"name": "Tom"}

if "age" in user:
    print("나이:", user["age"])
else:
    print("나이 정보 없음")
```

✅ **결과**: 안전하게 실행됨  
	`"age"` 키가 없더라도 미리 확인해서 에러 없음

---
2️⃣ 예외처리로 처리 (복구 중심)
```python
user = {"name": "Tom"}

try:
    print("나이:", user["age"])
except KeyError:
    print("나이 정보 없음")
```

✅ **결과**: 똑같이 잘 실행됨  
	하지만 `KeyError`가 **실제로 발생한 후에** 처리함

----
◽ try 블록에는 에러가 발생할 가능성이 있는 코드를 적습니다.

</> 하나더 예시
```python
try:
    x = int("10")     # 정수 변환 → 성공
    y = int("hello")  
    # 문자열을 숫자로 변환 → 💥 여기서 ValueError 발생 가능
    
    result = x / y  # 이 줄은 실행되지 않음 (앞에서 에러났으므로)
    
except ValueError:
    print("숫자로 변환할 수 없습니다.")
```

---
### 🔹 문자열 연산자와의 우선순위 비교

❓  문자열 연산 vs 숫자 연산, 왜 주의해야 할까?
	파이썬은 **왼쪽에서 오른쪽**으로 연산을 진행하며,  
	덧셈(+) 연산자는 숫자용 + 문자열용이 각각 다르게 정의되어 있습니다.
	따라서 문자열과 숫자를 섞은 계산식에서는 연산 순서나 괄호가 없으면 예상치 못한 에러가 발생할 수 있습니다.

🔍 해설: 
	문자열과 숫자는 절대 자동으로 변환되지 않습니다.
	문자열 + 숫자 + 문자열처럼 혼합된 수식에서는 왼쪽부터 순서대로 평가되므로,  `str()`, 괄호, f-string 같은 처리 없이는 **TypeError**가 자주 발생합니다.


</>예시코드: 올바른 형변환 후 문자열 연결 (복합 구조)
``` python
name = "Eve"
score1 = 85
score2 = 90

print("학생: " + name + " / 총점: " + str(score1 + score2) + "점")
#만약 str(score1) + str(score2) 이렇게쓰면 8590 으로나오나요? 8590
```

🖨️ 출력결과:
```python
학생: Eve / 총점: 175점
```

🔍 해설:
	`score1 + score2`를 먼저 계산한 뒤, `str()`로 형변환
	이후 문자열끼리 연결되므로 오류 없이 실행

---
조건:
	단, 금액과 세금은 변수에 넣어서 출력해주세요.
	추가, f-string으로 사용해서도 출력해보세요.

</> 수정 예시 1: `str()`로 형변환
```python
price = 1200
tax = 100

total = price + tax -> 1300

print("총 결제 금액: " + str(total) + "원")
```

</> 수정 예시 2: `f-string` 사용
```python
price = 1200
tax = 100

print(f'총 결제 금액: {price + tax}원')
```

---
📝 문제1] 아래 코드를 실행하면 오류가 발생합니다. 오류 원인을 파악하고, `총점이 150점 이상이면 "우수"`를 출력하도록 수정하세요.
```python
student = {
    "name": "Mina",
    "math": 80,
    "english": 75
}

if student["math"] + student["english"] >= 150:
    print("이름: " + student["name"] + ", 총점: " + student["math"] + student["english"])
```

✅ 정답 코드:
```python
student = {
    "name": "Mina",
    "math": 80,
    "english": 75
}

total = student["math"] + student["english"]

if total >= 150:
	print(f'이름: {student["name"]}, 총점: {total} -> 우수')
```

🖨️ 출력 결과:
```python
이름: Mina, 총점: 155 → 우수
```

---
🌟사전지식: 
	 어떤수라도 2로 나누면 나머지가 0 또는 1로 떨어진다.

```python
num = int(input("숫자를 입력하세요."))

# 짝수입니다.
# 홀수입니다.
```

✅ 정답:
```python
num = int(input("숫자를 입력하세요.")) # 31

if num % 2  == 0 # 짝수니?
# -> num / 2 ---나머지는?
else:
    print("홀수입니다.")
```

---
어떤수 % 7 ---- 0~6

weekdays = ["일", "월", "화", "수", "목", "금", "토"]

input =("오늘은 무슨 요일인가요? (예: 월, 화, 수, 목, 금, 토, 일)")
변수명. index( ) 인덱스 번호가 호출됩니다.

| 요일  | 숫자  |
| --- | --- |
| 일요일 | 0   |
| 월요일 | 1   |
| 화요일 | 2   |
| 수요일 | 3   |
| 목요일 | 4   |
| 금요일 | 5   |
| 토요일 | 6   |

오늘은 { } 요일 이고,
{"21" } 일 후에는 { }요일입니다.

✅ 정답코드: 다른분들 코드도 참고해주세요. 
```python
# 요일을 숫자로 매칭하는 리스트
weekdays = ["일", "월", "화", "수", "목", "금", "토"]

# 사용자 입력
today_name = input("오늘은 무슨 요일인가요? (예: 월, 화, 수): ")
after_days = int(input("며칠 후를 계산할까요? (예: 5): "))

# 요일 계산
today = weekdays.index(today_name) # 일~토 문자열
day_index = (today + after_days) % 7 # 0~6

# 결과 출력
print(f"오늘은 {weekdays[today]}요일입니다.")
print(f"{after_days}일 후는 {weekdays[day_index]}요일입니다.")
```

</> 세일님코드:
```python
weekdays = ["일", "월", "화", "수", "목", "금", "토"] 

today_weekday = input("오늘은 무슨 요일인가요? ") 
after_day = int(input("몇일 후 요일이 궁금하신가요? ")) 

next_weekday = weekdays[(weekdays.index(today_weekday) + after_day) % 7] 

print(f""" 오늘은 {today_weekday}요일이고, {after_day}일 후에는 {next_weekday}요일입니다. """)
```

</>정훈님 코드:
```python
weekdays = int(input("숫자를 입력하세요."))
if weekdays >= 7: 
	weekdays = weekdays % 7 days= ["일요일","월요일","화요일","수요일","목요일","금요일","토요일"] 

print("오늘은 ", days[weekdays], "이고") 
weekdays2 = int(input("숫자를 입력하세요.")) 
weekdays= weekdays2+weekdays 

if weekdays >= 7: 
	weekdays = weekdays % 7 

print(weekdays2,"일 후에는",days[weekdays],"요일입니다.")
```

</> 원욱님 코드:
```python
weekdays = ["일", "월", "화", "수", "목", "금", "토"] 
day = int(input("오늘은 무슨 요일인가요? 번호를 눌러주세요. " "0. 일요일" + " 1. 월요일" + " 2. 화요일" + "3. 수요일" + " 4. 목요일" + "5. 금요일" + " 6. 토요일")) 

addday = int(input("몇 일 후의 요일을 알고싶으신지 숫자를 눌러주세요. " )) 
futureday = (day + addday) % 7 

print(f'오늘은 {weekdays[day]}요일이고, {addday}일 후에는 {weekdays[futureday]}요일입니다.')
```
