---
date: 2024-12-04
publish: true
tags:
---
# Here Document란?
Here Document(이하 heredoc)는 여러 줄의 문자열을 쉽고 깔끔하게 처리할 수 있는 쉘 프로그래밍의 특별한 기능이다. 마치 레스토랑에서 주방에 주문서를 전달하는 것처럼, 프로그램에 여러 줄의 텍스트를 한 번에 전달할 수 있다.

# 기본 동작 방식
heredoc는 지정된 구분자(보통 EOF나 EOT 등)를 사용하여 시작과 끝을 표시한다. 구분자 사이의 모든 텍스트는 그대로 유지되며, 변수 확장과 명령 치환도 가능하다.

기본 문법:
```bash
command << DELIMITER
text content
more text content
DELIMITER
```

# 실제 사용 예시

## 1. 간단한 파일 생성
```bash
# 일반적인 방식
echo "Hello" > greeting.txt
echo "World" >> greeting.txt

# heredoc 활용
cat << EOF > greeting.txt
Hello
World
EOF
```

## 2. 설정 파일 생성
```bash
# MySQL 설정 파일 생성 예시
sudo cat << EOF > /etc/mysql/conf.d/custom.cnf
[mysqld]
max_connections = 100
query_cache_size = 32M
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
EOF
```

# 고급 활용법

## 1. 변수 확장 제어
```bash
# 변수 확장 허용 (기본)
name="John"
cat << EOF
Hello, $name!
EOF

# 변수 확장 방지
cat << 'EOF'
Hello, $name!
EOF
```

## 2. 들여쓰기 처리
```bash
# 들여쓰기 유지
cat << 'EOF' | sed 's/^/    /'
function example() {
    console.log("Hello");
}
EOF
```

# 주의사항
1. 구분자(EOF 등)는 줄의 맨 앞에 있어야 한다
2. 구분자 주변에 추가 공백이 없어야 한다
3. 의도하지 않은 변수 확장을 주의해야 한다
4. 큰따옴표로 묶인 heredoc은 변수 확장이 발생한다

# 실제 사용 시나리오

## Docker 컨테이너 설정
```bash
# Dockerfile 생성
cat << 'EOF' > Dockerfile
FROM ubuntu:20.04
RUN apt-get update && apt-get install -y \
    nginx \
    curl
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
EOF
```

## 시스템 서비스 설정
```bash
# 시스템 서비스 파일 생성
sudo cat << EOF > /etc/systemd/system/myapp.service
[Unit]
Description=My Application Service
After=network.target

[Service]
Type=simple
User=myapp
ExecStart=/usr/local/bin/myapp
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```

# 성능과 보안 고려사항
1. 대용량 데이터 처리 시 메모리 사용량 고려
2. 변수 확장 시 보안 취약점 주의
3. 민감한 정보 처리 시 권한 설정 확인
4. 임시 파일 생성 최소화

# 결론
heredoc은 복잡한 문자열이나 설정 파일을 다룰 때 매우 유용한 도구다. 특히 여러 줄의 텍스트를 처리하거나, 동적으로 설정 파일을 생성할 때 코드의 가독성과 유지보수성을 크게 향상시킬 수 있다.