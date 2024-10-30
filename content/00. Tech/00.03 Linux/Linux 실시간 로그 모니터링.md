---
publish: true
tags:
---

# 기본 로그 모니터링

## tail 명령어 기본 사용법
```bash
# 기본 사용법 (마지막 10줄 표시)
tail path/to/file.log

# 실시간 모니터링
tail -f path/to/file.log

# 특정 줄 수 지정
tail -n 100 -f path/to/file.log
```

# 고급 모니터링 기법

## 1. 여러 파일 동시 모니터링
```bash
# 여러 파일 동시에 보기
tail -f app.log error.log

# 파일명 표시와 함께 보기
tail -f -v app.log error.log
```

## 2. 패턴 매칭과 함께 사용
```bash
# grep과 함께 사용
tail -f access.log | grep "ERROR"

# 대소문자 구분 없이 검색
tail -f access.log | grep -i "error"

# 컬러 하이라이트
tail -f access.log | grep --color "ERROR"
```

## 3. 시간 기반 필터링
```bash
# 현재 시간 기준 필터링
tail -f access.log | grep "$(date +%Y-%m-%d)"

# 특정 시간대 로그만 보기
tail -f access.log | grep "2024-01-"
```

# 실용적인 사용 예시

## 1. 웹 서버 로그 모니터링
```bash
# Nginx 액세스 로그 모니터링
tail -f /var/log/nginx/access.log

# 특정 IP의 요청만 보기
tail -f /var/log/nginx/access.log | grep "192.168.1.1"

# 404 에러만 보기
tail -f /var/log/nginx/access.log | grep "404"
```

## 2. 애플리케이션 로그 모니터링
```bash
# Django 애플리케이션 로그
tail -f /var/log/django/app.log

# 에러 로그만 필터링
tail -f /var/log/django/app.log | grep -E "ERROR|CRITICAL"
```

## 3. 시스템 로그 모니터링
```bash
# 시스템 로그 보기
tail -f /var/log/syslog

# 보안 관련 로그만 보기
tail -f /var/log/auth.log
```

# 고급 팁과 트릭

## 1. 로그 하이라이팅
```bash
# 여러 패턴 색상 구분
tail -f app.log | grep --color -E 'ERROR|WARN|INFO|DEBUG|$'
```

## 2. 로그 필터링과 저장
```bash
# 실시간 모니터링하면서 파일로 저장
tail -f access.log | tee -a filtered.log

# 필터링된 내용만 저장
tail -f access.log | grep "ERROR" | tee -a errors.log
```

## 3. 타임스탬프 추가
```bash
# 각 라인에 타임스탬프 추가
tail -f access.log | while read line; do echo "$(date +%Y-%m-%d_%H:%M:%S) $line"; done
```

# 문제 해결

## 1. 로그 파일이 회전되는 경우
```bash
# -F 옵션 사용으로 파일 회전 추적
tail -F /var/log/app.log
```

## 2. 큰 로그 파일 처리
```bash
# 마지막 1MB만 보기
tail -c 1M app.log

# 특정 바이트 이후부터 보기
tail -c +1000 app.log
```

# 성능 고려사항

1. 리소스 사용
   - 파일 크기가 매우 큰 경우 메모리 사용 주의
   - 많은 파일을 동시에 모니터링할 때 CPU 사용량 고려

2. 로그 회전
   - logrotate와 함께 사용 시 주의
   - -F 옵션 활용

3. 디스크 I/O
   - 높은 로그 발생률의 영향 고려
   - 필요한 경우 로깅 레벨 조정

# 보안 고려사항

1. 권한 관리
```bash
# 로그 파일 권한 확인
ls -l /var/log/app.log

# 적절한 권한 설정
chmod 644 /var/log/app.log
```

2. 민감한 정보
```bash
# 민감한 정보 필터링
tail -f access.log | grep -v "password"
```

# 결론

효율적인 로그 모니터링을 위한 체크리스트:
1. 적절한 tail 옵션 선택
2. 필요한 필터링 적용
3. 리소스 사용 모니터링
4. 보안 고려사항 확인

이러한 도구와 기법을 활용하여 시스템과 애플리케이션의 상태를 효과적으로 모니터링할 수 있습니다.