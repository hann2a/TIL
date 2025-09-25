> 요약: url.py에서 `urlpattern`의 path 함수에 들어가는 `<int:pk>/`는 converter로 타입을 제한하고, pk라는 이름으로 view에 해당 인자를 넘겨주는 것을 의미한다 


# Django URL path converter(경로 변환기) 문법

형식: `<converter:name>/`

- converter: 값의 타입을 제한하는 변환기
- name: 뷰 함수(혹은 클래스)의 인자로 넘어갈 변수 이름

그래서 '<int:pk>/'는 정수만 매칭해서 그 값을 **pk라는 이름으로** 뷰에 넘겨w준다. 

```python
# urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('<int:pk>/', views.detail),  # 예: /42/ -> views.detail(request, pk=42)
]
```

뷰에서는 이렇게 받는다. 

```python 
def detail(request, pk):
    # pk는 int 타입
    ...
```

## Django가 기본으로 제공하는 converter

- `str`: 슬래시(/) 제외한 문자열 (기본값; '<name>/'와 같음)
- `int`: 0 이상의 정수
- `slug`: 슬러그(영문자, 숫자, 하이픈, 밑줄)
- `uuid`: UUID 형식
- `path`: 슬래시 포함한 경로(하위 디렉터리까지)