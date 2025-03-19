---
date: 2025-03-18
publish: true
tags:
---
```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 0 # Include headings from the specified level
maxLevel: 0 # Include headings up to the specified level
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```
평가: 미래에 도움이 될 점수 - 9/10 (다양한 언어로 실제 구현 예시를 포함하고 디자인 패턴의 핵심 개념을 설명하므로 높은 점수 부여)

# 전략 패턴 - 런타임 알고리즘 교체

## 키워드

- 행동 디자인 패턴
- 런타임 알고리즘 교체
- 인터페이스 추상화

## 내용

### 전략 패턴 개요

전략 패턴(Strategy Pattern)은 알고리즘을 캡슐화하고 이를 교체 가능하게 만드는 행동 디자인 패턴이다. 이 패턴은 클라이언트 코드를 변경하지 않고도 런타임에 알고리즘을 교체할 수 있게 해준다. 전략 패턴은 소프트웨어의 확장성과 유지보수성을 크게 향상시킨다. 이 패턴은 객체지향 프로그래밍의 핵심 원칙인 '개방-폐쇄 원칙(OCP)'과 '단일 책임 원칙(SRP)'을 적용한 대표적인 예시이다. 미래의 내가 이 패턴을 기억해야 하는 이유는 다양한 알고리즘이 필요한 상황에서 코드 중복을 줄이고 유연성을 높이는 데 매우 효과적이기 때문이다.

### 전략 패턴의 구조

전략 패턴은 크게 세 가지 주요 구성 요소로 이루어진다. 첫째, '전략(Strategy)' 인터페이스는 지원되는 모든 알고리즘에 대한 공통 메서드를 정의한다. 둘째, '구체적 전략(Concrete Strategy)' 클래스들은 전략 인터페이스를 구현하며 각각 다른 알고리즘을 제공한다. 셋째, '컨텍스트(Context)' 클래스는 전략 객체를 참조하고 이를 통해 알고리즘을 실행한다. 컨텍스트는 전략 객체를 교체할 수 있는 메서드를 제공하여 런타임에 알고리즘을 변경할 수 있게 한다. 이러한 구조를 통해 새로운 알고리즘을 추가할 때 기존 코드를 변경하지 않고도 확장이 가능하다.

### 파이썬으로 구현한 전략 패턴
```python
from abc import ABC, abstractmethod
from typing import List

# 전략 인터페이스
class SortStrategy(ABC):
    @abstractmethod
    def sort(self, data: List) -> List:
        pass

# 구체적 전략 1: 버블 정렬
class BubbleSortStrategy(SortStrategy):
    def sort(self, data: List) -> List:
        print("버블 정렬 수행 중...")
        result = data.copy()
        n = len(result)
        for i in range(n):
            for j in range(0, n - i - 1):
                if result[j] > result[j + 1]:
                    result[j], result[j + 1] = result[j + 1], result[j]
        return result

# 구체적 전략 2: 퀵 정렬
class QuickSortStrategy(SortStrategy):
    def sort(self, data: List) -> List:
        print("퀵 정렬 수행 중...")
        result = data.copy()
        if len(result) <= 1:
            return result
        
        pivot = result[len(result) // 2]
        left = [x for x in result if x < pivot]
        middle = [x for x in result if x == pivot]
        right = [x for x in result if x > pivot]
        
        return self.sort(left) + middle + self.sort(right)

# 구체적 전략 3: 파이썬 내장 정렬
class PythonSortStrategy(SortStrategy):
    def sort(self, data: List) -> List:
        print("파이썬 내장 정렬 수행 중...")
        return sorted(data)

# 컨텍스트
class Sorter:
    def __init__(self, strategy: SortStrategy = None):
        self._strategy = strategy or PythonSortStrategy()
    
    def set_strategy(self, strategy: SortStrategy):
        self._strategy = strategy
    
    def sort(self, data: List) -> List:
        return self._strategy.sort(data)

# 클라이언트 코드
if __name__ == "__main__":
    # 데이터 준비
    data = [7, 1, 4, 6, 3, 8, 2, 5]
    
    # 컨텍스트 생성
    sorter = Sorter()
    
    # 기본 전략 (파이썬 내장 정렬) 사용
    print(f"기본 정렬 결과: {sorter.sort(data)}")
    
    # 버블 정렬 전략으로 변경
    sorter.set_strategy(BubbleSortStrategy())
    print(f"버블 정렬 결과: {sorter.sort(data)}")
    
    # 퀵 정렬 전략으로 변경
    sorter.set_strategy(QuickSortStrategy())
    print(f"퀵 정렬 결과: {sorter.sort(data)}")
```
파이썬 예시에서는 정렬 알고리즘을 전략 패턴으로 구현했다. `SortStrategy` 추상 클래스는 모든 정렬 전략이 구현해야 하는 인터페이스를 정의한다. 세 가지 구체적 전략인 버블 정렬, 퀵 정렬, 파이썬 내장 정렬은 각각 다른 알고리즘을 제공한다. `Sorter` 클래스는 컨텍스트 역할을 하며 런타임에 정렬 전략을 변경할 수 있다. 파이썬에서는 `abc` 모듈을 사용해 추상 클래스를 만들고 타입 힌팅을 통해 코드의 가독성을 높였다. 이 구현은 새로운 정렬 알고리즘을 추가할 때 기존 코드를 수정하지 않고도 확장할 수 있는 장점이 있다.

