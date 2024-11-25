---
date: 2024-11-25
publish: true
tags:
---
# 개념 소개

Laravel Testing에서 Mock 객체를 생성할 때 'overload:'와 'alias:'는 서로 다른 목적으로 사용되는 중요한 패턴이다.

# Mock 타입별 특징

## overload: 패턴
새로운 클래스 인스턴스 생성을 가로채서 Mock 객체로 대체하는 방식이다.

### 주요 특징
- new 키워드로 생성되는 인스턴스를 가로챔
- 클래스의 인스턴스 메서드를 테스트할 때 사용
- 의존성 주입이 어려운 상황에서 유용

```php
class OrderProcessor {
    public function process()
    {
        $paymentGateway = new PaymentGateway();
        return $paymentGateway->charge();
    }
}

class OrderProcessorTest extends TestCase
{
    public function testProcessOrder()
    {
        // PaymentGateway 인스턴스 생성을 가로챔
        $mock = Mockery::mock('overload:PaymentGateway');
        $mock->shouldReceive('charge')
             ->once()
             ->andReturn(true);

        $processor = new OrderProcessor();
        $result = $processor->process();

        $this->assertTrue($result);
    }
}
```

## alias: 패턴
정적(static) 메서드 호출을 Mock 객체로 대체하는 방식이다.

### 주요 특징
- 클래스의 static 메서드를 대체
- 파사드(Facade) 패턴이나 정적 메서드를 테스트할 때 사용
- 전역적으로 사용되는 유틸리티 메서드 테스트에 유용

```php
class NotificationService {
    public function send()
    {
        return Logger::log('Notification sent');
    }
}

class NotificationServiceTest extends TestCase
{
    public function testSendNotification()
    {
        // Logger의 정적 메서드를 Mock으로 대체
        $mock = Mockery::mock('alias:Logger');
        $mock->shouldReceive('log')
             ->with('Notification sent')
             ->once()
             ->andReturn(true);

        $service = new NotificationService();
        $result = $service->send();

        $this->assertTrue($result);
    }
}
```

# 실제 활용 사례

## overload: 활용 예시

```php
class UserRepository {
    public function create($data)
    {
        $validator = new DataValidator();
        if ($validator->validate($data)) {
            return User::create($data);
        }
        return false;
    }
}

class UserRepositoryTest extends TestCase
{
    public function testCreateUser()
    {
        // DataValidator 인스턴스 생성을 가로챔
        $validatorMock = Mockery::mock('overload:DataValidator');
        $validatorMock->shouldReceive('validate')
                     ->once()
                     ->andReturn(true);

        $repo = new UserRepository();
        $result = $repo->create(['name' => 'Test User']);
        
        $this->assertInstanceOf(User::class, $result);
    }
}
```

## alias: 활용 예시

```php
class EmailService {
    public function sendWelcomeEmail($user)
    {
        if (Config::get('mail.enabled')) {
            return Mailer::send('welcome', ['user' => $user]);
        }
        return false;
    }
}

class EmailServiceTest extends TestCase
{
    public function testSendWelcomeEmail()
    {
        // Config와 Mailer의 정적 메서드를 Mock으로 대체
        $configMock = Mockery::mock('alias:Config');
        $configMock->shouldReceive('get')
                  ->with('mail.enabled')
                  ->andReturn(true);

        $mailerMock = Mockery::mock('alias:Mailer');
        $mailerMock->shouldReceive('send')
                  ->once()
                  ->andReturn(true);

        $service = new EmailService();
        $result = $service->sendWelcomeEmail(['name' => 'Test User']);
        
        $this->assertTrue($result);
    }
}
```

# 사용 시 주의사항

## overload: 주의사항
- 테스트 실행 중 해당 클래스의 모든 인스턴스 생성에 영향
- 한 번 overload된 클래스는 해당 테스트 실행 동안 계속 적용
- 다른 테스트에 영향을 줄 수 있으므로 tearDown에서 정리 필요

## alias: 주의사항
- 정적 메서드만 mock 가능
- 일반 인스턴스 메서드는 영향받지 않음
- 전역적으로 적용되므로 다른 테스트와의 격리 주의

# 선택 가이드

## overload: 사용 시기
- new 키워드로 생성되는 객체를 대체해야 할 때
- 의존성 주입이 어려운 레거시 코드 테스트
- 인스턴스 생성 과정을 제어해야 할 때

## alias: 사용 시기
- 정적 메서드를 사용하는 코드 테스트
- Laravel Facade 패턴 테스트
- 전역적으로 사용되는 유틸리티 함수 테스트

# 성능 고려사항

## overload:
- 클래스 로더 수준에서 동작하므로 약간의 성능 영향
- 많은 수의 overload를 사용할 경우 테스트 실행 속도 저하 가능

## alias:
- 정적 호출만 영향받으므로 상대적으로 가벼움
- 메모리 사용량이 적음

# 결론

Laravel 테스트에서 'overload:'와 'alias:'는 각각 인스턴스 생성과 정적 메서드 호출을 Mock하는 데 특화된 패턴이다. 테스트 대상 코드의 특성과 의존성 패턴에 따라 적절한 방식을 선택해야 한다.