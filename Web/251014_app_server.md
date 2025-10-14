# 동기
- 요청을 보낸 후 응답이 올 때까지 대기 
- 작업을 순차적으로 처리 
```python 
response = requests.get('https://api.com/data') # <-response가 늦게 오면 
print(response.text) # <- 이 구문은 response가 올 때까지 실행되지 않고 기다림 
```
# 비동기 
- 요청을 보낸 후 응답을 기다리지 않고 다른 작업을 수행 
- 동시에 여러 요청을 처리 
```python
async def async_example(request):
    async with aiohttp.ClientSession() as session:
        # 외부 API 호출 (비동기)
        get_api_data = session.get("https://api.com/data")  
        # 동시에 다른 작업 수행
        print("다른 작업 하는 중...")
        await asyncio.sleep(1)
        
        # 이제 API 응답 받기
        async with await get_api_data as response:
            data = await response.json()

    return JsonResponse({
        "message": "비동기 요청 완료!",
        "api": data["user"]
    })
```
---

# 동기와 비동기 차이 
- 동기 방식이 구조 단순, 디버깅 쉬움, 안정적 작동 
- 비동기 방식은 한 번에 여러 개를 처리할 수 있어 전체적인 성능을 올려주지만 버그가 발생하기 쉬움 

## 그럼 보통 동기, 비동기를 어떤 상황에 사용할까? 
동기 
- 파일 읽기/쓰기(서버 내부 작업)
  - 서버 안에서는 보통 속도가 빨라 단순 순차 처리가 안정적 
- 초기 설정, 프로그램 시작
  - 순서가 중요함 
- 디버깅, 로직 확인 

비동기 
- 웹 요청
- UI 작업 
- 대규모 I/O 작업 
- 실시간 서비스 

### 정리 
- 백엔드(Django, Flask 등) → 내부 연산은 동기, 외부 요청(API)은 비동기
- 프론트엔드(JS) → 화면 응답성 위해 대부분 비동기 (fetch, axios, setTimeout)
- 데이터 처리 스크립트 → 순차적 계산 위주라 동기

# Gunicorn 
- Django나 Flask 같은 웹 프레임워크 코드를 “진짜 서버”로 띄워주는 역할
```scss
[클라이언트 브라우저]
        ↓
     [Nginx]      ← 보안·로드밸런싱·정적파일
        ↓
    [Gunicorn]    ← WSGI 서버 (Python 실행)
        ↓
     [Django]     ← 웹 프레임워크 (비즈니스 로직)
```
- Nginx : 앞단에서 요청을 받아 분배 (보안, 캐시, 정적파일)
- Gunicorn : Python 코드(Django 앱)를 실제로 실행하는 중간 서버
- Django : Gunicorn이 호출하는 애플리케이션 로직

## Gunicorn의 실행 구조 
```rust
1️⃣ gunicorn 실행
2️⃣ myproject.wsgi:application 불러옴
3️⃣ 요청을 받을 worker 프로세스 여러 개 생성
4️⃣ 클라이언트 요청이 오면 -> WSGI 규약에 맞춰 Django에 전달
5️⃣ Django가 응답(HttpResponse)을 만들면 -> Gunicorn이 클라이언트에 전달
```