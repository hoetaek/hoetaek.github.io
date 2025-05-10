---
date: 2025-05-10
publish: true
tags:
---
## 개요

parametrize는 pytest에서 테스트 코드의 중복을 줄이고 다양한 입력값에 대한 테스트를 간결하게 작성할 수 있도록 도와주는 유용한 기능입니다. 하나의 테스트 함수로 여러 입력 조합에 대해 독립적인 테스트를 실행할 수 있어, 비슷한 패턴의 테스트를 반복 작성하는 대신 간결하고 명확한 코드로 다양한 시나리오를 커버할 수 있습니다.

## 주요 장점

- **코드 중복 감소**: 여러 입력값을 하나의 테스트 함수로 처리
- **가독성 향상**: 다양한 입력과 기대 결과를 명확하게 표현
- **유지보수 용이성**: 테스트 케이스 추가 및 수정이 간편
- **테스트 커버리지 확대**: 다양한 입력 조합 테스트 가능

## 기본 사용법

```python
@pytest.mark.parametrize("인자명", [값1, 값2, 값3])
def test_함수명(인자명):
    # 테스트 로직
    assert ...
```

## 주요 기능

### 1. 여러 인자 parametrize

```python
@pytest.mark.parametrize("a, b, expected", [
    (1, 2, 3),
    (5, 5, 10),
    (-1, 1, 0),
    (0, 0, 0),
    (100, 200, 300)
])
def test_add(a, b, expected):
    assert add(a, b) == expected
```

**실행 결과:**

```
test_math.py::test_add[1-2-3] PASSED
test_math.py::test_add[5-5-10] PASSED
test_math.py::test_add[-1-1-0] PASSED
test_math.py::test_add[0-0-0] PASSED
test_math.py::test_add[100-200-300] PASSED
```

**보충 설명:** 여러 인자를 한 번에 parametrize 할 수 있으며, 각 튜플은 하나의 테스트 케이스가 됩니다. 위 예시에서는 5개의 인자 조합에 대해 test_add 함수가 5번 독립적으로 실행됩니다. pytest는 자동으로 각 케이스에 대한 테스트 ID를 생성합니다(예: `[1-2-3]`).

### 2. 테스트 식별성 높이기 (ids 사용)

```python
@pytest.mark.parametrize("a, b, expected", [
    (1, 2, 3),
    (5, 5, 10),
    (-1, 1, 0),
    (0, 0, 0)
], ids=["simple_add", "double_five", "add_neg_pos", "add_zeros"])
def test_add_with_custom_ids(a, b, expected):
    assert add(a, b) == expected
```

**실행 결과:**

```
test_math_with_ids.py::test_add_with_custom_ids[simple_add] PASSED
test_math_with_ids.py::test_add_with_custom_ids[double_five] PASSED
test_math_with_ids.py::test_add_with_custom_ids[add_neg_pos] PASSED
test_math_with_ids.py::test_add_with_custom_ids[add_zeros] PASSED
```

**보충 설명:** 테스트 케이스가 많아질수록 자동 생성된 ID(`[1-2-3]`)만으로는 어떤 테스트인지 파악하기 어려울 수 있습니다. `ids` 파라미터를 사용하면 각 테스트 케이스에 의미 있는 이름을 부여할 수 있어 테스트 결과를 해석하고 디버깅하기 쉬워집니다. 특히 테스트가 실패했을 때 어떤 케이스에서 문제가 발생했는지 빠르게 확인할 수 있습니다.

### 3. pytest.param 활용

```python
@pytest.mark.parametrize("a, b, expected", [
    pytest.param(1, 2, 3, id="positive_numbers_check"),
    pytest.param(5, 5, 10, id="equal_numbers_check"),
    pytest.param(-1, 1, 0, id="positive_and_negative_check"),
    pytest.param(0, 0, 0, id="zeros_check"),
])
def test_add_with_pytest_param_ids(a, b, expected):
    assert add(a, b) == expected
```

**실행 결과:**

```
test_math_with_ids.py::test_add_with_pytest_param_ids[positive_numbers_check] PASSED
test_math_with_ids.py::test_add_with_pytest_param_ids[equal_numbers_check] PASSED
test_math_with_ids.py::test_add_with_pytest_param_ids[positive_and_negative_check] PASSED
test_math_with_ids.py::test_add_with_pytest_param_ids[zeros_check] PASSED
```

**보충 설명:** `pytest.param`을 사용하면 각 테스트 케이스마다 ID뿐만 아니라 다른 pytest 마크(marks)도 개별적으로 적용할 수 있습니다. 이는 단순히 ID를 부여하는 것보다 더 많은 제어가 필요할 때 유용합니다. 예를 들어, 특정 케이스만 건너뛰거나(skip) 실패가 예상되는 케이스(xfail)를 표시할 수 있습니다.

### 4. 여러 parametrize 데코레이터 조합

```python
@pytest.mark.parametrize("x", [0, 1])
@pytest.mark.parametrize("y", [2, 3])
def test_foo(x, y):
    print(f"x: {x}, y: {y}")
    assert True
```

**실행 결과:**

