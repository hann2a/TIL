# 왜 {% url 'accounts:login' %}를 href에 쓸 수 있나?

템플릿의 url 태그가 내부적으로 **reverse()**를 호출해,
네임스페이스:이름(accounts:login) → 실제 경로 문자열(예: /accounts/login/)로 바꿔 주기 때문.

## reverse란?

- django.urls.reverse 함수.
- “이름으로 경로 찾기”를 수행하는 URL 역방향 매핑 유틸리티.
- URLconf(각 앱의 urls.py에 등록된 패턴들)를 거꾸로 탐색해서, name으로 등록된 URL의 실제 문자열을 돌려준다.

### 시그니처 요약:
```
reverse(viewname, urlconf=None, args=None, kwargs=None, current_app=None)
```
- viewname: 'accounts:login'처럼 네임스페이스 포함 이름 또는 뷰 경로
- args/kwargs: 동적 경로 변수 채울 때 사용

## 설정 전제
```python
# accounts/urls.py
from django.urls import path
from . import views

app_name = 'accounts'          # 네임스페이스
urlpatterns = [
    path('login/', views.login_view, name='login'),  # URL 이름
]
```
- 프로젝트 urls.py에서 include('accounts.urls')로 연결되어 있어야 함.

## 템플릿에서 사용
```html
<a href="{% url 'accounts:login' %}">로그인</a>
```

### 렌더링 결과(예):
```html
<a href="/accounts/login/">로그인</a>
```

### 파라미터가 있는 URL
```python
# accounts/urls.py
path('users/<int:pk>/', views.detail, name='detail')
```

### 템플릿:
```
<a href="{% url 'accounts:detail' pk=user.id %}">프로필</a>
```

### 파이썬 코드(동일 효과):
```python
from django.urls import reverse
reverse('accounts:detail', kwargs={'pk': user.id})  # "/accounts/users/42/"
```

## 자주 나는 오류

- app_name 누락, name 오타, args/kwargs 불일치 → NoReverseMatch

# 왜 reverse를 쓰나? (장점)

- URL 경로가 바뀌어도 이름만 유지하면 템플릿/코드 수정 최소화
- 하드코딩 경로(/accounts/login/) 대신 의미 기반 참조 → 유지보수 용이
- 네임스페이스로 앱 간 중복 이름 충돌 방지