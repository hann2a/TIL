> 요약 
  redirect('articles:index')는 “네임스페이스가 articles이고 이름이 index인 URL로 302 리다이렉트”를 의미하며, PRG 패턴으로 안전하게 후속 페이지를 보여준다.

# TIL

- `articles:index`는 파일 경로가 아님
- Django의 URL 이름을 가리키는 문자열이고, 형식은 `네임스페이스:이름`.

## 네임스페이스와 이름 정의(필수)

```python
# articles/urls.py
from django.urls import path
from . import views

app_name = 'articles'                 # ← 네임스페이스
urlpatterns = [
    path('', views.index, name='index')  # ← URL 이름
]


→ 'articles:index'는 “네임스페이스 articles 안의 이름 index URL”을 의미.
```

## 프로젝트 라우팅 연결(필수)

```python
# config/urls.py
from django.urls import path, include

urlpatterns = [
    path('articles/', include('articles.urls')),
]
```

### 왜 콜론(:)을 쓰나?
여러 앱에 같은 URL 이름이 있어도 충돌 없이 구분하려고:
- articles:index, accounts:index …

### PRG 패턴(중복 제출 방지) 필수 이해
POST → redirect(302) → GET
- 새로고침 시 POST 재전송(중복 저장) 문제를 피하기 위해 render() 대신 redirect() 사용.

### 실제 동작 흐름 요약

- 폼 제출(POST) → create()에서 DB 저장
- redirect('articles:index') → 내부적으로 reverse()로 URL 계산
- 브라우저가 Location으로 GET /articles/ 요청
- index 뷰가 목록 렌더링

### 역참조 확인 팁

```python
from django.urls import reverse
reverse('articles:index')  # '/articles/' 같은 실제 경로 반환
```

템플릿에서도 동일:
```python 
<a href="{% url 'articles:index' %}">홈</a>
```

자주 나는 오류 체크리스트

- app_name = 'articles' 누락 → NoReverseMatch
- name='index' 오타
- 프로젝트 urls.py에서 include('articles.urls') 누락