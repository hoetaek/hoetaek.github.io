---
title: python
tags:
---
# Cocurrent jobs
## Futures
js에서의 Promise와 같은 것이다
futures.ThreadPoolExecutor, futures.ProcessPoolExecutor를 이용해서 과거보다 더 쉽게 multithreading과 multiprocessing을 사용할 수 있다
1. executor.submit
2. future.result
3. future.done
4. future.cancel
5. concurrent.futures.as_completed
위와 같은 메소드들을 사용할 수 있다
## Coroutine과 asyncio
asyncio는 event loop을 받아와서 coroutine을 실행시킨다. 그 절차를 간단하게 한 코드는 다음과 같다
`asyncio.run()`
아래는 긴 버전이다
```python
loop = asyncio.get_event_loop()
loop.run_until_complete(print_add(1, 2))
loop.close()
```
함수 안에서 coroutine을 실행하는 방법은 다음과 같이 3가지가 있다
1. await
2. create_task
3. gather
# Pytest

## fixture
- 공통으로 쓰이는 변수를 fixture로 사용할 수 있다
- convention으로 conftest.py에 다른 테스트에서 공통으로 쓰이는 fixture를 정의함
- conftest와 같은 위치 혹은 하위 폴더에 위치한 파일들은 모두 그 fixture를 사용할 수 있다
## test 중에 sql문 출력
```python
from django.test.utils import CaptureQueriesContext  
from django.db import connection  
  
with CaptureQueriesContext(connection) as ctx:
```
위 코드를 테스트 메소드 안에 넣어서 실행하면 된다 
대신에 아래와 같이 django logger이 django.db.backends로 설정되어 있으며 log level은 debug여야 한다
```python
LOGGING = {  
    "version": 1,  
    "disable_existing_loggers": False,  
    "handlers": {  
        "console": {  
            "level": "DEBUG",  
            "class": "logging.StreamHandler",  
        }  
    },  
    "loggers": {  
        "django.db.backends": {  
            "handlers": ["console"],  
            "level": "DEBUG",  
        },  
    },  
}
```
## Query 최대 개수 설정
django-pytest에는 **DjangoAssertNumQueries**, DjangoCaptureOnCommitCallbacks, DjangoDbBlocker가 있다. 이 중에서 DjangoAssertNumQueries를 이용하면 컨텍스트 내에서 원하는 수만큼의 쿼리 수로 실행했는지 확인할 수 있다
```python
def test_사용자_목록을_확인할_쿼리가_3개_필요하다(api_client):
	with django_assert_max_num_queries(3):  
	    res = api_client.get("/users")  
	    assert res.status_code == status.HTTP_200_OK  
	    assert len(res.json()) == 10
```
## pytest 병렬적으로 돌리기
```sh
pip install pytest-xdist
pytest -n 3
```
# Poetry
## poetry로 프로젝트 내부에 virtualenv 환경 구축하는 방법
```sh
poetry config virtualenvs.in-project true
poetry config virtualenvs.path "./.venv"

# 프로젝트 내부에 venv 새로 설치
poetry install && poetry update
```
[링크](https://amazingguni.medium.com/python-poetry%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EB%A5%BC-vscode%EC%97%90%EC%84%9C-%EA%B0%9C%EB%B0%9C%ED%95%A0-%EB%95%8C-interpreter%EB%A5%BC-%EC%9E%A1%EB%8A%94-%EB%B0%A9%EB%B2%95-e1806f093e6d)
