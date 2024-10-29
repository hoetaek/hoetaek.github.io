---
title: 무제 파일
publish: false
tags:
---
Laravel의 큐 시스템은 무거운 작업을 비동기적으로 처리하여 애플리케이션의 응답 시간을 개선할 수 있게 해준다. 하지만 큐를 사용할 때는 여러 가지 고려해야 할 사항들이 있다.

## 1. 작업 시간 관리

### 타임아웃 설정
```php
class ProcessPodcast implements ShouldQueue
{
    public $timeout = 120; // 2분
    public $tries = 3;    // 재시도 횟수
    
    public function handle()
    {
        // 긴 작업 처리
    }
}
```

### 작업 시간 초과 시 대처 방법
```php
class ProcessPodcast implements ShouldQueue
{
    public function handle()
    {
        // 1. 작업을 더 작은 단위로 나누기
        $chunks = $this->data->chunk(100);
        foreach ($chunks as $chunk) {
            ProcessChunk::dispatch($chunk);
        }
        
        // 2. 진행 상황 저장
        Cache::put('process_progress', 50);
    }
    
    public function retryUntil()
    {
        // 최대 24시간까지만 재시도
        return now()->addHours(24);
    }
}
```

## 2. 데이터 직렬화 문제

### 잘못된 방식
```php
class SendWelcomeEmail implements ShouldQueue
{
    private $user;
    private $temporaryData;

    public function __construct(User $user)
    {
        // 잘못된 예: private 속성은 직렬화되지 않는다
        $this->user = $user;
        $this->temporaryData = Session::get('temp_data');
    }
}
```

### 올바른 방식
```php
class SendWelcomeEmail implements ShouldQueue
{
    public $user;
    public array $additionalData;

    public function __construct(User $user, array $data)
    {
        // public 속성 사용
        $this->user = $user;
        $this->additionalData = $data;
    }
    
    public function handle()
    {
        if (!$this->user) {
            return;  // 모델이 삭제된 경우 처리
        }
    }
}
```

## 3. 메모리 관리

### 메모리 누수 방지
```php
class ProcessLargeDataset implements ShouldQueue
{
    public function handle()
    {
        // 1. 청크 단위로 처리
        User::chunk(100, function ($users) {
            foreach ($users as $user) {
                // 처리 로직
            }
            
            // 메모리 해제
            unset($users);
            gc_collect_cycles();
        });
        
        // 2. 커서 사용
        foreach (User::cursor() as $user) {
            // 처리 로직
        }
    }
}
```

## 4. 동시성 문제

### 레이스 컨디션 방지
```php
class UpdateUserStatus implements ShouldQueue
{
    public function handle()
    {
        // 잘못된 방식
        $user = User::find($this->userId);
        $user->status = 'active';
        $user->save();
        
        // 올바른 방식
        DB::transaction(function () {
            $user = User::lockForUpdate()->find($this->userId);
            $user->status = 'active';
            $user->save();
        });
    }
}
```

### 유니크 작업 처리
```php
class ProcessPayment implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
    
    public function uniqueId()
    {
        return $this->orderId;  // 주문당 하나의 작업만 큐에 들어간다
    }
}
```

## 5. 실패 처리

### 실패 감지와 로깅
```php
class SendNewsletter implements ShouldQueue
{
    public function failed($exception)
    {
        // 실패 로깅
        Log::error('뉴스레터 발송 실패', [
            'exception' => $exception->getMessage(),
            'user_id' => $this->userId
        ]);
        
        // 관리자에게 알림
        Notification::send(
            Admin::all(),
            new JobFailedNotification($this, $exception)
        );
    }
}
```

### 실패 후 정리
```php
class ProcessUpload implements ShouldQueue
{
    public function failed($exception)
    {
        // 임시 파일 정리
        Storage::delete($this->temporaryPath);
        
        // 진행 상태 초기화
        Cache::forget('upload_progress_' . $this->fileId);
    }
}
```

## 6. 의존성 주입 주의사항

### 잘못된 방식
```php
class SendNotification implements ShouldQueue
{
    private $service;
    
    public function __construct(NotificationService $service)
    {
        // 잘못됨: 직렬화할 수 없는 의존성
        $this->service = $service;
    }
}
```

### 올바른 방식
```php
class SendNotification implements ShouldQueue
{
    public function handle(NotificationService $service)
    {
        // 올바름: 핸들러에서 의존성 주입받기
        $service->send();
    }
}
```

## 7. 큐 모니터링과 관리

### 실패한 작업 모니터링
```php
// failed_jobs 테이블 확인
$failedJobs = DB::table('failed_jobs')
    ->orderBy('failed_at', 'desc')
    ->get();

// Horizon 대시보드 사용 (권장)
class HorizonServiceProvider extends HorizonApplicationServiceProvider
{
    protected function gate()
    {
        Gate::define('viewHorizon', function ($user) {
            return $user->isAdmin();
        });
    }
}
```

## 결론

Laravel의 큐 시스템을 사용할 때는 다음 사항들을 특히 주의해야 한다:

1. 작업 시간은 항상 제한을 두고 관리한다
2. 데이터 직렬화 시 public 속성만 사용한다
3. 메모리 관리를 철저히 한다
4. 동시성 문제를 고려하여 설계한다
5. 실패 처리를 반드시 구현한다
6. 의존성 주입은 handle 메소드에서 한다
7. 큐 작업을 모니터링하고 관리한다

이러한 주의사항들을 잘 지키면 안정적이고 효율적인 큐 시스템을 구축할 수 있다.