### 자바스크립트로 구현한 전략 패턴

```js
// 전략 인터페이스 (자바스크립트에서는 명시적 인터페이스가 없으므로 암묵적으로 구현)

// 구체적 전략 1: 신용카드 결제
class CreditCardPaymentStrategy {
  constructor(cardNumber, name, cvv, expiryDate) {
    this.cardNumber = cardNumber;
    this.name = name;
    this.cvv = cvv;
    this.expiryDate = expiryDate;
  }

  pay(amount) {
    console.log(`${amount}원을 신용카드로 결제합니다: ${this.cardNumber.slice(-4)}`);
    return true;
  }
}

// 구체적 전략 2: 페이팔 결제
class PayPalPaymentStrategy {
  constructor(email, password) {
    this.email = email;
    this.password = password;
  }

  pay(amount) {
    console.log(`${amount}원을 PayPal로 결제합니다: ${this.email}`);
    return true;
  }
}

// 구체적 전략 3: 암호화폐 결제
class CryptoPaymentStrategy {
  constructor(walletAddress) {
    this.walletAddress = walletAddress;
  }

  pay(amount) {
    console.log(`${amount}원을 암호화폐로 결제합니다: ${this.walletAddress.slice(0, 10)}...`);
    return true;
  }
}

// 컨텍스트
class PaymentProcessor {
  constructor(paymentStrategy = null) {
    this.paymentStrategy = paymentStrategy;
  }

  setPaymentStrategy(paymentStrategy) {
    this.paymentStrategy = paymentStrategy;
  }

  processPayment(amount) {
    if (!this.paymentStrategy) {
      throw new Error('결제 전략이 설정되지 않았습니다.');
    }
    return this.paymentStrategy.pay(amount);
  }
}

// 클라이언트 코드
function runExample() {
  // 상품 가격
  const price = 15000;
  
  // 결제 프로세서 생성
  const paymentProcessor = new PaymentProcessor();
  
  // 신용카드 결제 전략 사용
  const creditCardStrategy = new CreditCardPaymentStrategy(
    '1234-5678-9012-3456',
    '홍길동',
    '123',
    '12/25'
  );
  paymentProcessor.setPaymentStrategy(creditCardStrategy);
  paymentProcessor.processPayment(price);
  
  // 페이팔 결제 전략으로 변경
  const paypalStrategy = new PayPalPaymentStrategy(
    'user@example.com',
    'password123'
  );
  paymentProcessor.setPaymentStrategy(paypalStrategy);
  paymentProcessor.processPayment(price);
  
  // 암호화폐 결제 전략으로 변경
  const cryptoStrategy = new CryptoPaymentStrategy(
    '0x71C7656EC7ab88b098defB751B7401B5f6d8976F'
  );
  paymentProcessor.setPaymentStrategy(cryptoStrategy);
  paymentProcessor.processPayment(price);
}

// 예제 실행
runExample();
```
자바스크립트 예시에서는 온라인 결제 시스템에 전략 패턴을 적용했다. 이 구현에서는 신용카드, 페이팔, 암호화폐와 같은 다양한 결제 방식을 각각의 전략으로 캡슐화했다. 자바스크립트는 명시적인 인터페이스를 지원하지 않기 때문에 암묵적인 인터페이스를 통해 각 전략 클래스가 `pay` 메서드를 구현한다. `PaymentProcessor` 클래스는 컨텍스트 역할을 하며 다양한 결제 전략을 교체할 수 있게 한다. 이러한 구현은 새로운 결제 방식이 추가될 때마다 기존 코드를 변경하지 않고 확장할 수 있어 유지보수가 용이하다.

