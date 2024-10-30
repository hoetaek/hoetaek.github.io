---
title: 이벤트와 리스너(Events & Listeners)
publish: true
tags:
---
Laravel에서 이벤트와 리스너는 애플리케이션의 다양한 부분 간에 느슨한 결합을 구현하기 위한 강력한 도구다. 이벤트는 애플리케이션에서 발생하는 동작을 나타내며, 리스너는 이러한 이벤트에 반응하여 특정 작업을 수행한다.

## 이벤트의 필수 요구사항

### 1. 필수 Trait
이벤트 클래스는 다음 세 가지 trait를 반드시 사용해야 한다:
- `Dispatchable`: 이벤트를 발생시키는 데 필요하다
- `InteractsWithSockets`: 브로드캐스팅 기능을 지원한다
- `SerializesModels`: Eloquent 모델의 직렬화를 처리한다

### 2. 속성 정의
이벤트에서 사용할 데이터는 public 속성으로 정의해야 한다.

## 기본 구조

### 1. 이벤트 생성
```php
php artisan make:event UserRegistered
```

생성된 이벤트 클래스의 기본 구조 (속성 프로모션 사용):
```php
namespace App\Events;

use App\Models\User;
use Illuminate\Broadcasting\InteractsWithSockets;
use Illuminate\Foundation\Events\Dispatchable;
use Illuminate\Queue\SerializesModels;

class UserRegistered
{
    use Dispatchable, InteractsWithSockets, SerializesModels;
    
    // 속성 프로모션을 사용한 생성자
    public function __construct(
        public User $user,
        public string $referralCode,
        public bool $isNewCustomer = true
    ) {}
}
```

### 2. 리스너 생성
```php
php artisan make:listener SendWelcomeEmail --event=UserRegistered
```

## 리스너의 필수 요구사항

### 1. handle 메소드
모든 리스너는 반드시 handle 메소드를 구현해야 한다.

```php
namespace App\Listeners;

use App\Events\UserRegistered;

class SendWelcomeEmail
{
    public function handle(UserRegistered $event)
    {
        // 이벤트 객체에서 프로모션된 속성들에 직접 접근
        $user = $event->user;
        $referralCode = $event->referralCode;
        $isNewCustomer = $event->isNewCustomer;
        
        // 웰컴 이메일 발송 로직
    }
}
```

### 2. EventServiceProvider 등록
```php
protected $listen = [
    UserRegistered::class => [
        SendWelcomeEmail::class,
        UpdateUserStats::class,
    ],
];
```

## 리스너의 주요 특징

### 1. 생성자 주입
리스너는 생성자를 통해 의존성을 주입받을 수 있다.

```php
class SendWelcomeEmail
{
    protected $mailer;

    public function __construct(Mailer $mailer)
    {
        $this->mailer = $mailer;
    }

    public function handle(UserRegistered $event)
    {
        // 메일러 사용
    }
}
```

### 2. 큐 사용
리스너를 큐에서 실행하려면 ShouldQueue 인터페이스를 구현해야 한다.

```php
use Illuminate\Contracts\Queue\ShouldQueue;

class SendWelcomeEmail implements ShouldQueue
{
    public $connection = 'redis';
    public $queue = 'listeners';
    public $delay = 60;
}
```

## 실제 사용 예시

### 1. 이벤트 발생시키기
```php
use App\Events\UserRegistered;

class RegisterController extends Controller
{
    public function register(Request $request)
    {
        $user = User::create($request->all());
        
        // 이벤트 발생 시 생성자에 정의된 모든 필수 매개변수 전달
        event(new UserRegistered(
            user: $user,
            referralCode: $request->referral_code ?? 'DIRECT'
        ));
        
        // 또는 명명된 인수 없이
        UserRegistered::dispatch($user, 'DIRECT');
    }
}
```

### 2. 다중 리스너 구현
```php
protected $listen = [
    UserRegistered::class => [
        SendWelcomeEmail::class,
        NotifyAdministrator::class,
        UpdateUserMetrics::class,
    ],
];
```

## 리스너의 고급 기능

### 1. 실패 처리
```php
class SendWelcomeEmail implements ShouldQueue
{
    public function failed(UserRegistered $event, $exception)
    {
        // 실패 처리 로직
        Log::error('이메일 전송 실패', [
            'user' => $event->user->id,
            'error' => $exception->getMessage()
        ]);
    }
}
```

### 2. 조건부 실행
```php
public function shouldHandle(UserRegistered $event)
{
    return $event->user->isActive();
}
```

## 이벤트 사용 시 주의사항

1. public 속성만 직렬화된다
2. private나 protected 속성은 큐나 브로드캐스팅에서 사용할 수 없다
3. 속성 프로모션을 사용할 때는 타입 힌트를 명확히 지정하는 것이 좋다
4. Eloquent 모델을 사용할 때는 반드시 SerializesModels trait를 사용해야 한다

## 리스너 사용 시 주의사항

1. 리스너는 가능한 한 작은 단위의 책임만 가져야 한다
2. 큐를 사용할 때는 작업 시간을 고려해야 한다
3. 중요한 데이터는 항상 이벤트 객체에 포함시켜야 한다
4. 순환 참조를 피해야 한다

## 장점

1. 코드의 결합도를 낮췄다
2. 확장성이 향상됐다
3. 비동기 처리가 용이하다
4. 테스트가 쉬워졌다

## 성능 최적화

### 1. 배치 처리
```php
class SendWelcomeEmail implements ShouldQueue
{
    public $batch = true;
    public $batchSize = 100;
}
```

### 2. 타임아웃 설정
```php
class SendWelcomeEmail implements ShouldQueue
{
    public $timeout = 120;
    public $tries = 3;
    
    public function retryUntil()
    {
        return now()->addHours(24);
    }
}
```

### 3. 메모리 관리
```php
class SendWelcomeEmail implements ShouldQueue
{
    public $deleteWhenMissingModels = true;
    
    public function handle(UserRegistered $event)
    {
        if (!$event->user) {
            return;
        }
        
        // 처리 후 메모리 해제
        unset($event->user);
    }
}
```

## 결론

Laravel의 이벤트와 리스너 시스템은 애플리케이션의 다양한 부분을 효과적으로 분리하고 관리할 수 있게 해준다. 이벤트는 필수 trait들을 사용하고 public 속성으로 데이터를 정의해야 하며, 리스너는 반드시 handle 메소드를 구현해야 한다. 특히 PHP 8.0 이상에서는 생성자 속성 프로모션을 활용하여 더 간결하고 명확한 코드를 작성할 수 있다. 이벤트와 리스너를 적절히 활용하면 확장 가능하고 유지보수가 쉬운 애플리케이션을 구축할 수 있으며, 대규모 애플리케이션에서는 이러한 시스템을 통해 복잡성을 효과적으로 관리할 수 있다.