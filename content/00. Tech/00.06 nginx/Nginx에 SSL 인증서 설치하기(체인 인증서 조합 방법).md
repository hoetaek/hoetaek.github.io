---
date: 2025-05-10
publish: true
tags:
---
### 핵심 요약

- SSL/TLS 인증서는 검증 수준에 따라 DV, OV, EV 등으로 나뉘며, 신뢰 체인을 위해 루트, 중간(체인) 인증서가 함께 사용됩니다.
    
- 최근에는 체인 인증서가 분리되어 제공되는 경우가 많아, 서버 관리자가 직접 인증서 파일들을 올바른 순서로 통합해야 합니다.
    
- 체인 인증서(중간 인증서)는 도메인 인증서와 Root CA를 연결하여 신뢰 체인을 완성하며, 보안 및 관리 유연성 때문에 별도로 제공됩니다.
    
- 체인 인증서가 포함된 경우, 여러 인증서를 **올바른 순서(도메인 -> 중간(들) -> 루트(선택적))**로 합쳐 하나의 파일로 만들어야 합니다.
    
- Nginx 설정 파일에서 `listen 443 ssl;`, `ssl_certificate`, `ssl_certificate_key` 등의 지시어를 사용하여 SSL을 활성화하고 인증서 경로를 지정합니다.
    
- HTTP 요청을 HTTPS로 리디렉션하는 설정을 추가하는 것이 좋습니다.
    
- SSL 테스트 도구를 사용하여 체인 완성도를 포함한 전체 SSL 설정을 반드시 확인해야 합니다.
    

### 1. SSL/TLS 인증서의 기본 종류
[[SSL TLS 인증서의 기본 종류]]

### 2. SSL 인증서 적용 방식의 변화: 체인 인증서의 등장

과거 SSL 인증서를 적용할 때는 인증 기관(CA)에서 도메인 인증서와 필요한 중간(체인) 인증서가 이미 하나로 합쳐진 단일 파일 형태로 제공되거나, 서버가 체인을 자동으로 구성해주는 경우가 많았습니다. 이러한 **'기존 방식'에서는 사용자가 체인 인증서의 존재를 크게 의식하거나 파일을 직접 조합할 필요가 없었습니다.

하지만 최근에는 보안 정책 강화 및 인증서 관리의 유연성 증대로 인해, 도메인 인증서, 하나 이상의 체인(중간) 인증서, 그리고 루트 인증서가 각각 별도의 파일로 제공되는 경우가 늘고 있습니다. 이렇게 분리된 인증서들을 서버에 올바르게 적용하기 위해서는 각 인증서의 역할과 올바른 조합 순서를 이해하고 수동으로 설정하는 과정이 필요하게 되었습니다. 이 문서는 바로 이러한 **체인 인증서가 포함된 SSL 인증서를 Nginx에 적용하는 과정**을 상세히 다루고자 합니다.

### 3. Chain 인증서란? 왜 필요하고 별도로 제공될까?

[[Chain 인증서]]

### 4. Nginx SSL 설정 단계

##### 가. 인증서 파일 통합 (Chain 인증서가 있을 경우)

- 전달받은 여러 인증서 파일(Domain 인증서, Chain 인증서(들), 경우에 따라 Root 인증서)을 **올바른 순서로** 하나의 `.crt` 또는 `.pem` 파일로 합칩니다. 일반적인 순서는 `도메인 인증서 -> 중간 인증서(들) -> Root 인증서` 순입니다. (Root 인증서는 클라이언트가 이미 가지고 있는 경우가 많아 필수는 아니지만, 포함하는 것이 안전할 수 있습니다.)
    
- **Linux/Unix 계열:** 쉘에서 `cat` 명령어를 사용하여 각 파일의 내용을 순서대로 합쳐 하나의 파일로 만듭니다.

```
cat Domain인증서.crt Chain인증서1.crt Chain인증서2.crt Root인증서.crt > bundle.crt
```
(Chain 인증서가 여러 개일 경우 순서대로 추가합니다.)

