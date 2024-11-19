---
date: 2024-11-19
publish: false
tags:
---
# 개념 이해

## 기본 목적의 차이
ECDHE와 Ed25519는 서로 다른 암호화 목적을 가진다:
- ECDHE: 안전한 키 교환 (비밀 통신 채널 수립)
- Ed25519: 디지털 서명 (신원 확인 및 무결성 검증)

```mermaid
sequenceDiagram
    participant Client
    participant Server
    
    rect rgb(200, 255, 200)
        Note over Client,Server: 1. Ed25519로 신원 확인
        Client->>Server: Ed25519로 서명된 메시지
        Note over Server: 클라이언트 신원 확인
        Server->>Client: Ed25519로 서명된 응답
        Note over Client: 서버 신원 확인
    end
    
    rect rgb(200, 200, 255)
        Note over Client,Server: 2. ECDHE로 암호화 키 교환
        Client->>Server: ECDHE 공개키
        Server->>Client: ECDHE 공개키
        Note over Client,Server: 양쪽 모두 동일한 세션키 생성
    end
    
    rect rgb(255, 200, 200)
        Note over Client,Server: 3. 세션키로 암호화된 통신
        Client->>Server: 암호화된 실제 데이터
        Server->>Client: 암호화된 실제 데이터
    end
```

즉:
1. Ed25519로 "너 진짜 구글이야?" 확인
2. ECDHE로 "우리끼리만 아는 비밀 암호" 생성
3. 그 비밀 암호로 실제 데이터 주고받기

## 실생활 비유
두 알고리즘의 차이는 다음과 같은 상황과 유사하다:
- [[Ed25519 - 디지털 서명 알고리즘|Ed25519]]: 편지에 서명하여 신원 증명
- [[ECDHE - 타원곡선을 이용한 키 교환|ECDHE]]: 비밀 대화를 위한 암호 방식 합의

# 핵심 차이점

## 특성 비교
```mermaid
graph TB
    subgraph "ECDHE 특성"
        A[임시성] --> B[매 세션 새로운 키]
        C[목적] --> D[비밀 통신]
        E[결과] --> F[공유 비밀키]
    end

    subgraph "Ed25519 특성"
        G[영구성] --> H[동일 키 재사용]
        I[목적] --> J[신원 증명]
        K[결과] --> L[디지털 서명]
    end
```

## 키 수명
```mermaid
timeline
    title 키 수명 비교
    ECDHE : 세션 시작
            키 교환
            세션 종료
            키 폐기
    Ed25519 : 키 생성
              반복 사용
              장기 보관
              필요시 갱신
```

# 실제 사용 사례

## ECDHE 사용 예시
```python
def establish_secure_channel():
    """ECDHE를 사용한 보안 채널 수립"""
    # 매 연결마다 새로운 키 생성
    ephemeral_private = generate_private_key()
    ephemeral_public = derive_public_key(ephemeral_private)
    
    # 키 교환
    shared_secret = compute_shared_secret(
        ephemeral_private,
        received_public_key
    )
    
    # 세션 종료시 키 폐기
    del ephemeral_private
```

## Ed25519 사용 예시
```python
class DocumentSigner:
    """Ed25519를 사용한 문서 서명"""
    def __init__(self):
        # 한 번 생성하여 계속 사용
        self.private_key = load_permanent_key()
        
    def sign_document(self, document):
        # 동일한 키로 반복 서명
        signature = self.private_key.sign(document)
        return signature
```

# 실제 통신 과정

## 일반적인 보안 통신 흐름
```mermaid
sequenceDiagram
    participant Client
    participant Server
    
    Note over Client,Server: 1. Ed25519로 신원 확인
    Client->>Server: Ed25519 서명된 인증 요청
    Server->>Client: Ed25519 서명된 인증 응답
    
    Note over Client,Server: 2. ECDHE로 보안 채널 수립
    Client->>Server: ECDHE 공개키 교환
    Server->>Client: ECDHE 공개키 교환
    
    Note over Client,Server: 3. 암호화된 통신 시작
    Client->>Server: 암호화된 데이터
    Server->>Client: 암호화된 데이터
```

# 보안 특성

## 각 알고리즘의 보안 특징
```mermaid
graph TB
    subgraph "ECDHE 보안"
        A[Perfect Forward Secrecy] --> B[과거 통신 보호]
        C[임시키] --> D[세션 독립성]
    end
    
    subgraph "Ed25519 보안"
        E[부인 방지] --> F[서명 검증]
        G[무결성] --> H[변조 감지]
    end
```

# 함께 사용하는 이유

## 상호 보완적 특성
```mermaid
graph LR
    A[보안 요구사항] --> B[인증]
    A --> C[기밀성]
    
    B --> D[Ed25519]
    C --> E[ECDHE]
    
    D --> F[신원 확인]
    E --> G[안전한 통신]
```

# 구현 시 고려사항

## ECDHE 구현
```python
class ECDHESession:
    def __init__(self):
        self.private_key = None
        
    def start_session(self):
        # 새로운 세션키 생성
        self.private_key = generate_ephemeral_key()
        
    def end_session(self):
        # 세션 종료시 키 삭제
        self.private_key = None
```

## Ed25519 구현
```python
class Ed25519Identity:
    def __init__(self):
        # 영구적인 키 로드
        self.private_key = load_identity_key()
        
    def sign(self, data):
        # 동일 키로 서명
        return self.private_key.sign(data)
```

# 성능 특성

## 연산 부하
```mermaid
graph TB
    subgraph "연산 비용"
        A[ECDHE] --> B[키 교환 시 높음]
        A --> C[세션 중 낮음]
        
        D[Ed25519] --> E[서명 시 중간]
        D --> F[검증 시 낮음]
    end
```

# 결론

## 각 알고리즘의 역할
1. ECDHE:
   - 안전한 통신 채널 수립
   - 세션키 교환
   - 임시 키 사용

2. Ed25519:
   - 신원 증명
   - 문서 서명
   - 영구 키 사용

## 최적의 사용
1. 인증은 Ed25519
2. 키 교환은 ECDHE
3. 두 알고리즘의 장점 활용