```
test_multiple_parametrize.py::test_foo[2-0] PASSED
test_multiple_parametrize.py::test_foo[3-0] PASSED
test_multiple_parametrize.py::test_foo[2-1] PASSED
test_multiple_parametrize.py::test_foo[3-1] PASSED
```

**보충 설명:** 하나의 테스트 함수에 여러 `parametrize` 데코레이터를 적용하면, pytest는 모든 가능한 조합(데카르트 곱)을 생성하여 테스트를 실행합니다. 위 예시에서는 x의 2가지 값과 y의 2가지 값의 조합으로 총 4번의 테스트가 실행됩니다. 이 방식은 두 개 이상의 독립적인 변수들의 모든 조합을 테스트해야 할 때 매우 유용합니다. 데코레이터 순서에 따라 테스트 ID의 파라미터 순서가 결정되므로 주의해야 합니다.

### 5. fixture parametrize (indirect=True)

```python
@pytest.fixture
def my_fixture(request):
    # request.param은 parametrize에서 전달된 값입니다
    return request.param * 10

@pytest.mark.parametrize("my_fixture", [1, 2, 3], indirect=True)
def test_indirect_fixture(my_fixture):
    assert my_fixture % 10 == 0
    if my_fixture == 10:
        assert True
    elif my_fixture == 20:
        assert True
    elif my_fixture == 30:
        assert True
```

**실행 결과:**

```
test_indirect_parametrize.py::test_indirect_fixture[1] PASSED
test_indirect_parametrize.py::test_indirect_fixture[2] PASSED
test_indirect_parametrize.py::test_indirect_fixture[3] PASSED
```

**보충 설명:** `indirect=True` 옵션을 사용하면 parametrize된 값을 직접 테스트 함수의 인자로 전달하는 대신, 같은 이름의 fixture에 전달할 수 있습니다. 이 경우 fixture 함수 내에서 `request.param`을 통해 parametrize된 값에 접근할 수 있습니다.

이 방식은 다음과 같은 경우에 유용합니다:

- 테스트 실행 전에 복잡한 설정이 필요한 경우
- parametrize된 값을 가공해서 사용해야 하는 경우
- 동일한 테스트에 대해 서로 다른 환경 설정이 필요한 경우

또한 `indirect=['param1', 'param2']` 형태로 사용하여 여러 파라미터 중 일부만 간접 처리할 수도 있습니다.

### 6. 다른 마크와 함께 사용

```python
@pytest.mark.parametrize("value, expected_type", [
    (10, int),
    ("hello", str),
    pytest.param(5.5, float, marks=pytest.mark.skip(reason="부동 소수점 테스트는 현재 건너뜀")),
    pytest.param([1, 2], list, marks=pytest.mark.xfail(reason="리스트 타입은 실패 예상"))
])
def test_types(value, expected_type):
    assert isinstance(value, expected_type)
```

**실행 결과:**

```
test_parametrize_with_marks.py::test_types[10-<class 'int'>] PASSED
test_parametrize_with_marks.py::test_types[hello-<class 'str'>] PASSED
test_parametrize_with_marks.py::test_types[5.5-<class 'float'>] SKIPPED (부동 소수점 테스트는 현재 건너뜀)
test_parametrize_with_marks.py::test_types[[1, 2]-<class 'list'>] xfail (리스트 타입은 실패 예상)
```

**보충 설명:** `pytest.param`과 함께 다양한 pytest 마크를 사용하면 특정 테스트 케이스에 대해 세밀한 제어가 가능합니다:

- `pytest.mark.skip`: 특정 케이스를 건너뛰도록 지정
- `pytest.mark.xfail`: 실패가 예상되는 케이스를 표시 (테스트는 실행되지만 실패해도 전체 테스트 결과에 영향을 주지 않음)
- `pytest.mark.parametrize`: 이 예제에서처럼 다른 마크와 함께 사용 가능

이를 통해 개발 중인 기능이나 아직 구현되지 않은 부분에 대한 테스트를 미리 작성하고, 적절히 표시할 수 있습니다. 또한 특정 환경이나 조건에서만 실행해야 하는 테스트를 관리하기에도 좋습니다.

## 추가 팁

### 파라미터 값 생성을 위한 함수 사용

```python
def generate_test_data():
    return [(i, i*2) for i in range(5)]

@pytest.mark.parametrize("input, expected", generate_test_data())
def test_double(input, expected):
    assert input * 2 == expected
```

### 매개변수화된 클래스

```python
@pytest.mark.parametrize("value", [1, 2, 3])
class TestClass:
    def test_one(self, value):
        assert value >= 1
        
    def test_two(self, value):
        assert value <= 3
```

## 결론

pytest.mark.parametrize는 테스트 코드의 효율성과 가독성을 크게 향상시키며, 다양한 입력값에 대한 테스트를 간결하게 작성할 수 있게 해주는 강력한 도구입니다. 중복 코드를 줄이고, 테스트 케이스 관리가 용이하며, 다양한 시나리오에 대한 테스트 커버리지를 확대할 수 있습니다. 특히 여러 입력 조합에 대해 동일한 로직을 반복적으로 테스트해야 하는 경우 사용하면 매우 효과적입니다.