---
publish: true
tags:
---
# 개요

Walrus Operator(:=)는 Python 3.8에서 도입된 대입식(assignment expression) 연산자다. 표현식 내에서 변수에 값을 할당하고 동시에 그 값을 사용할 수 있게 해준다.

# 기본 문법

```python
# 기본 형태
변수명 := 표현식

# 괄호로 감싸서 사용
(변수명 := 표현식)
```

# 주요 사용 사례

## 조건문에서의 활용

### 기존 방식
```python
data = get_data()
if data is not None:
    process_data(data)
```

### Walrus 사용
```python
if (data := get_data()) is not None:
    process_data(data)
```

## 반복문에서의 활용

### 파일 읽기
```python
# 기존 방식
while True:
    chunk = file.read(8192)
    if not chunk:
        break
    process(chunk)

# Walrus 사용
while (chunk := file.read(8192)):
    process(chunk)
```

### 리스트 처리
```python
# 기존 방식
numbers = [1, 2, 3, 4, 5]
total = 0
while numbers:
    number = numbers.pop()
    total += number

# Walrus 사용
total = 0
while (number := numbers.pop()):
    total += number
```

## 리스트 컴프리헨션에서의 활용

### 복잡한 계산 결과 재사용
```python
# 기존 방식
results = []
for x in range(10):
    result = complex_calculation(x)
    results.append((result, result ** 2, result ** 3))

# Walrus 사용
results = [(y := complex_calculation(x), y ** 2, y ** 3) 
           for x in range(10)]
```

# 고급 활용 예제

## API 응답 처리
```python
def handle_api_response():
    while (response := api.get_next_page()):
        if (errors := response.get('errors')):
            log_errors(errors)
            continue
        if (data := response.get('data')):
            process_data(data)
```

## 데이터 검증
```python
def validate_user_data(data):
    if not (name := data.get('name')):
        raise ValueError("Name is required")
    if not (age := data.get('age')):
        raise ValueError("Age is required")
    if not (18 <= age <= 100):
        raise ValueError(f"Invalid age: {age}")
    
    return {'name': name, 'age': age}
```

## 정규식 매칭
```python
import re

def extract_version(text):
    if (match := re.search(r'version\s+(\d+\.\d+)', text)):
        return match.group(1)
    return None
```

# 성능 비교

## 메모리 사용량 비교
```python
# 기존 방식
def process_large_data_old(data):
    result = expensive_calculation(data)
    return [result, result ** 2, result ** 3]

# Walrus 사용
def process_large_data_new(data):
    return [(calc := expensive_calculation(data)), 
            calc ** 2, 
            calc ** 3]
```

비교 결과:
- 기존 방식: 중간 결과를 저장하기 위한 추가 변수 필요
- Walrus 사용: 임시 변수 없이 직접 결과 생성

# 주의사항

## 1. 가독성 고려
```python
# 피해야 할 예
if (n := len(x := f())) < 10: print(x)

# 권장하는 방식
x = f()
if (n := len(x)) < 10:
    print(x)
```

## 2. 스코프 규칙
```python
# 전역 스코프에서는 사용 불가
(x := 1)  # SyntaxError

# 함수 내부에서는 사용 가능
def func():
    (x := 1)
    return x
```

## 3. 중첩 사용 주의
```python
# 혼란스러운 코드
[(x := i + 1, y := x * 2) for i in range(5)]

# 더 명확한 방식
[(i + 1, (i + 1) * 2) for i in range(5)]
```

# 실제 활용 팁

## 1. 디버깅에 활용
```python
def debug_function(data):
    if (result := process_data(data)) is not None:
        print(f"Processing result: {result}")
        return result
    return None
```

## 2. 로깅과 함께 사용
```python
import logging

def process_with_logging(items):
    if (n := len(items)) > 1000:
        logging.warning(f"Processing large dataset: {n} items")
    return process_items(items)
```

# 결론

Walrus Operator는 코드를 더 간결하고 효율적으로 만들 수 있는 강력한 도구다. 특히 조건문, 반복문, 컴프리헨션에서 활용할 때 그 진가를 발휘한다. 다만, 과도한 사용은 코드의 가독성을 해칠 수 있으므로 적절한 상황에서만 사용하는 것이 좋다.