- 텍스트 편집기를 사용하여 수동으로 각 PEM 형식 인증서 내용을 복사하여 하나의 파일로 만들 수도 있습니다. 이때 각 인증서의 `-----BEGIN CERTIFICATE-----`와 `-----END CERTIFICATE-----` 블록이 정확히 복사되고, 블록 사이에 불필요한 문자나 공백이 없는지 확인해야 합니다.
        
- 합친 후, 인증서 내용이 올바르게 구분(개행)되어 있는지 확인합니다. (예: `-----END CERTIFICATE----------BEGIN CERTIFICATE-----`가 아닌 `-----END CERTIFICATE-----\n-----BEGIN CERTIFICATE-----` 형태)

##### 나. CRT와 PEM 파일 이해 및 변환 (선택적)
[[CRT와 PEM]]

##### 다. Nginx 설정 파일 수정

- SSL 설정은 각 도메인의 `server` 블록에서 변경합니다. (`nginx.conf` 또는 분리된 설정 파일)
    
- **기본 HTTP 설정 예시:**
    
    ```
    server {
        listen 80;
        listen [::]:80;
        server_name sample.co.kr;
    
        location / {
    
        }
    }
    ```
    
- **SSL 적용 설정 예시:**
    
    ```
    server {
        listen 443 ssl; # 443 포트에 ssl 명시
        listen [::]:443 ssl; # IPv6에도 동일하게 적용
        server_name sample.co.kr;
    
        ssl_certificate /var/cert/bundle.pem; # 통합되고 PEM 인코딩된 인증서 파일 경로
        ssl_certificate_key /var/cert/bundle.key.pem; # PEM 인코딩된 개인 Key 파일 경로
    
        ssl_protocols TLSv1.2 TLSv1.3; # 보안상 권장되는 최신 프로토콜 사용 (TLSv1, TLSv1.1은 취약점 발견으로 사용 중단 권고)
        ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384'; # 강력한 암호화 스위트 권장
        ssl_prefer_server_ciphers off; # 최신 브라우저는 클라이언트가 선호하는 암호화 방식을 사용하도록 off 권장 (on도 무방)
    
        location / {
            # 프록시 설정 등
        }
    }
    
    # HTTP 접속 시 HTTPS로 리디렉션
    server {
        listen 80;
        listen [::]:80;
        server_name sample.co.kr;
    
        # $host 변수를 사용하여 정확한 호스트 이름으로 리디렉션
        return 301 https://$host$request_uri;
    }
    ```
    
    - `ssl on;` 지시어는 최신 Nginx 버전에서는 `listen 443 ssl;`로 대체되었습니다.
        
    - `ssl_certificate`: 인증서 파일 경로를 지정합니다. **이 파일에는 도메인 인증서와 모든 중간 인증서가 올바른 순서로 포함되어야 하며, PEM 형식이어야 합니다.**
        
    - `ssl_certificate_key`: 인증서 개인 키 파일 경로를 지정합니다. **PEM 형식이어야 합니다.**
        
    - `ssl_protocols`, `ssl_ciphers`: 보안 강화를 위한 프로토콜 및 암호화 스위트를 설정합니다. 최신 보안 권고 사항을 따르는 것이 좋습니다.
        

### 5. 적용 테스트

- SSL 테스트 사이트 (예: Qualys SSL Labs의 SSL Server Test)를 통해 도메인 주소를 입력하여 정상 적용 여부, 체인 완성도, 프로토콜 및 암호화 스위트 강도 등을 종합적으로 확인합니다.
    
- 테스트 시 "Chain issues: Incomplete" 또는 "Extra certificates in chain" 등 체인 관련 메시지가 표시될 경우, `ssl_certificate` 파일에 포함된 인증서들의 순서나 누락/중복 여부를 확인해야 합니다. 일반적으로 `.pem` 대신 통합된 `.crt` 파일을 사용하는 것 자체보다는, 그 파일 내의 인증서 순서와 완전성이 중요합니다.
    
