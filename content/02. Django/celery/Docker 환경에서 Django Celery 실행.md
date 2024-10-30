---
title: Docker 환경에서 Django Celery 실행 가이드
publish: true
tags:
---
# Docker Compose 설정
프로젝트의 전체 `docker-compose.yml` 파일에 다음 내용을 추가합니다:

```yaml
version: '3.8'

services:
  # Celery Worker 서비스
  celery_worker:
    build:
      context: ./backend
      dockerfile: Dockerfile
      args:
        - DJANGO_SETTINGS_MODULE=my_project.settings
    command: celery -A my_project worker --loglevel=info
    depends_on:
      mysql:
        condition: service_healthy
      redis:
        condition: service_healthy
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      # 추가 환경 변수
      - DJANGO_SETTINGS_MODULE=my_project.settings
      - PYTHONUNBUFFERED=1
    volumes:
      - ./backend:/app
    restart: unless-stopped
    # 선택적 리소스 제한
    deploy:
      resources:
        limits:
          cpus: '1'
          memory: 1G

  # Celery Beat 서비스 (필요한 경우)
  celery_beat:
    build:
      context: ./backend
      dockerfile: Dockerfile
    command: celery -A my_project beat --loglevel=info
    depends_on:
      - celery_worker
    environment:
      - CELERY_BROKER_URL=redis://redis:6379/0
      - DJANGO_SETTINGS_MODULE=my_project.settings
    volumes:
      - ./backend:/app
    restart: unless-stopped

  # Redis 서비스
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 5s
      timeout: 30s
      retries: 5
    volumes:
      - redis_data:/data
    restart: unless-stopped

volumes:
  redis_data:
```

# 프로젝트 구조
```
project_root/
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── manage.py
│   └── my_project/
│       ├── __init__.py
│       ├── celery.py
│       └── settings.py
└── docker-compose.yml
```

# 실행 방법
Docker Compose로 서비스를 실행합니다:

```bash
# 전체 서비스 실행
docker-compose up -d

# Celery 로그 확인
docker-compose logs -f celery_worker

# 특정 서비스만 재시작
docker-compose restart celery_worker
```

# 주의사항

1. 볼륨 마운트
   - 개발 환경: 소스 코드 변경 사항이 바로 반영되도록 볼륨을 마운트합니다
   - 운영 환경: 성능과 보안을 위해 필요한 파일만 복사하는 것이 좋습니다

```yaml
# 개발 환경
volumes:
  - ./backend:/app

# 운영 환경
volumes:
  - ./backend/media:/app/media
  - ./backend/static:/app/static
```

2. 환경변수 관리
   - 민감한 정보는 `.env` 파일이나 시크릿으로 관리합니다

```yaml
environment:
  - CELERY_BROKER_URL=${CELERY_BROKER_URL}
  - DATABASE_URL=${DATABASE_URL}
```

3. 리소스 제한
   - Worker 수와 시스템 리소스를 고려하여 제한을 설정합니다

```yaml
command: celery -A my_project worker --loglevel=info --concurrency=4
deploy:
  resources:
    limits:
      cpus: '1'
      memory: 1G
```

4. 로깅 설정
   - 로그 레벨과 포맷을 환경에 맞게 조정합니다

```yaml
command: celery -A my_project worker --loglevel=info --logfile=/var/log/celery.log
```

# 운영 환경 최적화

1. Worker 프로세스 설정
```yaml
command: >
  celery -A my_project worker
  --loglevel=info
  --concurrency=4
  --max-tasks-per-child=100
  --optimization=fair
```

2. Healthcheck 추가
```yaml
healthcheck:
  test: ["CMD", "celery", "-A", "my_project", "inspect", "ping"]
  interval: 30s
  timeout: 10s
  retries: 3
```

3. 재시작 정책
```yaml
restart_policy:
  condition: on-failure
  delay: 5s
  max_attempts: 3
  window: 120s
```

# 모니터링 설정

1. Flower 추가 (Celery 모니터링 도구)
```yaml
flower:
  build:
    context: ./backend
    dockerfile: Dockerfile
  command: celery -A my_project flower
  ports:
    - "5555:5555"
  environment:
    - CELERY_BROKER_URL=redis://redis:6379/0
  depends_on:
    - celery_worker
```

이러한 설정으로 Docker 환경에서 안정적으로 Celery를 운영할 수 있습니다.