# 왜 `request.user`에 값이 생기는가?

---

## 1. 브라우저가 쿠키를 보냄
- 사용자가 로그인하면 Django가 **`sessionid`** 라는 쿠키를 브라우저에 심습니다.  
- 이후 요청마다 브라우저는 이 쿠키를 들고 서버에 요청을 보냅니다.  
- 로그인하지 않았거나 로그아웃했다면 쿠키가 없거나 무효합니다.  

---

## 2. `SessionMiddleware`
- 들어온 쿠키(`sessionid`)를 보고 서버에 저장된 **세션 데이터**를 꺼냅니다.  
- 이 데이터를 `request.session`(딕셔너리 비슷한 객체)에 담아줍니다.  

---

## 3. `AuthenticationMiddleware`
- `request.session` 안에 로그인 정보가 있는지 확인합니다.  
- 특히 두 가지 키를 봅니다:
  - **`_auth_user_id`** : 로그인한 유저의 PK(ID)  
  - **`_auth_user_backend`** : 어떤 인증 백엔드를 썼는지 (예: `ModelBackend`)  
- 값이 있으면 DB에서 해당 유저 객체를 꺼내와서  
  `request.user = User 인스턴스` 로 넣습니다.  
- 값이 없으면  
  `request.user = AnonymousUser()` 로 둡니다.  

👉 즉, **개발자가 직접 세팅하는 게 아니라**  
미들웨어가 세션을 근거로 **자동으로 `request.user`를 채워줍니다.**

---

## 4. 로그인할 때는 무슨 일이?
- `auth_login(request, user)` 를 호출하면 Django가:
  1. 세션 키를 새로 발급(rotate) → 보안 강화  
  2. 세션에 `'_auth_user_id'`, `'_auth_user_backend'` 저장  
  3. 응답에 `Set-Cookie: sessionid=...` 내려보냄  

- 그 다음 요청부터는 위 과정 덕분에 `request.user`가 자동 복원됩니다.  
- 참고: 같은 요청 안에서도 `request.user`는 즉시 그 유저로 바뀝니다.  

---

## 5. 필요한 설정 (없으면 `request.user`가 비어 있음)

### INSTALLED_APPS
```python
'django.contrib.auth',
'django.contrib.sessions',
```

### MIDDLEWARE (순서 중요)
```python
'django.contrib.sessions.middleware.SessionMiddleware',
'django.contrib.auth.middleware.AuthenticationMiddleware',
```

### (선택) 커스텀 유저 모델
```python
AUTH_USER_MODEL = 'accounts.User'
```

### 인증 백엔드
```python
AUTHENTICATION_BACKENDS = [
    'django.contrib.auth.backends.ModelBackend',
]
```

# ✅ 최종 정리
### 로그인 성공
세션에 유저 정보 저장 → 브라우저에 쿠키 심음 → 다음 요청 때 쿠키 기반으로 미들웨어가 유저 복원 → request.user = User 인스턴스

### 로그인 안 함
request.user = AnonymousUser