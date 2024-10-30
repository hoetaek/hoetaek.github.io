---
title: Django 병렬 테스트 실행 가이드
publish: false
tags:
---
대규모 Django 프로젝트에서 테스트 실행 시간을 단축하기 위해 pytest-xdist를 사용한 병렬 테스트 실행 방법을 알아본다.

## 설정

### 1. pytest-xdist 설치
```sh
# pip 사용 시
pip install pytest-xdist

# poetry 사용 시
poetry add pytest-xdist --dev
```

## 실행 방법

### 1. 기본 실행
```sh
# 자동으로 CPU 코어 수에 맞춰 실행
pytest -n auto

# 특정 프로세스 수 지정
pytest -n 3

# 특정 테스트만 병렬 실행
pytest -n auto tests/test_users.py
```

### 2. 상세 옵션 사용
```sh
# 실행 로그 상세 출력
pytest -n 3 -v

# 실패한 테스트만 재실행
pytest -n 3 --lf

# 특정 마커만 실행
pytest -n auto -m "slow"
```

## 주의사항

1. 테스트 간 독립성 확보가 중요하다
2. 데이터베이스 충돌에 주의해야 한다
3. 공유 리소스 사용 시 주의가 필요하다
4. 테스트 순서에 의존하면 안 된다

## 모범 사례

### 1. 데이터베이스 격리
```python
@pytest.mark.django_db(transaction=True)
def test_독립적인_데이터베이스_테스트():
    # 각 테스트는 독립적인 트랜잭션에서 실행
    pass
```

### 2. 공유 리소스 처리
```python
@pytest.fixture(scope="function")
def temp_directory():
    # 각 테스트마다 새로운 임시 디렉토리 사용
    with tempfile.TemporaryDirectory() as tmpdirname:
        yield tmpdirname
```

## 성능 최적화

### 1. 느린 테스트 식별
```sh
pytest -n auto --durations=10
```

### 2. 테스트 그룹화
```python
# 빠른 테스트
@pytest.mark.fast
def test_빠른_기능():
    pass

# 느린 테스트
@pytest.mark.slow
def test_느린_기능():
    pass

# 실행: pytest -n auto -m "not slow"
```

## 결론

pytest-xdist를 사용한 병렬 테스트 실행은 테스트 실행 시간을 크게 단축할 수 있다. 단, 테스트 간 독립성을 보장하고 공