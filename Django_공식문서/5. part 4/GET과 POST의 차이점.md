 🔹 GET 방식
    - 데이터를 가져올 때 사용 (조회)
    - 주소(URL)에 데이터가 포함됨 (예: ?id=123&name=kim)
    - 보안이 낮고, 주로 검색, 조회 등에 사용
◽ 예시:
    - 웹사이트에서 검색:`https://example.com/search?q=apple`
    - 블로그 글 보기: `https://blog.com/post?id=123`

---
◽ GET을 쓰는 이유
- 빠름: 
	캐싱(저장)이 가능해서 같은 요청을 하면 서버가 빠르게 응답할 수 있어요.
- 편리함: URL에 데이터가 포함되므로, 공유하거나 북마크(즐겨찾기)할 수 있습니다.
    - 예시: `https://example.com/search?q=apple` 
      → 다음에도 같은 검색 가능
- 조회 전용: 데이터를 변경하지 않고 "읽기"만 할 때 적절합니다.

</> 검색창 예시:
```html
<form action="/search/" method="get">
  <input type="text" name="query" placeholder="검색어">      <!-- 키워드 검색 -->
  <input type="text" name="category" placeholder="카테고리"> <!-- 카테고리 검색 -->
  <input type="text" name="min_price" placeholder="최소가격"> <!-- 필터 검색 -->
  <input type="text" name="max_price" placeholder="최대가격">
  <button type="submit">검색</button>
</form>
```

검색URL
```bash
/search/?query=신발&category=운동화&min_price=30000&max_price=100000
```

✨ 쿼리(Query): 어떤 시스템(주로 데이터베이스 또는 서버)에 정보를 요청하기 위해 보내는 질의나 명령어로 ?뒤부터 쿼리입니다.
? 신발 & 운동화 & 최소가격 & 최대가격 이런 명령을 보낸것입니다.

---
◽  GET의 보안 문제
- URL에 데이터 노출:
    - 브라우저 히스토리, 로그, 즐겨찾기 등에 저장될 수 있습니다.
    - 예시:`https://bank.com/account?user=kim&balance=1000000` → 이런 정보가 노출되면 안돼요.
- 데이터 길이 제한:
    - URL 길이에 제한이 있어서 많은 데이터를 보낼 수 없습니다.

---
🔹POST 방식
    - 데이터를 서버에 보낼 때 사용 (저장, 전송) 됩니다.
    - 주소(URL)에 데이터가 보이지 않는것이 특징입니다.
    - 보안이 필요할 때, 데이터 변경이 있을 때 사용합니다.
◽ 예시:
    - 회원가입:`https://example.com/signup` 
	    아이디, 비밀번호 입력 후 서버로 전송
    - 게시글 작성:`https://blog.com/newpost` 
	    제목, 내용 입력 후 저장

---
🧠 쉽게 기억하는 방법:
- GET → "가져오기" (조회)
- POST → "보내기" (저장, 전송)
---
🔹 보안이 필요할 때는?
	보안이 중요한 경우에는 POST를 사용해야합니다.
- 로그인, 개인정보 조회 등 민감한 데이터는 POST를 써야 합니다.
    - ❌ 잘못된 예:`https://example.com/login user=kim&password=1234`
      (GET 방식, 비번 노출!)
    - ⭕ 올바른 예: POST`https://example.com/login` (비번 숨김)

◽  GET을 사용할 때 보안 강화 방법
- 민감한 정보는 URL에 포함하지 않기
- HTTPS 사용: GET이라도 암호화됨 (중간에서 가로채는 걸 방지)
- 토큰 인증 사용: URL 대신 인증 토큰을 활용해서 접근 제어

---
HTTPS를 사용하려면 도메인 + SSL 인증서 + 서버 설정이 필요하고, 이것을 Docker 기반 서버에도 적용할 수 있습니다.

- HTTPS란?
	인터넷 통신을 안전하게 만드는 방법입니다.  
	사용자의 정보(비밀번호, 카드번호 등)를 암호화해서 중간에 해킹되지 않게 합니다.

- HTTPS를 쓰려면 필요한 3가지
	`도메인` : 나만의 주소 (예: myproject.com)
	`SSL 인증서`	: "이 사이트는 안전해요"라는 보안 증명서
	`서버 설정` : 이 인증서를 웹서버에 적용해야 함 (예: Nginx)

- Docker에도 적용할 수 있습니다.
	도커는 작동하는 내 프로그램을 통째로 담아서 다른 컴퓨터에서도 똑같이 돌릴 수 있게 하는 컨테이너라고 생각하면 됩니다.
	외부에서 접속하는 부분(Nginx 등)에 인증서를 적용하면  
	HTTPS도 정상 작동합니다.

---
✨ 정리하면:
- GET: 조회용, 민감한 정보 X (예: 검색, 게시글 보기)
- POST: 보안이 필요한 데이터 전송 (예: 로그인, 회원가입, 결제)
- 보안이 필요한 GET 요청? → HTTPS + 토큰 방식 활용!

즉, 조회할 때는 GET이 빠르고 효율적이지만, 보안이 필요하면 POST를 사용해야 한다!

