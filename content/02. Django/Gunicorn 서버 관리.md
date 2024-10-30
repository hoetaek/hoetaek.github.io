---
publish: false
tags:
---
# 기본 실행
Gunicorn을 실행하는 기본적인 방법입니다:

```bash
# 기본 실행
gunicorn myapp.wsgi:application

# 포트 지정 실행
gunicorn myapp.wsgi:application -b 0.0.0.0:8000

# 백그라운드 실행
gunicorn myapp.wsgi:application -D
```

# Worker 설정

## Worker 클래스 선택
```bash
# Sync worker (기본값)
gunicorn myapp.wsgi:application -w 4

# Gevent worker
gunicorn myapp.wsgi:application -w 4 -k gevent

# Uvicorn worker
gunicorn myapp.wsgi:application -w 4 -k uvicorn.workers.UvicornWorker
```

## Worker 개수 설정
```bash
# CPU 코어 수 * 2 + 1 권장
gunicorn myapp.wsgi:application --workers=4

# Thread worker 사용
gunicorn myapp.wsgi:application --workers=2 --threads=4
```

# 프로세스 관리

## 프로세스 종료하기
```bash
# 기본 종료 (SIGTERM)
pkill gunicorn

# 강제 종료 (SIGKILL)
ps aux | grep gunicorn | grep -v grep | awk '{print $2}' | xargs kill -9
```

## 프로세스 재시작
```bash
# Graceful 재시작 (SIGHUP)
kill -HUP `cat gunicorn.pid`
```

# 주의사항

1. 강제 종료(`kill -9`) 사용
   - 마지막 수단으로만 사용합니다
   - 진행 중인 요청이 모두 중단됩니다
   - 데이터 정합성 문제가 발생할 수 있습니다

2. 안전한 종료 순서
   ```bash
   # 1. 일반 종료 시도
   pkill gunicorn

   # 2. 잠시 대기
   sleep 5

   # 3. 남은 프로세스 확인
   ps aux | grep gunicorn

   # 4. 필요한 경우에만 강제 종료
   ps aux | grep gunicorn | grep -v grep | awk '{print $2}' | xargs kill -9
   ```

# 모니터링

## 프로세스 상태 확인
```bash
# 실행 중인 Gunicorn 프로세스 확인
ps aux | grep gunicorn

# 특정 포트 사용 확인
netstat -tulpn | grep gunicorn
```

## 로그 확인
```bash
# 액세스 로그 설정
gunicorn myapp.wsgi:application --access-logfile gunicorn-access.log

# 에러 로그 설정
gunicorn myapp.wsgi:application --error-logfile gunicorn-error.log
```