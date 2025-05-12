---
publish: false
tags:
---
Django 데이터베이스 인덱싱 최적화 가이드

# 개념 이해하기
데이터베이스 인덱스는 도서관의 색인과 같습니다. 책을 찾을 때 모든 책장을 둘러보지 않고 색인을 통해 빠르게 찾을 수 있듯이, 인덱스를 통해 데이터를 빠르게 검색할 수 있습니다.

```mermaid
graph TD
    A[데이터베이스 테이블] --> B[인덱스 없음]
    A --> C[인덱스 있음]
    B --> D[전체 테이블 스캔<br>O(n)]
    C --> E[인덱스 스캔<br>O(log n)]
```

# 기본 인덱스 설정

## 1. 단일 필드 인덱스
```python
class Article(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True, db_index=True)
    
    # 또는 Meta 클래스 사용
    class Meta:
        indexes = [
            models.Index(fields=['created_at']),
        ]
```

장점:
- 구현이 간단함
- 특정 필드 기준 검색 속도 향상
- 정렬 성능 개선

단점:
- 저장 공간 추가 사용
- 쓰기 작업 시 오버헤드

사용처:
- 자주 검색되는 단일 컬럼
- 정렬이 자주 필요한 필드
- 외래 키 필드

## 2. 복합 인덱스
```python
class Order(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    product = models.ForeignKey(Product, on_delete=models.CASCADE)
    ordered_at = models.DateTimeField(auto_now_add=True)

    class Meta:
        indexes = [
            models.Index(fields=['user', 'ordered_at']),
        ]
```

장점:
- 여러 필드 조합 검색 최적화
- 첫 번째 필드 기준 검색도 가능

단점:
- 더 많은 저장 공간 필요
- 필드 순서가 중요함
- 쓰기 작업 시 더 큰 오버헤드

사용처:
- 함께 검색되는 필드 조합
- 필터링과 정렬이 동시에 필요한 경우
- 보고서 생성 쿼리 최적화

## 3. 부분 인덱스
```python
class Product(models.Model):
    name = models.CharField(max_length=200)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    is_active = models.BooleanField(default=True)

    class Meta:
        indexes = [
            models.Index(
                fields=['name'],
                name='active_products_idx',
                condition=Q(is_active=True)
            ),
        ]
```

장점:
- 인덱스 크기 감소
- 특정 조건의 검색 최적화
- 불필요한 데이터 제외

단점:
- 조건이 맞지 않으면 사용되지 않음
- 구현이 복잡할 수 있음

사용처:
- 특정 상태의 레코드만 검색
- 소프트 삭제된 레코드 제외
- 활성 사용자 데이터 검색

## 4. 함수 기반 인덱스
```python
from django.db.models.functions import Upper

class Customer(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

    class Meta:
        indexes = [
            models.Index(Upper('name'), name='name_upper_idx'),
        ]
```

장점:
- 함수 결과 기반 검색 최적화
- 대소문자 구분 없는 검색 가능

단점:
- 데이터베이스 종속적
- 추가 저장 공간 필요
- 복잡한 함수는 성능 저하 가능

사용처:
- 대소문자 구분 없는 검색
- 날짜 형식 변환 검색
- 계산된 값 기반 검색

# 고급 인덱싱 기법

## 1. GiST 인덱스 (PostgreSQL)
```python
from django.contrib.postgres.indexes import GistIndex

class Location(models.Model):
    name = models.CharField(max_length=200)
    location = models.PointField()

    class Meta:
        indexes = [
            GistIndex(fields=['location'])
        ]
```

장점:
- 지리 데이터 검색 최적화
- 범위 검색 효율적

단점:
- PostgreSQL 전용
- 일반 검색보다 느릴 수 있음

사용처:
- 지리 정보 시스템
- 범위 검색이 필요한 경우

## 2. 해시 인덱스
```python
from django.contrib.postgres.indexes import HashIndex

class User(models.Model):
    email = models.EmailField()

    class Meta:
        indexes = [
            HashIndex(fields=['email'])
        ]
```

장점:
- 정확한 일치 검색 최적화
- 작은 저장 공간

단점:
- 범위 검색 불가
- 일부 데이터베이스만 지원

사용처:
- 정확한 값 검색
- 해시 값 기반 검색

# 성능 모니터링

## 1. 인덱스 사용 확인
```python
from django.db import connection

def check_query_performance():
    with connection.cursor() as cursor:
        cursor.execute("""
            EXPLAIN ANALYZE
            SELECT * FROM myapp_model
            WHERE field = 'value'
        """)
        return cursor.fetchall()
```

## 2. 쿼리 최적화
```python
# 인덱스 활용 쿼리
Article.objects.filter(created_at__gte='2024-01-01').select_related('author')

# 인덱스 힌트 사용 (MySQL)
from django.db.models import Index
Article.objects.filter(status='published').using(Index('status_idx'))
```

# 주의사항

1. 인덱스 과다 사용
```python
# 피해야 할 예
class BadExample(models.Model):
    # 모든 필드에 인덱스 설정 - 안 좋은 예시
    field1 = models.CharField(max_length=100, db_index=True)
    field2 = models.CharField(max_length=100, db_index=True)
    field3 = models.CharField(max_length=100, db_index=True)
    # ... 더 많은 인덱스
```

2. 업데이트가 잦은 필드
```python
# 주의해야 할 경우
class Counter(models.Model):
    count = models.IntegerField(db_index=True)  # 빈번한 업데이트 시 성능 저하
```

# 결론

효율적인 인덱싱을 위한 체크리스트:
1. 실제 쿼리 패턴 분석
2. 적절한 인덱스 유형 선택
3. 성능 영향 모니터링
4. 주기적인 인덱스 재구성

각 상황에 맞는 인덱싱 전략:
- 읽기 위주: 적극적인 인덱싱
- 쓰기 위주: 최소한의 필수 인덱스
- 복합 작업: 균형잡힌 인덱스 설계