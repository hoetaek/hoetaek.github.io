---
title: Coroutine과 asyncio
publish: false
tags:
---
# asyncio 기본 개념

asyncio는 Python의 비동기 프로그래밍을 위한 라이브러리다. 코루틴을 사용하여 동시성 코드를 작성할 수 있게 해준다.

# 이벤트 루프 실행 방법

## 간단한 방법
```python
import asyncio

async def hello():
    return "Hello, World!"

# Python 3.7+
result = asyncio.run(hello())
```

## 전통적인 방법
```python
async def main():
    return "Hello, World!"

# 이벤트 루프 직접 제어
loop = asyncio.get_event_loop()
result = loop.run_until_complete(main())
loop.close()
```

# 코루틴 실행 방법

## 1. await 키워드 사용
```python
async def fetch_data():
    await asyncio.sleep(1)  # 비동기 대기
    return "Data fetched"

async def main():
    # await로 코루틴 직접 실행
    result = await fetch_data()
    print(result)  # "Data fetched"

asyncio.run(main())
```

## 2. create_task 사용
```python
async def long_operation(name, seconds):
    print(f"{name} 시작")
    await asyncio.sleep(seconds)
    print(f"{name} 완료")
    return f"{name} 결과"

async def main():
    # 태스크 생성 및 백그라운드 실행
    task1 = asyncio.create_task(long_operation("작업1", 2))
    task2 = asyncio.create_task(long_operation("작업2", 1))
    
    # 다른 작업 수행 가능
    print("다른 작업 수행 중...")
    
    # 나중에 결과 수집
    result1 = await task1
    result2 = await task2
    
    print(result1, result2)

asyncio.run(main())
```

## 3. gather 사용
```python
async def process_item(item):
    await asyncio.sleep(1)
    return f"처리된 {item}"

async def main():
    # 여러 코루틴을 동시에 실행
    items = ["A", "B", "C"]
    results = await asyncio.gather(
        *[process_item(item) for item in items]
    )
    print(results)  # ['처리된 A', '처리된 B', '처리된 C']

asyncio.run(main())
```

# 실제 활용 예제

## 웹 API 호출
```python
import aiohttp
import asyncio

async def fetch_url(session, url):
    async with session.get(url) as response:
        return await response.text()

async def main():
    urls = [
        "https://api.example.com/data1",
        "https://api.example.com/data2",
        "https://api.example.com/data3"
    ]
    
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        
        for url, result in zip(urls, results):
            print(f"{url}: {len(result)} bytes")

asyncio.run(main())
```

## 데이터베이스 작업
```python
import asyncpg

async def process_users():
    conn = await asyncpg.connect(
        user='user', password='password',
        database='database', host='localhost'
    )
    
    try:
        # 동시 쿼리 실행
        queries = [
            conn.fetch("SELECT * FROM users WHERE age > $1", age)
            for age in range(20, 30)
        ]
        
        results = await asyncio.gather(*queries)
        return results
        
    finally:
        await conn.close()
```

# 실행 방법별 차이점

## await vs create_task
```python
async def demonstration():
    print("시작")
    
    # await 사용 - 순차 실행
    start = time.time()
    await asyncio.sleep(1)
    await asyncio.sleep(1)
    print(f"await 완료: {time.time() - start}초")  # ~2초
    
    # create_task 사용 - 동시 실행
    start = time.time()
    task1 = asyncio.create_task(asyncio.sleep(1))
    task2 = asyncio.create_task(asyncio.sleep(1))
    await task1
    await task2
    print(f"create_task 완료: {time.time() - start}초")  # ~1초
```

## gather vs create_task
```python
async def comparison():
    # create_task 사용
    tasks = []
    for i in range(3):
        task = asyncio.create_task(long_operation(f"task{i}", 1))
        tasks.append(task)
    
    # 개별적으로 await
    for task in tasks:
        result = await task
        print(result)
    
    # gather 사용 - 더 간단
    results = await asyncio.gather(
        *[long_operation(f"gather{i}", 1) for i in range(3)]
    )
    print(results)
```

# 성능 최적화

## 세마포어를 사용한 동시성 제어
```python
async def limited_concurrency():
    # 최대 3개의 동시 작업으로 제한
    semaphore = asyncio.Semaphore(3)
    
    async def controlled_task(name):
        async with semaphore:
            return await long_operation(name, 1)
    
    tasks = [controlled_task(f"task{i}") for i in range(10)]
    results = await asyncio.gather(*tasks)
    return results
```

# 오류 처리

## 예외 처리
```python
async def error_handling():
    try:
        # gather에서의 예외 처리
        results = await asyncio.gather(
            long_operation("task1", 1),
            long_operation("task2", 1),
            return_exceptions=True  # 예외를 결과로 반환
        )
        
        # create_task에서의 예외 처리
        task = asyncio.create_task(long_operation("task3", 1))
        try:
            await task
        except Exception as e:
            print(f"작업 실패: {e}")
```

# 주의사항

## 1. 블로킹 작업 처리
```python
async def blocking_operation():
    # CPU 집약적인 작업은 ThreadPoolExecutor 사용
    loop = asyncio.get_event_loop()
    result = await loop.run_in_executor(
        None, time.sleep, 1
    )
```

## 2. 코루틴 취소
```python
async def cancellation_demo():
    try:
        task = asyncio.create_task(long_operation("task", 10))
        await asyncio.sleep(2)
        task.cancel()  # 작업 취소
        await task
    except asyncio.CancelledError:
        print("작업이 취소됨")
```

# 결론

asyncio와 코루틴은 Python에서 효율적인 비동기 프로그래밍을 가능하게 한다. await, create_task, gather는 각각 다른 상황에서 유용하게 사용될 수 있으며, 적절한 사용을 통해 성능을 최적화할 수 있다. 특히 I/O 바운드 작업에서 큰 성능 향상을 기대할 수 있다.