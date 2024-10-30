---
title: Django Fixture 활용 가이드
publish: false
tags:
---
Fixture는 테스트에서 공통으로 사용되는 데이터나 객체를 재사용 가능한 형태로 제공하는 기능이다. pytest와 함께 사용할 때 특히 유용하다.

## conftest.py 설정

conftest.py는 여러 테스트 파일에서 공유되는 fixture를 정의하는 특별한 파일이다. 이 파일은 다음과 같은 특징을 가진다:
- 파일이 위치한 디렉토리와 하위 디렉토리의 모든 테스트가 접근 가능하다
- 별도의 import 없이 사용할 수 있다
- 프로젝트의 여러 위치에 둘 수 있다

### 기본 conftest.py 예시
```python
# conftest.py
import pytest
from rest_framework.test import APIClient
from django.contrib.auth import get_user_model

User = get_user_model()

@pytest.fixture
def api_client():
    return APIClient()

@pytest.fixture
def user_factory():
    def create_user(username="testuser", password="testpass123"):
        return User.objects.create_user(
            username=username,
            password=password
        )
    return create_user

@pytest.fixture
def authenticated_client(api_client, user_factory):
    user = user_factory()
    api_client.force_authenticate(user=user)
    return api_client
```

## Fixture 스코프

### 스코프 종류
```python
# 함수 레벨 (기본값)
@pytest.fixture(scope="function")
def temp_user():
    user = User.objects.create(username="temp")
    yield user
    user.delete()

# 클래스 레벨
@pytest.fixture(scope="class")
def class_data():
    return {"key": "value"}

# 모듈 레벨
@pytest.fixture(scope="module")
def module_data():
    return {"module": "test"}

# 세션 레벨
@pytest.fixture(scope="session")
def db_setup():
    return setup_test_database()
```
[[Pytest Fixture Scope]]
## Fixture 사용 예시

### 기본 사용법
```python
def test_user_detail(authenticated_client, user_factory):
    # fixture 조합 사용
    user = user_factory(username="testuser")
    response = authenticated_client.get(f"/api/users/{user.id}/")
    assert response.status_code == 200
```

### 팩토리 패턴 활용
```python
@pytest.fixture
def article_factory():
    def create_article(title="Test Article", author=None):
        if author is None:
            author = User.objects.create_user(username="author")
        return Article.objects.create(
            title=title,
            author=author,
            content="Test content"
        )
    return create_article

def test_article_creation(article_factory, user_factory):
    author = user_factory(username="specific_author")
    article = article_factory(
        title="Custom Title",
        author=author
    )
    assert article.author.username == "specific_author"
```

## 고급 Fixture 테크닉

### 1. 파라미터화된 Fixture
```python
@pytest.fixture(params=["admin", "user", "guest"])
def user_role(request):
    return request.param

def test_user_permissions(user_role, user_factory):
    user = user_factory(role=user_role)
    assert user.has_permission() == (user_role == "admin")
```

### 2. Fixture 조합
```python
@pytest.fixture
def user_with_articles(user_factory, article_factory):
    user = user_factory()
    articles = [
        article_factory(author=user)
        for _ in range(3)
    ]
    return {"user": user, "articles": articles}

def test_user_articles(user_with_articles):
    assert len(user_with_articles["articles"]) == 3
```

## 주의사항

1. Fixture 의존성을 명확히 하기
2. 스코프 설정 시 메모리 사용량 고려하기
3. 데이터베이스 Fixture는 가능한 작게 유지하기
4. cleanup 코드 필요 시 yield 사용하기

## 모범 사례

### 1. 명확한 이름 지정
```python
# 좋은 예
@pytest.fixture
def authenticated_admin_client():
    pass

# 피해야 할 예
@pytest.fixture
def client1():
    pass
```

### 2. 문서화
```python
@pytest.fixture
def mock_payment_gateway():
    """
    결제 게이트웨이를 모킹하는 fixture.
    
    Returns:
        MagicMock: 결제 처리를 시뮬레이션하는 mock 객체
    """
    pass
```

## 결론

Fixture는 테스트 코드의 재사용성과 유지보수성을 크게 향상시킨다. conftest.py를 통한 전역 Fixture 관리와 적절한 스코프 설정으로 효율적인 테스트 환경을 구축할 수 있다. 특히 Django 테스트에서는 데이터베이스 객체의 생성과 인증된 클라이언트 설정 등에 매우 유용하게 활용된다.