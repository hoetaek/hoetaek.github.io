---
date: 2025-05-10
publish: true
tags:
---
SSL 인증서 작업을 하다 보면 `.crt`와 `.pem`이라는 확장자를 가진 파일들을 자주 접하게 됩니다. 이 둘의 차이점과 변환에 대해 알아보겠습니다.

- **`.crt` (Certificate File)**
    
    - `.crt`는 주로 인증서(Certificate) 자체를 담고 있는 파일을 나타내는 확장자입니다.
        
    - 이 파일은 바이너리(DER 인코딩) 형식이거나, Base64로 인코딩된 텍스트(PEM 인코딩) 형식일 수 있습니다.
        
    - 주로 PEM 인코딩된 파일에 사용되지만, 간혹 DER 인코딩된 바이너리 인증서 파일에도 사용될 수 있습니다.
        
    - 내용물은 공개키, 인증서 소유자 정보, 발급자 정보, 유효 기간 등 인증서의 핵심 정보를 포함합니다.
        
- **`.pem` (Privacy Enhanced Mail)**
    
    - `.pem`은 원래 이메일 보안을 위해 개발된 형식의 파일 확장자이지만, 현재는 다양한 암호화 키, 인증서, 인증 요청 등을 저장하는 데 널리 사용됩니다.
        
    - PEM 파일은 항상 Base64로 인코딩된 ASCII 텍스트 형식이며, `-----BEGIN CERTIFICATE-----`와 `-----END CERTIFICATE-----` 같은 헤더와 푸터 라인으로 각 데이터 블록을 구분합니다.
        
    - 하나의 `.pem` 파일 안에는 단일 인증서뿐만 아니라, 개인 키(Private Key), 공개 키(Public Key), 전체 인증서 체인(도메인 인증서 + 중간 인증서(들) + 루트 인증서) 등 여러 개의 암호화 객체를 함께 포함할 수 있습니다. 이것이 `.pem` 파일의 중요한 특징 중 하나입니다.
        
- **주요 차이점 및 혼용 가능성:**
    
    - **인코딩:** `.crt`는 바이너리(DER) 또는 텍스트(PEM)일 수 있지만, `.pem`은 항상 텍스트(PEM)입니다.
        
    - **내용물:** `.crt`는 주로 단일 인증서를 의미하지만, `.pem`은 인증서, 개인 키, 전체 체인 등 다양한 내용을 담을 수 있습니다.
        
    - **확장자 관행:** 실제로는 확장자만으로 파일 형식을 단정하기 어려울 때가 많습니다. 예를 들어, `.crt` 확장자를 가진 파일이 실제로는 PEM 인코딩된 텍스트 파일일 수 있고, 반대로 `.pem` 파일이 단일 인증서만 포함할 수도 있습니다.
        
    - **Nginx에서의 사용:** Nginx와 같은 많은 웹 서버는 PEM 인코딩된 인증서 파일을 선호합니다. 따라서 `.crt` 파일이 바이너리 DER 형식이라면 PEM 형식으로 변환해야 할 수 있습니다. 만약 `.crt` 파일이 이미 PEM 인코딩된 텍스트 파일이라면, 확장자를 `.pem`으로 변경하거나 그대로 사용해도 무방한 경우가 많습니다. **중요한 것은 파일의 확장자가 아니라 실제 내용과 인코딩 형식입니다.**
        
- **변환 (openssl 사용):**
    
    - **DER(.crt, .cer) to PEM:**
        
        ```
        openssl x509 -inform DER -in certificate.crt -out certificate.pem
        ```
        
    - **PEM to DER:**
        
        ```
        openssl x509 -inform PEM -in certificate.pem -out certificate.der
        ```
        
    - **PKCS#12(.pfx, .p12) to PEM (인증서 + 개인키):**
        
        ```
        openssl pkcs12 -in certificate.pfx -out certificate_bundle.pem -nodes
        ```
        
        (`-nodes` 옵션은 개인키 암호화를 제거합니다. 필요에 따라 암호화된 상태로 둘 수도 있습니다.)
        
- 개인 키 파일(.key)의 PEM 변환:
    
    개인 키 파일 역시 PEM 형식으로 저장되는 경우가 많으며, -----BEGIN PRIVATE KEY-----와 같은 헤더를 가집니다.
    
    ```
    # RSA 개인키를 PEM 형식으로 변환 (이미 PEM 형식이지만, 내용을 확인하거나 재포맷)
    openssl rsa -in private.key -text -out private.key.pem
    ```
    
    원본 글에서는 `bundle.key`를 `bundle.key.pem`으로 변환하는 예시가 있었는데, 이는 `.key` 파일이 이미 PEM 형식이지만 명시적으로 `.pem` 확장자를 사용하거나 내용을 확인하기 위한 과정일 수 있습니다. Nginx는 PEM 형식의 개인 키 파일을 사용합니다.
    

결론적으로, Nginx에 SSL 인증서를 설정할 때는 `ssl_certificate` 지시어에 사용될 파일이 PEM 인코딩된 인증서(또는 인증서 체인)여야 하고, `ssl_certificate_key` 지시어에 사용될 파일이 PEM 인코딩된 개인 키여야 합니다. 확장자가 `.crt`라도 내용이 PEM 형식이면 그대로 사용 가능할 수 있지만, 혼동을 피하기 위해 `.pem`으로 통일하거나, 파일 내용을 직접 확인하는 것이 좋습니다.
