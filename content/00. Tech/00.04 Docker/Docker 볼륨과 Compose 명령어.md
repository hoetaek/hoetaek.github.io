---
title: Docker Compose
publish: false
tags:
---
# 볼륨(Volume)

## Bind Mounts
호스트 시스템과 도커 컨테이너 간의 파일 시스템을 연결하는 방법이다.

```mermaid
graph LR
    A[호스트 파일시스템] -->|Bind Mount| B[도커 컨테이너]
    B -->|데이터 공유| A
```

### 기본 문법
```yaml
# compose.yml
services:
  web:
    volumes:
      - ./src:/var/www/html   # 호스트:컨테이너
      - ./logs:/var/log/nginx
```

### 일반적인 사용 사례
```yaml
services:
  php:
    volumes:
      # 소스 코드 마운트
      - ./backend:/var/www/html
      # 설정 파일 마운트
      - ./php.ini:/usr/local/etc/php/conf.d/custom.ini
      # 로그 디렉토리 마운트
      - ./logs:/var/log/php
```

# Docker Compose 명령어

## exec vs run 차이점

### exec 명령어
```bash
# 실행 중인 컨테이너에서 명령어를 실행한다
docker compose exec php composer install
```
- 기존 실행 중인 컨테이너를 사용한다
- 서비스의 현재 상태를 유지한다
- 컨테이너가 실행 중이어야 한다

### run 명령어
```bash
# 새 컨테이너를 생성하여 명령어를 실행한다
docker compose run --rm php composer update
```
- 새로운 컨테이너를 생성한다
- 독립적인 환경에서 실행된다
- `--rm` 옵션으로 실행 후 자동 삭제가 가능하다

## 주요 사용 예시

### 1. 개발 환경에서의 사용
```bash
# 의존성 설치
docker compose exec php composer install

# 데이터베이스 마이그레이션
docker compose exec php php artisan migrate

# 일회성 명령 실행
docker compose run --rm php php artisan make:controller UserController
```

### 2. 테스트 실행
```bash
# 기존 컨테이너에서 테스트
docker compose exec php php artisan test

# 격리된 환경에서 테스트
docker compose run --rm php php artisan test
```

# Compose 파일 규칙

## 파일 이름 변경
최신 Docker 버전에서는 다음 순서로 Compose 파일을 찾는다:
1. compose.yml
2. compose.yaml
3. docker-compose.yml
4. docker-compose.yaml

## 기본 구조
```yaml
# compose.yml
services:
  php:
    build: 
      context: ./docker/php
    volumes:
      - ./:/var/www/html
    
  nginx:
    image: nginx:alpine
    volumes:
      - ./:/var/www/html
      - ./docker/nginx/conf.d:/etc/nginx/conf.d
```

# 실무 팁과 주의사항

## 1. 볼륨 사용 시 주의사항
- 성능: 대용량 파일 시스템 마운트 시 성능 저하가 발생할 수 있다
- 권한: 호스트와 컨테이너의 사용자 권한을 맞춰야 한다
- 경로: 절대 경로보다 상대 경로 사용을 권장한다

```yaml
# 권장하는 방식
volumes:
  - ./logs:/var/log/nginx  # 상대 경로 사용

# 피해야 할 방식
volumes:
  - /absolute/path:/var/log/nginx  # 절대 경로는 이식성이 떨어진다
```

## 2. 명령어 사용 팁
```bash
# 개발 환경
docker compose exec php php artisan config:cache

# 일회성 작업
docker compose run --rm php composer update

# 디버깅
docker compose exec php php artisan tinker
```

# 결론

1. Bind Mount는:
   - 개발 환경에서 코드 변경 사항을 바로 반영할 수 있다
   - 설정 파일을 쉽게 관리할 수 있다
   - 로그 파일을 호스트에서 직접 접근할 수 있다

2. exec와 run은:
   - 각각의 사용 목적이 다르다
   - exec는 기존 환경을 유지해야 할 때 사용한다
   - run은 격리된 환경이 필요할 때 사용한다

3. Compose 파일은:
   - 최신 버전에서는 compose.yml을 우선한다
   - 하위 호환성을 위해 docker-compose.yml도 지원한다