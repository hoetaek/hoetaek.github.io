---
publish: true
tags:
---

Django 애플리케이션의 성능을 최적화하기 위해서는 SQL 쿼리를 모니터링하고 분석하는 것이 중요하다. 특히 테스트 환경에서 쿼리 수와 실행 시간을 확인하는 방법을 알아본다.

## 쿼리 로깅 설정

### settings.py 설정
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

## 쿼리 캡처하기

### CaptureQueriesContext 사용
```python
from django.test.utils import CaptureQueriesContext
from django.db import connection

def test_user_list_queries(client):
    with CaptureQueriesContext(connection) as ctx:
        response = client.get("/api/users/")
        
        # 실행된 모든 쿼리 출력
        for query in ctx.captured_queries:
            print(f"SQL: {query['sql']}")
            print(f"시간: {query['time']}")
            print(f"호출 위치: {query['stacktrace']}")
```

## 쿼리 수 제한하기

### django_assert_max_num_queries 사용
```python
from pytest_django.asserts import django_assert_max_num_queries

def test_최적화된_리스트_조회(api_client):
    with django_assert_max_num_queries(3):
        response = api_client.get("/api/users")
        assert response.status_code == 200
        assert len(response.json()) == 10
```

### 복잡한 시나리오 테스트
```python
def test_복잡한_데이터_조회(api_client):
    with django_assert_max_num_queries(5) as ctx:
        response = api_client.get("/api/complex-data")
        queries = ctx.captured_queries
        
        # 특정 유형의 쿼리 개수 확인
        select_queries = len([q for q in queries if q['sql'].startswith('SELECT')])
        assert select_queries <= 3, "SELECT 쿼리가 너무 많습니다"
```

## 쿼리 최적화 테크닉

### 1. select_related 사용 확인
```python
def test_최적화된_관계_조회():
    with CaptureQueriesContext(connection) as ctx:
        # 최적화 전
        users = User.objects.all()
        for user in users:
            print(user.profile.email)
            
    query_count_before = len(ctx.captured_queries)
    
    with CaptureQueriesContext(connection) as ctx:
        # 최적화 후
        users = User.objects.select_related('profile').all()
        for user in users:
            print(user.profile.email)
            
    query_count_after = len(ctx.captured_queries)
    
    assert query_count_after < query_count_before
```

### 2. prefetch_related 검증
```python
def test_최적화된_역참조_조회():
    with django_assert_max_num_queries(2):
        users = User.objects.prefetch_related('posts').all()
        for user in users:
            assert len(user.posts.all()) >= 0
```

## 성능 분석

### 쿼리 실행 시간 분석
```python
def test_쿼리_실행시간():
    with CaptureQueriesContext(connection) as ctx:
        response = api_client.get("/api/expensive-operation")
        
        slow_queries = [
            query for query in ctx.captured_queries
            if float(query['time']) > 0.1  # 100ms 이상 소요된 쿼리
        ]
        
        assert len(slow_queries) == 0, f"""
        느린 쿼리가 {len(slow_queries)}개 발견되었습니다:
        {slow_queries}
        """
```

## 주의사항

1. DEBUG 모드가 켜져 있어야 쿼리 로깅이 작동한다
2. 운영 환경에서는 쿼리 로깅을 비활성화해야 한다
3. 대량의 쿼리 로깅은 성능에 영향을 줄 수 있다
4. 테스트 환경과 실제 환경의 쿼리 수가 다를 수 있다

## 결론

SQL 쿼리 디버깅은 Django 애플리케이션의 성능 최적화에 필수적이다. CaptureQueriesContext와 django_assert_max_num_queries를 활용하여 쿼리의 수와 실행 시간을 모니터링하고, 이를 바탕으로 성능 개선을 진행할 수 있다.