### PHP로 구현한 전략 패턴

```php
<?php

// 전략 인터페이스
interface LoggerStrategy {
    public function log(string $message): void;
}

// 구체적 전략 1: 파일 로거
class FileLogger implements LoggerStrategy {
    private $filePath;
    
    public function __construct(string $filePath) {
        $this->filePath = $filePath;
    }
    
    public function log(string $message): void {
        $timestamp = date('Y-m-d H:i:s');
        $logMessage = "[$timestamp] $message" . PHP_EOL;
        
        echo "파일에 로그 기록: $logMessage";
        // 실제 구현에서는 아래와 같이 파일에 기록
        // file_put_contents($this->filePath, $logMessage, FILE_APPEND);
    }
}

// 구체적 전략 2: 데이터베이스 로거
class DatabaseLogger implements LoggerStrategy {
    private $connection;
    
    public function __construct(string $connectionString) {
        // 실제 구현에서는 DB 연결 설정
        $this->connection = $connectionString;
    }
    
    public function log(string $message): void {
        $timestamp = date('Y-m-d H:i:s');
        echo "데이터베이스에 로그 기록: [$timestamp] $message" . PHP_EOL;
        
        // 실제 구현에서는 아래와 같이 DB에 기록
        // $query = "INSERT INTO logs (timestamp, message) VALUES (?, ?)";
        // execute query...
    }
}

// 구체적 전략 3: 이메일 로거
class EmailLogger implements LoggerStrategy {
    private $recipientEmail;
    
    public function __construct(string $recipientEmail) {
        $this->recipientEmail = $recipientEmail;
    }
    
    public function log(string $message): void {
        $timestamp = date('Y-m-d H:i:s');
        echo "이메일로 로그 전송: [$timestamp] $message (To: {$this->recipientEmail})" . PHP_EOL;
        
        // 실제 구현에서는 아래와 같이 이메일 전송
        // mail($this->recipientEmail, 'Log Notification', $message);
    }
}

// 컨텍스트
class LogManager {
    private $logger;
    
    public function __construct(LoggerStrategy $logger = null) {
        $this->logger = $logger;
    }
    
    public function setLogger(LoggerStrategy $logger): void {
        $this->logger = $logger;
    }
    
    public function log(string $message): void {
        if ($this->logger === null) {
            throw new Exception('로거가 설정되지 않았습니다.');
        }
        
        $this->logger->log($message);
    }
}

// 클라이언트 코드
function runExample() {
    // 로그 매니저 생성
    $logManager = new LogManager();
    
    // 파일 로거 전략 사용
    $fileLogger = new FileLogger('/var/log/app.log');
    $logManager->setLogger($fileLogger);
    $logManager->log('사용자가 로그인했습니다.');
    
    // 데이터베이스 로거 전략으로 변경
    $dbLogger = new DatabaseLogger('mysql:host=localhost;dbname=logs');
    $logManager->setLogger($dbLogger);
    $logManager->log('중요 데이터가 변경되었습니다.');
    
    // 이메일 로거 전략으로 변경 (심각한 오류 발생 시)
    $emailLogger = new EmailLogger('admin@example.com');
    $logManager->setLogger($emailLogger);
    $logManager->log('심각한 시스템 오류가 발생했습니다!');
}

// 예제 실행
runExample();
```
PHP 예시에서는 로깅 시스템에 전략 패턴을 적용했다. 이 구현에서는 파일, 데이터베이스, 이메일과 같은 다양한 로깅 방식을 각각의 전략으로 캡슐화했다. `LoggerStrategy` 인터페이스는 모든 로거가 구현해야 하는 `log` 메서드를 정의한다. `LogManager` 클래스는 컨텍스트 역할을 하며 다양한 로깅 전략을 교체할 수 있게 한다. PHP에서는 인터페이스를 명시적으로 선언하고 클래스가 이를 구현하는 방식으로 전략 패턴을 구현했다. 이 패턴은 로깅 요구사항이 변경되거나 새로운 로깅 방식이 필요할 때 기존 코드를 수정하지 않고 확장할 수 있게 한다.

