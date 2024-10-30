---
title: docker로 dev 환경에서 실행하는 법
publish: true
tags:
---
Django 프로젝트 환경 설정 완벽 가이드

# 1. 프로젝트 구조

## 1.1 기본 디렉토리 구조
```
my_project/
├── config/
│   ├── __init__.py
│   ├── settings/
│   │   ├── __init__.py
│   │   ├── base.py
│   │   ├── local.py
│   │   └── production.py
│   ├── urls.py
│   ├── wsgi.py
│   └── asgi.py
├── scripts/
│   ├── start_dev.sh
│   └── start_prod.sh
├── .env
├── .env.example
├── requirements.txt
├── Dockerfile
└── docker-compose.yml
```

# 2. 환경 변수 설정

## 2.1 django-environ 설정
```python
# config/settings/base.py
from pathlib import Path
import environ

# env 초기화
env = environ.Env(
    DEBUG=(bool, False),
    ALLOWED_HOSTS=(list, []),
    CORS_ORIGIN_WHITELIST=(list, []),
)

# Build paths
BASE_DIR = Path(__file__).resolve().parent.parent.parent

# Take environment variables from .env file
environ.Env.read_env(BASE_DIR / '.env')

# 기본 설정
SECRET_KEY = env('SECRET_KEY')
DEBUG = env('DEBUG')
ALLOWED_HOSTS = env('ALLOWED_HOSTS')
CORS_ORIGIN_WHITELIST = env('CORS_ORIGIN_WHITELIST')

# 데이터베이스 설정
DATABASES = {
    'default': env.db(),
}

# 이메일 설정
EMAIL_CONFIG = env.email_url('EMAIL_URL', default='smtp://user:password@localhost:25')
vars().update(EMAIL_CONFIG)
```

## 2.2 환경변수 파일
```env
# .env
DEBUG=True
SECRET_KEY=your-secret-key
DJANGO_SETTINGS_MODULE=config.settings.local

# 호스트 설정
ALLOWED_HOSTS=example.com,api.example.com,localhost,127.0.0.1
CORS_ORIGIN_WHITELIST=http://127.0.0.1:5173,http://localhost:5173

# 데이터베이스 URL
DATABASE_URL=postgres://user:password@db:5432/dbname

# 이메일 URL
EMAIL_URL=smtp://user:password@smtp.gmail.com:587

# Redis URL (캐싱용)
REDIS_URL=redis://redis:6379/1
```

# 3. 실행 스크립트

## 3.1 개발 환경 스크립트
```bash
#!/bin/sh
# scripts/start_dev.sh

echo "Running Development Server"

# 데이터베이스 마이그레이션
echo "Making migrations..."
python manage.py makemigrations

echo "Applying migrations..."
python manage.py migrate --no-input

# 개발 서버 실행
echo "Starting development server..."
python manage.py runserver 0.0.0.0:8000
```

## 3.2 프로덕션 환경 스크립트
```bash
#!/bin/sh
# scripts/start_prod.sh

echo "Running Production Server"

# 정적 파일 수집
echo "Collecting static files..."
python manage.py collectstatic --no-input

# 데이터베이스 마이그레이션
echo "Making migrations..."
python manage.py makemigrations

echo "Applying migrations..."
python manage.py migrate

# Gunicorn 실행
echo "Starting Gunicorn..."
gunicorn config.wsgi:application -b 0.0.0.0:8000
```

# 4. Docker 설정

## 4.1 Dockerfile
```dockerfile
FROM python:3.12-slim

# 환경 변수 설정
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1

# 작업 디렉토리 설정
WORKDIR /app

# 시스템 패키지 설치
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        postgresql-client \
        build-essential \
        python3-dev \
        libpq-dev \
    && rm -rf /var/lib/apt/lists/*

# 프로젝트 의존성 설치
COPY requirements.txt /tmp/requirements.txt
COPY ./scripts /scripts

# Python 가상환경 설정 및 패키지 설치
RUN python -m venv /py && \
    /py/bin/pip install --upgrade pip && \
    /py/bin/pip install -r /tmp/requirements.txt && \
    rm -rf /tmp && \
    adduser --disabled-password --no-create-home django-user && \
    mkdir -p /app/static && \
    chown -R django-user:django-user /app && \
    chmod -R +x /scripts

# 프로젝트 파일 복사
COPY . .

# 정적 파일 디렉토리 권한 설정
RUN chown -R django-user:django-user /app/static

# 사용자 전환
USER django-user

# 환경 변수 PATH 설정
ENV PATH="/scripts:/py/bin:$PATH"

# 포트 노출
EXPOSE 8000
```

