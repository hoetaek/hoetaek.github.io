---
publish: false
tags:
---
Django 데이터베이스 마이그레이션 초기화하기

# 개요
개발 과정에서 Django 마이그레이션 파일이 너무 많아지거나 꼬여서 초기화가 필요한 경우가 있습니다. 이 가이드는 마이그레이션을 안전하게 초기화하는 방법을 설명합니다.

# 초기화 절차

## 1. 마이그레이션 파일 삭제
모든 앱의 마이그레이션 파일을 삭제합니다(`__init__.py` 제외):

```bash
# Python 마이그레이션 파일 삭제
find . -path "*/migrations/*.py" -not -name "__init__.py" -delete

# 컴파일된 마이그레이션 파일 삭제
find . -path "*/migrations/*.pyc" -delete
```

## 2. 데이터베이스 초기화

### PostgreSQL 사용 시
```bash
# PostgreSQL
dropdb database_name
createdb database_name
```

### MySQL 사용 시
```sql
DROP DATABASE database_name;
CREATE DATABASE database_name;
```

### SQLite 사용 시
```bash
# db.sqlite3 파일 삭제
rm db.sqlite3
```

## 3. 마이그레이션 재생성
```bash
# 마이그레이션 파일 생성
python manage.py makemigrations

# 마이그레이션 적용
python manage.py migrate
```

# 주의사항

1. 데이터 백업
   ```bash
   # PostgreSQL 데이터 백업
   pg_dump database_name > backup.sql
   
   # MySQL 데이터 백업
   mysqldump -u username -p database_name > backup.sql
   ```

2. 운영 환경 주의
   - 운영 환경에서는 절대 실행하지 않습니다
   - 개발/테스트 환경에서만 사용합니다

3. 가상환경 확인
   ```bash
   # 가상환경 활성화 상태 확인
   echo $VIRTUAL_ENV
   
   # 필요시 가상환경 활성화
   source venv/bin/activate
   ```

# 문제 해결

## 일반적인 문제

### 1. 특정 앱만 초기화
```bash
# 특정 앱의 마이그레이션만 삭제
find ./myapp/migrations -type f -not -name "__init__.py" -delete

# 해당 앱만 마이그레이션 재생성
python manage.py makemigrations myapp
```

### 2. 의존성 문제 해결
```python
# settings.py에서 앱 순서 확인
INSTALLED_APPS = [
    'django.contrib.auth',
    'django.contrib.contenttypes',
    # ... 기타 기본 앱들
    'myapp',  # 의존성 있는 앱들은 순서 주의
]
```

# 실행 스크립트
편의를 위한 쉘 스크립트 예시:

```bash
#!/bin/bash
# reset_migrations.sh

# 가상환경 활성화
source venv/bin/activate

# 마이그레이션 파일 삭제
find . -path "*/migrations/*.py" -not -name "__init__.py" -delete
find . -path "*/migrations/*.pyc" -delete

# 데이터베이스 재생성 (PostgreSQL 예시)
dropdb database_name
createdb database_name

# 마이그레이션 재생성
python manage.py makemigrations
python manage.py migrate

echo "마이그레이션 초기화 완료"
```

# 개발 환경별 사용법

## 1. 로컬 개발 환경
```bash
# 스크립트 실행 권한 부여
chmod +x reset_migrations.sh

# 스크립트 실행
./reset_migrations.sh
```

## 2. Docker 환경
```bash
# 컨테이너 내부에서 실행
docker-compose exec web bash -c "find . -path \"*/migrations/*.py\" -not -name \"__init__.py\" -delete"
docker-compose exec web python manage.py makemigrations
docker-compose exec web python manage.py migrate
```

# 결론

마이그레이션 초기화는 다음 상황에서 유용합니다:
1. 개발 초기 단계에서 모델 구조가 자주 변경될 때
2. 테스트 환경을 깨끗하게 초기화하고 싶을 때
3. 마이그레이션 파일이 너무 많아져서 정리가 필요할 때

하지만 운영 환경에서는 절대 사용하지 말아야 하며, 개발 환경에서도 데이터 백업 후 신중하게 실행해야 합니다.

출처: [Simple is Better Than Complex](https://simpleisbetterthancomplex.com/tutorial/2016/07/26/how-to-reset-migrations.html)