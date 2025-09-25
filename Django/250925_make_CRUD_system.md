# TIL 

## 가장 처음 해야할 것 
- 가상 환경은 이미 잘 알고 있으므로 생략한다. 
- `가상환경 -> activate -> pip install -r requirements.txt`

## project와 app 생성 

```bash 
django-admin startproject my_pjt .
django-admin startapp articles 
```
- 앱을 만든 뒤엔 꼭 ! `project/settings.py`에 가서 `INSTALLED_APP`에 앱 이름을 추가한다. 

## table 정의하기 
`Article/models.py`
```python 
from django.db import models

# Create your models here.
class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    created_at = models.TimeField()
    updated_at = models.TimeField()
```
## migrate 하기 
```bash 
python manage.py makemigrations articles 
python manage.py migrate
```

## templates 만들기 
- `앱/templates/앱에 html 파일을 생성한다. 
- 필요한 페이지는 `index.html`, `new.html`, `detail.html`, `edit.html`

## 로직의 흐름 
- `urls.py`에 app_name = 'articles'라 정의한다. 
  - 이미 `프로젝트/urls.py`에 `from django/urls import path, include`가 되어있고 
  - urlpatterns에 `path('articles/', include('articles.urls')),`가 추가 되어 있어야 한다. 

- `urls.py`에 `path('', views.index, name='index')` 추가
  - `views.index` 는 `views`파일의 `index 함수`를 호출하는 거, `name='index'`는 나중에 `url`을 잘 찾아오기 위해서. 
- `views.py`에 `def index()` 생성 
  ```python 
  # 전체 게시글 조회(1) 후 메인 페이지 응답(2)
  def index(request):
      # 1. DB에 전체 게시글을 조회
      articles = Article.objects.all()

      # 2. 전체 게시글 목록을 템플릿과 함께 응답
      context = {
          'articles': articles,
      }
      return render(request, 'articles/index.html', context)
  ```
- `context`를 넘겨주는 이유는 템플릿(.html)에서 'articles'라는 이름으로 `queryset`을 사용할 수 있게 되기 때문이다. 
- 즉 하나씩 조회하는 for문을 돌릴 수 있게 된다. 