## 4.2 Docker Compose
```yaml
version: "3.9"

services:
  db:
    container_name: ${PROJECT_NAME}_db
    image: postgres:14-alpine
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    environment:
      - POSTGRES_DB=${DB_NAME}
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER} -d ${DB_NAME}"]
      interval: 5s
      timeout: 5s
      retries: 5

  backend:
    container_name: ${PROJECT_NAME}_backend
    build:
      context: .
      dockerfile: Dockerfile
    command: /scripts/start_dev.sh  # 개발환경
    # command: /scripts/start_prod.sh  # 프로덕션 환경
    volumes:
      - .:/app
      - static_volume:/app/static
    env_file:
      - .env
    ports:
      - "8000:8000"
    depends_on:
      db:
        condition: service_healthy

volumes:
  postgres_data:
  static_volume:
```

# 5. requirements.txt

```txt
# 기본 의존성
Django>=4.2.0,<5.0.0
django-environ>=0.11.2
django-cors-headers>=4.3.0
djangorestframework>=3.14.0

# 데이터베이스
psycopg2-binary>=2.9.9

# 서버
gunicorn>=21.2.0
```

# 6. 배포 전 체크리스트

## 6.1 환경 변수 확인
- [ ] DATABASE_URL 설정이 올바른가?
- [ ] ALLOWED_HOSTS가 프로덕션 도메인을 포함하는가?
- [ ] CORS_ORIGIN_WHITELIST가 올바르게 설정되었는가?
- [ ] SECRET_KEY가 안전한 값으로 설정되었는가?
- [ ] DEBUG가 프로덕션에서 False로 설정되었는가?

## 6.2 보안 설정
```python
# config/settings/production.py
from .base import *

DEBUG = False

# HTTPS 설정
SECURE_SSL_REDIRECT = True
SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
SECURE_CONTENT_TYPE_NOSNIFF = True

# 세션 설정
SESSION_COOKIE_SECURE = True
CSRF_COOKIE_SECURE = True
```

## 6.3 성능 설정
```python
# config/settings/production.py

# 캐시 설정
CACHES = {
    'default': env.cache('REDIS_URL')
}

# 정적 파일 설정
STATIC_ROOT = BASE_DIR / 'static'
STATIC_URL = '/static/'
STATICFILES_STORAGE = 'django.contrib.staticfiles.storage.ManifestStaticFilesStorage'
```

# 7. 자주 발생하는 문제 해결

## 7.1 데이터베이스 연결 문제
```bash
# 데이터베이스 연결 상태 확인
docker-compose exec db psql -U $DB_USER -d $DB_NAME -c "\l"

# Django 데이터베이스 상태 확인
docker-compose exec backend python manage.py dbshell
```

## 7.2 정적 파일 문제
```bash
# 정적 파일 수집 강제 실행
docker-compose exec backend python manage.py collectstatic --no-input --clear

# 정적 파일 권한 확인
docker-compose exec backend ls -la /app/static
```

## 7.3 마이그레이션 문제
```bash
# 마이그레이션 상태 확인
docker-compose exec backend python manage.py showmigrations

# 마이그레이션 초기화 (주의: 데이터 손실 가능)
docker-compose exec backend python manage.py migrate app_name zero
```

# 8. 결론

이 가이드는 Django 프로젝트의 기본적인 설정부터 배포까지의 전체 과정을 다룹니다. 환경 변수 관리를 위해 django-environ을 사용하고, Docker를 통한 컨테이너화로 일관된 개발/배포 환경을 구성합니다. 각 환경(개발/프로덕션)에 맞는 설정을 분리하고, 보안과 성능을 고려한 설정을 포함하고 있습니다.

주의할 점은 프로젝트의 요구사항에 따라 이 설정들을 적절히 수정해야 한다는 것입니다. 특히 보안 설정과 성능 최적화는 프로젝트의 특성에 맞게 조정되어야 합니다.