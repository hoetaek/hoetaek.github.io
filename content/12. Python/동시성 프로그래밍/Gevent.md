---
title: 무제 파일
publish: false
tags:
---
# Gevent 기본 개념

Gevent는 libev 기반의 동시성 라이브러리다. 네트워크와 동시성 작업을 위한 다양한 API를 제공하며, Greenlet을 통해 경량 코루틴을 구현한다.

# Greenlet 이해하기

Greenlet은 경량 코루틴으로, 메인 프로세스 내에서 동작하는 실행 단위다. OS 수준의 쓰레드나 프로세스와는 다르게, 하나의 쓰레드 안에서 협력적으로 실행된다.

```python
import gevent

def foo():
    print('Running in foo')
    gevent.sleep(0)
    print('Explicit context switch to foo again')

def bar():
    print('Explicit context to bar')
    gevent.sleep(0)
    print('Implicit context switch back to bar')

# Greenlet 생성 및 실행
gevent.joinall([
    gevent.spawn(foo),
    gevent.spawn(bar),
])
```

# 동기 vs 비동기 실행

## 비동기 실행의 이점
```python
import gevent
import random

def task(pid):
    gevent.sleep(random.randint(0,2)*0.001)
    print(f'Task {pid} done')

def synchronous():
    for i in range(1,10):
        task(i)

def asynchronous():
    threads = [gevent.spawn(task, i) for i in range(1,10)]
    gevent.joinall(threads)

print('Synchronous:')
synchronous()

print('Asynchronous:')
asynchronous()
```

# Monkey Patching

Monkey patching은 파이썬 기본 라이브러리를 gevent 호환 버전으로 대체하는 기술이다.

```python
from gevent import monkey

# 모든 기본 라이브러리를 패치
monkey.patch_all()

# 또는 특정 모듈만 패치
monkey.patch_socket()
monkey.patch_ssl()
monkey.patch_select()
```

# 동시성 제어 도구들

## 1. Event
```python
from gevent.event import Event

evt = Event()

def waiter():
    print("대기 시작")
    evt.wait()  # 이벤트가 set될 때까지 대기
    print("대기 종료")

def setter():
    print("3초 후 이벤트 설정")
    gevent.sleep(3)
    evt.set()

gevent.joinall([
    gevent.spawn(waiter),
    gevent.spawn(setter),
])
```

## 2. Queue
```python
from gevent.queue import Queue

tasks = Queue(maxsize=3)  # 크기가 제한된 큐

def worker(name):
    while not tasks.empty():
        task = tasks.get()
        print(f'Worker {name}: {task}')
        gevent.sleep(0)

def boss():
    for i in range(1,10):
        tasks.put(i)

gevent.spawn(boss).join()
workers = [gevent.spawn(worker, f'worker-{i}') for i in range(3)]
gevent.joinall(workers)
```

# 실전 응용

## WSGI 서버 구현
```python
from gevent.pywsgi import WSGIServer

def application(environ, start_response):
    status = '200 OK'
    headers = [('Content-Type', 'text/html')]
    start_response(status, headers)
    
    return [b"<h1>Hello World</h1>"]

server = WSGIServer(('127.0.0.1', 8000), application)
server.serve_forever()
```

## 웹소켓 서버
```python
from gevent import pywsgi
from geventwebsocket.handler import WebSocketHandler

def websocket_app(environ, start_response):
    ws = environ['wsgi.websocket']
    while True:
        message = ws.receive()
        if message is None:
            break
        ws.send(message)

server = pywsgi.WSGIServer(('', 8000), websocket_app,
                          handler_class=WebSocketHandler)
server.serve_forever()
```

# 성능 최적화

## Pool 사용
```python
from gevent.pool import Pool

pool = Pool(2)

def process_item(item):
    gevent.sleep(0.1)  # 시뮬레이션된 I/O
    return item * 2

# 병렬 처리 제한
results = pool.map(process_item, range(100))
```

# 주의사항

1. CPU 바운드 작업은 피해야 한다
2. Monkey patching은 신중하게 사용해야 한다
3. 블로킹 연산은 gevent 호환 버전을 사용해야 한다
4. 전역 상태 공유는 피해야 한다

# 모범 사례

```python
# 리소스 정리가 보장되는 패턴
from contextlib import contextmanager

@contextmanager
def gevent_timeout(seconds):
    timeout = gevent.Timeout(seconds)
    timeout.start()
    try:
        yield
    finally:
        timeout.cancel()

with gevent_timeout(2):
    gevent.sleep(1)  # OK
    # gevent.sleep(3)  # 예외 발생
```

# 결론

Gevent는 비동기 프로그래밍을 위한 강력한 도구다. Greenlet을 통한 경량 코루틴, 다양한 동시성 제어 도구, 그리고 네트워크 프로그래밍을 위한 기능들을 제공한다. 특히 I/O 바운드 작업에서 뛰어난 성능을 발휘하며, 웹 서버나 네트워크 서비스 구현에 적합하다.