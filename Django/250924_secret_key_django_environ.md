# Django 환경변수 관리 (django-environ)

## 0. secret_key 재발급 
```bash
python -c "from django.core.management.utils import get_random_secret_key as g; print(g())"
```

## 1. 설치
```bash
pip install django-environ
```

## 2. .env.example
```env
SECRET_KEY=your-secret-key-here
DEBUG=True
```

## 3. .gitignore
```gitignore
# Secrets
.env
```

## 4. settings.py
```python 
import os
from pathlib import Path
import environ

BASE_DIR = Path(__file__).resolve().parent.parent

env = environ.Env(
    DEBUG=(bool, False)
)

# .env 파일 로드
environ.Env.read_env(os.path.join(BASE_DIR, ".env"))

SECRET_KEY = env("SECRET_KEY")
DEBUG = env("DEBUG")

DATABASES = {
    "default": env.db("DATABASE_URL", default=f"sqlite:///{BASE_DIR / 'db.sqlite3'}")
}
```

## 5. 실제 .env (로컬 전용)
```env
SECRET_KEY=django-insecure-abcdef1234567890
DEBUG=True
```