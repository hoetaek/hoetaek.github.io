---
date: 2024-11-25
publish: false
tags:
---
# 기본 개념

SSL/TLS에서 인증서(Certificate)와 인증서 키(Certificate Key)는 공개키 암호화 방식의 핵심 구성요소다.

## 구성 요소의 역할
1. 인증서(Certificate)
   - 서버의 신원을 증명하는 공개 문서다
   - 공개키를 포함한다
   - 누구나 열람할 수 있다

2. 인증서 키(Certificate Key)
   - 서버의 비밀키다
   - 절대 공개되어서는 안 된다
   - 암호화된 데이터를 복호화하는 데 사용한다

실생활 비유:
```text
인증서 (Certificate) = 주민등록증
- 누구나 볼 수 있음
- 신원을 증명함
- 위조가 어려움

인증서 키 (Certificate Key) = 도장/서명
- 본인만 가지고 있음
- 신원을 확인하는 수단
- 절대 타인에게 맡기면 안 됨
```

# 동작 방식

```mermaid
sequenceDiagram
    participant Client
    participant Server
    Client->>Server: 연결 요청
    Server->>Client: 인증서 전송 (공개키 포함)
    Client->>Client: 인증서 검증
    Client->>Server: 공개키로 세션키 암호화하여 전송
    Server->>Server: 비밀키로 세션키 복호화
    Note over Client,Server: 이후 세션키로 암호화된 통신
```

# 실제 파일 구조

## 인증서 파일 (.crt, .pem)
```text
-----BEGIN CERTIFICATE-----
MIIDfTCCAmWgAwIBAgIUJ40Zk+ft8qgugXJ/0...
[인코딩된 인증서 데이터]
-----END CERTIFICATE-----
```

## 비밀키 파일 (.key)
```text
-----BEGIN PRIVATE KEY-----
MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgw...
[인코딩된 비밀키 데이터]
-----END PRIVATE KEY-----
```

# 키 쌍 생성과 관리

## 키 쌍 생성
```bash
# 비밀키 생성
openssl genrsa -out private.key 2048

# 공개키 추출
openssl rsa -in private.key -pubout -out public.key

# CSR(Certificate Signing Request) 생성
openssl req -new -key private.key -out request.csr

# 자체 서명 인증서 생성
openssl x509 -req -days 365 -in request.csr \
    -signkey private.key -out certificate.crt
```

## 권한 관리
```bash
# 비밀키 권한 설정 (중요!)
chmod 600 private.key
chown root:root private.key

# 인증서 권한 설정
chmod 644 certificate.crt
```

# 실제 서버 설정 예시

## Nginx 설정
```nginx
server {
    listen 443 ssl;
    server_name example.com;
    
    # 인증서 설정
    ssl_certificate /etc/nginx/ssl/certificate.crt;
    ssl_certificate_key /etc/nginx/ssl/private.key;
    
    # 보안 설정
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_prefer_server_ciphers off;
}
```

# 보안 고려사항

## 비밀키 보호
1. 파일 시스템 보안
```bash
# 비밀키 디렉터리 보호
chmod 700 /etc/ssl/private
```

2. 백업 보안
```bash
# 암호화된 백업 생성
openssl rsa -aes256 -in private.key -out encrypted.key
```

3. 전송 보안
```bash
# SCP를 통한 안전한 전송
scp -P 22 private.key user@server:/path/to/destination
```

## 모니터링
```bash
# 파일 변경 모니터링
inotifywait -m /etc/ssl/private -e modify,delete,create
```

# 문제 해결 가이드

## 일반적인 문제들

1. 키 쌍 불일치
```bash
# 인증서와 키 쌍 검증
openssl x509 -noout -modulus -in certificate.crt | openssl md5
openssl rsa -noout -modulus -in private.key | openssl md5
# 결과가 동일해야 함
```

2. 권한 문제
```bash
# 권한 문제 해결
sudo chown www-data:www-data /etc/ssl/private/private.key
sudo chmod 600 /etc/ssl/private/private.key
```

## 인증서 체인 문제
```bash
# 체인 검증
openssl verify -verbose -CAfile chain.pem certificate.crt
```

# 성능 최적화

## 세션 캐시 설정
```nginx
ssl_session_cache shared:SSL:10m;
ssl_session_timeout 10m;
```

## OCSP Stapling
```nginx
ssl_stapling on;
ssl_stapling_verify on;
resolver 8.8.8.8 8.8.4.4;
```

# 유지보수 가이드

## 인증서 갱신
```bash
# 만료일 확인
openssl x509 -noout -enddate -in certificate.crt

# CSR 재사용하여 갱신
openssl x509 -req -days 365 -in request.csr \
    -signkey private.key -out new_certificate.crt
```

## 키 교체 절차
1. 새 키 쌍 생성
2. 새 CSR 생성
3. 새 인증서 발급
4. 서비스 무중단 교체

# 결론

Certificate와 Certificate Key는 SSL/TLS 보안의 기반이 되는 요소다:
- Certificate는 공개되어도 안전한 공개키를 포함한다
- Certificate Key는 절대 노출되어서는 안 되는 비밀키다
- 두 요소는 항상 쌍으로 관리되어야 한다

적절한 관리와 보안 조치를 통해 안전한 HTTPS 통신을 구현할 수 있다.