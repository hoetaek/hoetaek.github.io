---
title: Pytest Fixture Scope
publish: false
tags:
---
Pytest fixture의 scope는 fixture가 재사용되는 범위를 정의한다. 적절한 scope 선택은 테스트 실행 속도와 리소스 사용을 최적화하는 데 중요하다.

## Scope의 종류

Pytest는 다음 5가지 scope를 제공한다:
- function (기본값)
- class
- module
- package
- session

## 1. Function Scope (Default)

매 테스트 함수마다 새로운 fixture가 생성된다.

### 사용해야 할 때
- 테스트 간 독립성이 중요할 때
- 상태를 공유하면 안 되는 경우
- 가벼운 리소스를 다룰 때

```python
@pytest.fixture
def user_data():
    return {"username": "testuser", "email": "test@example.com"}

def test_create_user(user_data):
    user = User.objects.create(**user_data)
    assert user.username == "testuser"

def test_modify_user_data(user_data):
    # user_data는 이전 테스트와 독립적인 새로운 객체
    user_data["email"] = "new@example.com"
    user = User.objects.create(**user_data)
    assert user.email == "new@example.com"
```

## 2. Class Scope

테스트 클래스 내에서 fixture가 재사용된다.

### 사용해야 할 때
- 연관된 테스트들이 같은 설정을 공유할 때
- 클래스 단위로 상태를 유지해야 할 때
- 설정 비용이 적당한 수준일 때

```python
@pytest.fixture(scope="class")
def db_test_data(request):
    data = create_test_database()
    request.cls.test_data = data
    yield data
    cleanup_test_database()

@pytest.mark.usefixtures("db_test_data")
class TestUserOperations:
    def test_user_search(self):
        # test_data는 클래스의 모든 테스트 메서드에서 공유
        results = search_users(self.test_data)
        assert len(results) > 0
    
    def test_user_filter(self):
        # 같은 test_data를 재사용
        filtered = filter_users(self.test_data)
        assert len(filtered) <= len(self.test_data)
```

reference
- https://docs.pytest.org/en/stable/how-to/fixtures.html#use-fixtures-in-classes-and-modules-with-usefixtures
## 3. Module Scope

모듈(파일) 내의 모든 테스트에서 fixture가 공유된다.

### 사용해야 할 때
- 모듈 단위로 무거운 설정이 필요할 때
- 데이터베이스 스키마 설정같은 비용이 큰 작업
- 모듈의 모든 테스트가 같은 기본 설정을 필요로 할 때

```python
@pytest.fixture(scope="module")
def database_schema():
    # 모듈 시작시 한 번만 실행
    print("Setting up database schema...")
    setup_schema()
    yield
    print("Tearing down database schema...")
    teardown_schema()

def test_first_operation(database_schema):
    # schema가 이미 설정된 상태
    assert perform_operation() == "success"

def test_second_operation(database_schema):
    # 같은 schema 재사용
    assert perform_another_operation() == "success"
```

## 4. Session Scope

전체 테스트 실행 동안 fixture가 한 번만 생성되고 재사용된다.

### 사용해야 할 때
- 매우 무거운 설정 작업이 필요할 때
- 외부 서비스 연결이 필요할 때
- 전체 테스트 suite가 공통 설정을 필요로 할 때

```python
@pytest.fixture(scope="session")
def api_client():
    # 테스트 세션 전체에서 한 번만 실행
    client = create_api_client()
    client.setup_authentication()
    return client

@pytest.fixture(scope="session")
def docker_container():
    # Docker 컨테이너 시작
    container = start_test_container()
    yield container
    # 모든 테스트 완료 후 정리
    container.stop()

def test_api_request(api_client):
    response = api_client.get("/users")
    assert response.status_code == 200

def test_another_request(api_client):
    # 같은 api_client 인스턴스 재사용
    response = api_client.post("/users")
    assert response.status_code == 201
```

## 실제 사용 예시

### 1. 데이터베이스 테스트 최적화
```python
@pytest.fixture(scope="session")
def django_db_setup():
    # 전체 테스트 세션에서 한 번만 DB 설정
    setup_test_database()
    yield
    teardown_test_database()

@pytest.fixture(scope="function")
def test_user():
    # 각 테스트마다 새로운 사용자 생성
    user = User.objects.create(username="testuser")
    yield user
    user.delete()

@pytest.fixture(scope="class")
def test_data_set():
    # 테스트 클래스당 한 번만 데이터 셋 생성
    data = create_large_dataset()
    yield data
    cleanup_dataset(data)
```

### 2. API 테스트 구성
```python
@pytest.fixture(scope="session")
def base_api_client():
    # 세션 전체에서 사용할 기본 클라이언트
    return APIClient()

@pytest.fixture(scope="function")
def authenticated_client(base_api_client):
    # 각 테스트마다 새로운 인증 토큰 사용
    token = generate_test_token()
    base_api_client.credentials(HTTP_AUTHORIZATION=f'Bearer {token}')
    return base_api_client
```

## 주의사항

1. Scope 의존성
- 더 넓은 scope의 fixture는 더 좁은 scope의 fixture에 의존할 수 없다
```python
# 잘못된 예
@pytest.fixture(scope="session")
def session_fixture(function_fixture):  # 오류 발생!
    pass

# 올바른 예
@pytest.fixture(scope="function")
def function_fixture(session_fixture):
    pass
```

2. 상태 공유
- 넓은 scope를 사용할 때는 상태 변경에 주의해야 한다
```python
@pytest.fixture(scope="module")
def shared_data():
    return {"count": 0}

def test_increment(shared_data):
    # 주의: 이 상태 변경은 다른 테스트에 영향을 줄 수 있다
    shared_data["count"] += 1
```

## 결론

적절한 fixture scope 선택은 테스트의 효율성과 신뢰성에 큰 영향을 미친다. 일반적으로:
- 독립성이 중요한 경우 function scope 사용
- 설정 비용이 큰 경우 class나 module scope 고려
- 매우 무거운 리소스는 session scope 사용
- 항상 가장 좁은 범위의 scope를 우선 고려하고, 필요한 경우에만 범위를 넓히는 것이 좋다