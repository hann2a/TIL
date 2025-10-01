0. 이 공부를 한 이유

장고의 request 객체가 어떤 형태인지, <form> 전송 시 값들이 어떻게 들어오는지 궁금해서.

특히 GET / POST 방식에 따라 request.GET과 request.POST가 어떻게 채워지는지 이해하기 위해.

1. <form> 태그와 요청 흐름
(POST 방식)
```html
<form action="/accounts/login/" method="POST">
  <input type="text" name="username">
  <input type="password" name="password">
  <input type="submit" value="로그인">
</form>
```

submit 시 → 브라우저가 입력값을 HTTP body에 담아서 보냄.

(GET 방식)
```html
<form action="/accounts/login/" method="GET">
  <input type="text" name="username">
  <input type="password" name="password">
  <input type="submit" value="로그인">
</form>
```

submit 시 → 브라우저가 입력값을 URL 쿼리스트링(?username=...&password=...)에 붙여서 보냄.

2. 브라우저가 보내는 실제 HTTP 요청 (예시)
POST 요청
```
POST /accounts/login/ HTTP/1.1
Host: localhost:8000
Content-Type: application/x-www-form-urlencoded
Content-Length: 27

username=sohee&password=1234
```
GET 요청
```
GET /accounts/login/?username=sohee&password=1234 HTTP/1.1
Host: localhost:8000
```
3. Django에서 어떻게 보이냐?
POST
```html
def login_view(request):
    if request.method == "POST":
        print(request.POST)
        <!--<QueryDict: {'username': ['sohee'], 'password': ['1234']}>--> 
```
GET
```html
def login_view(request):
    if request.method == "GET":
        print(request.GET)
        <!--<QueryDict: {'username': ['sohee'], 'password': ['1234']}>--> 
```
4. 정리

POST → <form> 값들이 요청 body에 담겨서 request.POST에 들어감.

GET → <form> 값들이 쿼리스트링으로 붙어서 request.GET에 들어감.

둘 다 Django에서 QueryDict 형태로 제공되므로, 뷰 함수 안에서

```python
request.POST["username"]
request.GET["username"]
```

같은 방식으로 접근 가능.