### 전략 패턴의 실제 사용 사례

전략 패턴은 실제 소프트웨어 개발에서 광범위하게 사용된다. 대표적인 사례로는 결제 게이트웨이, 데이터 압축 알고리즘, 인증 메커니즘, 라우팅 알고리즘, 텍스트 포맷팅 등이 있다. 프레임워크와 라이브러리에서도 이 패턴을 자주 볼 수 있다. 예를 들어, Spring 프레임워크의 의존성 주입, Java의 Comparator 인터페이스, Node.js의 미들웨어 등이 전략 패턴의 원리를 활용한다. 이 패턴은 특히 알고리즘의 동작이 런타임에 결정되어야 하는 경우나 여러 알고리즘 중 하나를 선택해야 하는 경우에 유용하다.

### 전략 패턴의 장단점

전략 패턴의 주요 장점은 여러 알고리즘을 캡슐화하고 런타임에 교체할 수 있다는 점이다. 이를 통해 코드의 재사용성과 확장성이 향상되고, 조건문(if-else, switch)의 복잡성을 줄일 수 있다. 또한 새로운 전략을 추가할 때 기존 코드를 수정하지 않아도 되므로 개방-폐쇄 원칙을 준수한다. 단점으로는 전략이 많아질수록 클래스의 수가 증가하고, 클라이언트가 전략을 선택하기 위해 전략 간의 차이점을 알아야 한다는 점이다. 또한 간단한 알고리즘에서는 오히려 복잡성이 증가할 수 있으므로 적절한 상황에서 사용하는 것이 중요하다.

### 결론 및 액션 아이템

전략 패턴은 알고리즘을 캡슐화하고 교체 가능하게 만드는 강력한 디자인 패턴이다. 이 패턴은 코드의 유연성, 확장성, 재사용성을 향상시키며 객체지향 설계 원칙을 효과적으로 적용할 수 있게 한다. 전략 패턴을 활용하면 조건문으로 인한 코드 복잡성을 줄이고 새로운 알고리즘을 쉽게 추가할 수 있다. 또한 테스트 용이성도 향상된다.

#### 액션 아이템:

1. 기존 프로젝트에서 조건문이 많고 알고리즘이 자주 변경되는 부분을 식별하여 전략 패턴 적용을 검토한다.
2. 세 가지 언어 예시를 참고하여 자신의 주력 언어로 전략 패턴 샘플 코드를 작성하고 연습한다.
3. 다음 프로젝트에서 결제, 인증, 로깅 등과 같이 다양한 알고리즘이 필요한 기능에 전략 패턴을 적용한다.
4. 전략 패턴이 오버엔지니어링이 되지 않도록 복잡성과 이점 사이의 균형을 고려한다.
5. 팀원들과 디자인 패턴 스터디를 진행하여 전략 패턴과 다른 행동 패턴의 차이점을 학습한다.

