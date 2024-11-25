---
date: 2024-11-21
publish: false
tags:
---
# 파일 시스템 명령어
## 기본 탐색 및 조작
### 디렉토리 관련
```bash
pwd     # 현재 작업 디렉토리 출력
cd      # 디렉토리 이동
  cd ..          # 상위 디렉토리로 이동
  cd ~           # 홈 디렉토리로 이동
  cd /path/to/dir # 절대 경로로 이동
ls      # 디렉토리 내용 출력
  ls -l          # 자세한 정보 출력
  ls -la         # 숨김 파일 포함 출력
  ls -lh         # 파일 크기 사람이 읽기 쉽게 출력
```

### 파일 조작
```bash
cp      # 파일 복사
  cp file1 file2         # file1을 file2로 복사
  cp -r dir1 dir2        # 디렉토리 재귀적 복사
mv      # 파일 이동/이름변경
  mv file1 file2         # file1을 file2로 이동/이름변경
  mv file1 dir/          # file1을 dir로 이동
rm      # 파일 삭제
  rm file               # 파일 삭제
  rm -r dir            # 디렉토리 재귀적 삭제
  rm -f file           # 강제 삭제
```

# 텍스트 처리 명령어
## 파일 내용 확인
```bash
cat     # 파일 내용 출력
  cat file              # 파일 내용 전체 출력
  cat -n file          # 줄 번호 포함 출력
less    # 페이지 단위로 파일 내용 확인
head    # 파일 앞부분 출력
  head -n 5 file       # 상위 5줄 출력
tail    # 파일 뒷부분 출력
  tail -f log          # 실시간 로그 모니터링
```

## 텍스트 검색 및 처리
### grep 명령어
```bash
grep    # 텍스트 패턴 검색
  grep pattern file     # file에서 pattern 검색
  grep -i pattern file  # 대소문자 구분 없이 검색
  grep -r pattern dir   # 디렉토리 내 재귀적 검색
  grep -v pattern file  # pattern을 포함하지 않는 줄 검색
```

### awk 명령어
```bash
# 기본 문법
awk 'pattern {action}' file

# 주요 예제
# 1. 특정 컬럼 출력
awk '{print $1}' file           # 첫 번째 컬럼 출력
awk '{print $1,$3}' file        # 1,3번 컬럼 출력

# 2. 필드 구분자 지정
awk -F: '{print $1}' /etc/passwd   # : 구분자로 첫 번째 필드 출력

# 3. 조건 처리
awk '$3 > 100 {print $1,$3}' file  # 3번째 컬럼이 100 초과인 행 출력

# 4. 계산 처리
awk '{sum += $1} END {print sum}' file  # 첫 번째 컬럼의 합계 출력

# 5. 패턴 매칭
awk '/pattern/' file            # pattern이 있는 행 출력
```
[[AWK 프로그래밍 언어]]
## sed 명령어
```bash
# 텍스트 치환
sed 's/old/new/' file          # old를 new로 치환
sed 's/old/new/g' file         # 모든 old를 new로 치환
```

# 시스템 모니터링 명령어
## 프로세스 관리
```bash
ps      # 프로세스 목록 출력
  ps aux               # 모든 프로세스 상세 정보
  ps -ef              # 전체 프로세스 정보
top     # 실시간 프로세스 모니터링
htop    # 향상된 프로세스 모니터링
kill    # 프로세스 종료
  kill PID            # 프로세스 종료
  kill -9 PID         # 강제 종료
```

## 시스템 정보
```bash
df      # 디스크 사용량
  df -h                # 사람이 읽기 쉬운 형태로 출력
du      # 디렉토리 사용량
  du -sh *            # 현재 디렉토리 내 용량 확인
free    # 메모리 사용량
  free -h             # 사람이 읽기 쉬운 형태로 출력
```

# 네트워크 명령어
```bash
ping    # 네트워크 연결 테스트
  ping -c 4 8.8.8.8   # 4번 ping 테스트
netstat # 네트워크 상태 확인
  netstat -tuln       # TCP/UDP 리스닝 포트 확인
ss      # 소켓 상태 확인
  ss -tuln           # TCP/UDP 리스닝 포트 확인
```

# 권한 관리 명령어
```bash
chmod   # 파일 권한 변경
  chmod 755 file      # rwxr-xr-x 권한 설정
  chmod u+x file      # 소유자에게 실행 권한 추가
chown   # 파일 소유자 변경
  chown user:group file  # 소유자와 그룹 변경
```

# 고급 AWK 활용 예제
## 1. 로그 분석
```bash
# Apache 접근 로그에서 IP별 접근 횟수 집계
awk '{ip[$1]++} END {for (i in ip) print i, ip[i]}' access.log

# 시간대별 접근 횟수 분석
awk '{hour=substr($4,14,2); count[hour]++} END {for (h in count) print h, count[h]}' access.log
```

## 2. 시스템 모니터링
```bash
# 메모리 사용량이 높은 프로세스 추출
ps aux | awk '$4>1.0 {print $4,$11}'

# 디스크 사용량 분석
df -h | awk 'NR>1 {print $5,$6}' | sort -nr
```

## 3. 데이터 처리
```bash
# CSV 파일 특정 컬럼 합계
awk -F, '{sum+=$3} END {print sum}' data.csv

# 조건부 데이터 필터링
awk -F, '$3 > 1000 && $4 == "Active" {print $1,$2,$3}' data.csv
```

# 실전 활용 스크립트
## 시스템 모니터링 스크립트
```bash
#!/bin/bash
# 시스템 리소스 모니터링
echo "=== System Resources ==="
echo "Memory Usage:"
free -h | awk 'NR==2 {print "Used: "$3" / Total: "$2}'

echo "Disk Usage:"
df -h | awk '$NF=="/" {print "Used: "$3" / Total: "$2}'

echo "CPU Load:"
uptime | awk '{print $8,$9,$10}'
```

# 주의사항
1. 권한 관련
- rm, chmod 등 파일 조작 명령어 사용 시 주의
- sudo 사용 시 명령어 확인 필수

2. 시스템 리소스
- 대용량 파일 처리 시 메모리 사용량 고려
- 복잡한 awk 처리 시 성능 영향 고려

# 결론
Linux 명령어는 시스템 관리와 데이터 처리에 필수적인 도구이다. 특히 awk와 같은 텍스트 처리 도구는 로그 분석과 데이터 처리에 강력한 기능을 제공한다. 각 명령어의 특성과 용도를 이해하고 적절히 활용하는 것이 중요하다.