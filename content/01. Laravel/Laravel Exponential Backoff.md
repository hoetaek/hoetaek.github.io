---
publish: false
tags:
---
# 개념 이해
Exponential Backoff는 작업 실패 시 재시도 간격을 점진적으로 늘려가는 알고리즘이다.

```mermaid
graph LR
    A[첫 시도 실패] --> B[2초 대기]
    B --> C[4초 대기]
    C --> D[8초 대기]
    D --> E[16초 대기]
    E --> F[최대 대기 시간]
```

# Laravel 구현 방법

## 1. Job Retry 구현
```php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;

class ProcessPodcast implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    /**
     * 최대 재시도 횟수를 지정한다
     */
    public $tries = 5;

    /**
     * 재시도 간격을 배열로 정의한다
     */
    public function backoff()
    {
        return [2, 4, 8, 16, 32];
    }

    /**
     * 작업을 실행한다
     */
    public function handle()
    {
        // 작업 로직을 구현한다
    }
}
```

## 2. HTTP Client 구현
```php
use Illuminate\Support\Facades\Http;

class ExternalApiService
{
    public function fetchData()
    {
        return Http::retry(3, function ($exception, $request) {
            // 연결 실패 시에만 재시도한다
            return $exception instanceof ConnectionException;
        }, function ($exception, $request) {
            $attempt = $request->tries();
            // 재시도 간격을 2의 제곱으로 증가시킨다
            return (2 ** $attempt) * 1000;
        })->get('api.example.com/data');
    }
}
```

## 3. 커스텀 재시도 구현
```php
namespace App\Services;

use Illuminate\Support\Facades\Log;

class RetryService
{
    public function executeWithBackoff(callable $operation, int $maxAttempts = 5)
    {
        for ($attempt = 1; $attempt <= $maxAttempts; $attempt++) {
            try {
                return $operation();
            } catch (\Exception $e) {
                if ($attempt === $maxAttempts) {
                    throw $e;
                }

                $delay = $this->calculateDelay($attempt);
                Log::warning("작업 실패, {$delay}초 후 재시도", [
                    'attempt' => $attempt,
                    'exception' => $e->getMessage()
                ]);

                sleep($delay);
            }
        }
    }

    private function calculateDelay(int $attempt): int
    {
        // 최대 30초까지만 대기한다
        return min(30, (2 ** ($attempt - 1)));
    }
}
```

# 실제 활용 예시

## 1. Queue Worker 설정
```php
// config/queue.php에서 재시도 설정을 정의한다
return [
    'default' => env('QUEUE_CONNECTION', 'redis'),
    'connections' => [
        'redis' => [
            'driver' => 'redis',
            'connection' => 'default',
            'queue' => env('REDIS_QUEUE', 'default'),
            'retry_after' => 90,
            'block_for' => null,
        ],
    ],
];
```

## 2. API 통합
```php
namespace App\Services;

use Illuminate\Support\Facades\Http;
use App\Exceptions\ApiException;

class PaymentService
{
    public function processPayment($orderId)
    {
        // 결제 API 호출을 재시도한다
        return Http::retry(3, function ($exception) {
            return $exception instanceof ConnectionException;
        }, function ($attempt) {
            return [2000, 4000, 8000][$attempt - 1] ?? 8000;
        })->post('payment.example.com/process', [
            'order_id' => $orderId,
        ])->throw(function ($response, $e) {
            throw new ApiException('결제 처리 실패: ' . $e->getMessage());
        });
    }
}
```

# 모니터링 구현

## 1. 로그 설정
```php
// config/logging.php에서 재시도 로그 채널을 설정한다
'channels' => [
    'retry' => [
        'driver' => 'daily',
        'path' => storage_path('logs/retry.log'),
        'level' => 'debug',
        'days' => 14,
    ],
],
```

## 2. 모니터링 코드
```php
namespace App\Monitoring;

class RetryMonitor
{
    public function handleRetry($job, $attempts)
    {
        // 재시도 정보를 로그에 기록한다
        \Log::channel('retry')->info('작업 재시도', [
            'job' => get_class($job),
            'attempt' => $attempts,
            'queue' => $job->queue
        ]);

        // 메트릭을 기록한다
        \Metrics::increment('job.retries', 1, [
            'job_type' => get_class($job)
        ]);
    }
}
```

# 주의사항

1. 재시도 횟수는 적절하게 제한한다
2. 재시도 간격은 시스템 부하를 고려하여 설정한다
3. 일시적인 오류만 재시도하도록 구현한다
4. 영구적인 오류는 즉시 실패 처리한다

# 결론

Laravel의 Exponential Backoff를 사용하면:
1. 시스템의 안정성이 향상된다
2. 일시적인 오류를 효과적으로 처리할 수 있다
3. 시스템 리소스를 효율적으로 사용할 수 있다
4. 장애 상황에서 복원력이 강화된다