---
title: Concurrent.Futures
publish: false
tags:
---
# Futures 개요

Futures는 Python에서 비동기 작업을 처리하기 위한 고수준 인터페이스를 제공한다. JavaScript의 Promise와 유사한 개념으로, 비동기 작업의 결과를 나타내는 객체다.

# 기본 사용법

## Executor 생성과 작업 제출
```python
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor

def process_item(x):
    return x * x

# ThreadPoolExecutor 사용
with ThreadPoolExecutor(max_workers=3) as executor:
    future = executor.submit(process_item, 5)
    result = future.result()
    print(f"Result: {result}")  # Result: 25
```

# 주요 메서드

## 1. executor.submit()
```python
def long_running_task(name):
    time.sleep(2)
    return f"Task {name} completed"

with ThreadPoolExecutor(max_workers=3) as executor:
    # 작업 제출
    future1 = executor.submit(long_running_task, "A")
    future2 = executor.submit(long_running_task, "B")
```

## 2. future.result()
```python
def process_data(data):
    time.sleep(random.random())
    return f"Processed {data}"

with ThreadPoolExecutor(max_workers=2) as executor:
    future = executor.submit(process_data, "sample")
    try:
        # 타임아웃 설정
        result = future.result(timeout=3)
        print(result)
    except TimeoutError:
        print("작업이 시간 초과되었습니다")
```

## 3. future.done()
```python
def check_status(future):
    if future.done():
        print("작업이 완료되었습니다")
        print(f"결과: {future.result()}")
    else:
        print("작업이 아직 진행 중입니다")

with ThreadPoolExecutor() as executor:
    future = executor.submit(long_running_task, "test")
    check_status(future)  # 작업이 아직 진행 중입니다
    time.sleep(2)
    check_status(future)  # 작업이 완료되었습니다
```

## 4. future.cancel()
```python
def cancellable_task():
    for i in range(5):
        time.sleep(1)
        print(f"Step {i+1}")

with ThreadPoolExecutor() as executor:
    future = executor.submit(cancellable_task)
    time.sleep(2)  # 2초 후 작업 취소
    
    if future.cancel():
        print("작업이 취소되었습니다")
    else:
        print("작업을 취소할 수 없습니다")
```

## 5. as_completed()
```python
from concurrent.futures import as_completed

def process_url(url):
    time.sleep(random.random())
    return f"Processed {url}"

urls = [f"http://example.com/{i}" for i in range(5)]

with ThreadPoolExecutor(max_workers=3) as executor:
    futures = [executor.submit(process_url, url) for url in urls]
    
    for future in as_completed(futures):
        try:
            result = future.result()
            print(result)
        except Exception as e:
            print(f"작업 실행 중 오류 발생: {e}")
```

# ThreadPoolExecutor 활용

## 웹 크롤링 예제
```python
import requests
from concurrent.futures import ThreadPoolExecutor

def fetch_url(url):
    try:
        response = requests.get(url)
        return f"{url}: {response.status_code}"
    except Exception as e:
        return f"{url}: Error - {str(e)}"

urls = [
    "https://python.org",
    "https://google.com",
    "https://github.com"
]

with ThreadPoolExecutor(max_workers=3) as executor:
    results = list(executor.map(fetch_url, urls))

for result in results:
    print(result)
```

## 이미지 처리 예제
```python
from PIL import Image
import os

def process_image(image_path):
    with Image.open(image_path) as img:
        # 이미지 크기 조정
        img_resized = img.resize((800, 600))
        # 저장
        save_path = f"processed_{os.path.basename(image_path)}"
        img_resized.save(save_path)
        return save_path

image_files = ["image1.jpg", "image2.jpg", "image3.jpg"]

with ThreadPoolExecutor(max_workers=3) as executor:
    futures = [executor.submit(process_image, img) for img in image_files]
    
    for future in as_completed(futures):
        print(f"Processed: {future.result()}")
```

# ProcessPoolExecutor 활용

## CPU 집약적 작업 처리
```python
def calculate_prime_numbers(n):
    primes = []
    for num in range(2, n):
        if all(num % i != 0 for i in range(2, int(num ** 0.5) + 1)):
            primes.append(num)
    return len(primes)

numbers = [100000, 200000, 300000, 400000]

with ProcessPoolExecutor() as executor:
    results = executor.map(calculate_prime_numbers, numbers)
    
    for n, count in zip(numbers, results):
        print(f"Found {count} prime numbers until {n}")
```

# 성능 최적화 팁

## 1. 적절한 Executor 선택
```python
def choose_executor(task_type):
    if task_type == 'io':
        # I/O 바운드 작업에는 ThreadPoolExecutor
        return ThreadPoolExecutor(max_workers=4)
    elif task_type == 'cpu':
        # CPU 바운드 작업에는 ProcessPoolExecutor
        return ProcessPoolExecutor(max_workers=os.cpu_count())
```

## 2. 작업 그룹화
```python
def process_batch(items):
    results = []
    for item in items:
        results.append(process_item(item))
    return results

# 큰 데이터셋을 배치로 나누기
large_dataset = list(range(1000))
batch_size = 100
batches = [large_dataset[i:i+batch_size] 
           for i in range(0, len(large_dataset), batch_size)]

with ProcessPoolExecutor() as executor:
    results = list(executor.map(process_batch, batches))
```

# 주의사항

## 1. 리소스 관리
```python
def safe_executor_usage():
    try:
        with ThreadPoolExecutor(max_workers=4) as executor:
            future = executor.submit(long_running_task)
            return future.result(timeout=5)
    except TimeoutError:
        print("작업 시간 초과")
    except Exception as e:
        print(f"예외 발생: {e}")
```

## 2. 데드락 방지
```python
# 잘못된 예
def recursive_task(executor, depth):
    if depth <= 0:
        return
    # 데드락 발생 가능
    future = executor.submit(recursive_task, executor, depth - 1)
    future.result()

# 올바른 예
def safe_recursive_task(depth):
    if depth <= 0:
        return
    return depth + safe_recursive_task(depth - 1)

with ThreadPoolExecutor() as executor:
    future = executor.submit(safe_recursive_task, 5)
```

# 결론

concurrent.futures 모듈은 Python에서 동시성 프로그래밍을 쉽게 구현할 수 있게 해준다. ThreadPoolExecutor는 I/O 바운드 작업에, ProcessPoolExecutor는 CPU 바운드 작업에 적합하다. 적절한 Executor 선택과 리소스 관리를 통해 효율적인 동시성 프로그래밍을 구현할 수 있다.