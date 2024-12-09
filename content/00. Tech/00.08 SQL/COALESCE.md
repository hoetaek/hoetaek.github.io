---
date: 2024-12-06
publish: false
tags:
---
```table-of-contents
title: # 목차
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 0 # Include headings from the specified level
maxLevel: 5 # Include headings up to the specified level
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```
# 개념 이해하기

COALESCE 함수는 여러 값 중에서 NULL이 아닌 첫 번째 값을 반환하는 SQL의 특별한 함수이다. 마치 여러 개의 백업 계획을 가지고 있다가 실행 가능한 첫 번째 계획을 선택하는 것과 같다. 실생활에서는 친구와 약속을 잡을 때, "토요일에 만나고, 안되면 일요일, 그것도 안되면 다음 주 수요일"과 같이 대안을 순서대로 검토하는 것과 비슷하다.

# 기본 문법과 작동 방식

```sql
COALESCE(값1, 값2, 값3, ..., 값N)
```

COALESCE는 다음과 같은 규칙으로 동작한다:
1. 왼쪽부터 오른쪽으로 값을 검사한다
2. NULL이 아닌 첫 번째 값을 만나면 즉시 그 값을 반환한다
3. 모든 값이 NULL이면 NULL을 반환한다

# 실제 데이터로 보는 COALESCE

## 테이블 구조와 데이터

```sql
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    middle_name VARCHAR(50),
    last_name VARCHAR(50),
    primary_phone VARCHAR(15),
    secondary_phone VARCHAR(15),
    emergency_phone VARCHAR(15),
    salary DECIMAL(10,2),
    bonus DECIMAL(10,2)
);
```

### 샘플 데이터
| employee_id | first_name | middle_name | last_name | primary_phone | secondary_phone | emergency_phone | salary  | bonus   |
|------------|------------|-------------|-----------|---------------|-----------------|-----------------|---------|---------|
| 1          | 김         | NULL        | 철수      | 01012345678   | NULL           | 01098765432     | 3000.00 | NULL    |
| 2          | 이         | 미          | 영희      | NULL          | 01023456789    | NULL            | 3500.00 | 500.00  |
| 3          | 박         | NULL        | 민수      | NULL          | NULL           | NULL            | 4000.00 | NULL    |

# COALESCE의 실제 활용 사례

## 1. 연락처 정보 처리

```sql
-- 사용 가능한 첫 번째 연락처 찾기
SELECT 
    employee_id,
    first_name,
    last_name,
    COALESCE(primary_phone, secondary_phone, emergency_phone, '연락처 없음') as contact_number
FROM 
    employees;
```

실행 결과:
| employee_id | first_name | last_name | contact_number |
|------------|------------|-----------|----------------|
| 1          | 김         | 철수      | 01012345678    |
| 2          | 이         | 영희      | 01023456789    |
| 3          | 박         | 민수      | 연락처 없음    |

## 2. 급여 계산

```sql
-- 총 수입 계산 (보너스가 NULL이면 0으로 처리)
SELECT 
    employee_id,
    first_name,
    salary,
    COALESCE(bonus, 0) as actual_bonus,
    salary + COALESCE(bonus, 0) as total_compensation
FROM 
    employees;
```

실행 결과:
| employee_id | first_name | salary  | actual_bonus | total_compensation |
|------------|------------|---------|--------------|-------------------|
| 1          | 김         | 3000.00 | 0.00         | 3000.00          |
| 2          | 이         | 3500.00 | 500.00       | 4000.00          |
| 3          | 박         | 4000.00 | 0.00         | 4000.00          |

## 3. 이름 형식 지정

```sql
-- 완전한 이름 조합 (middle_name이 있는 경우에만 포함)
SELECT 
    employee_id,
    CONCAT(
        first_name,
        COALESCE(CONCAT(' ', middle_name), ''),
        ' ',
        last_name
    ) as full_name
FROM 
    employees;
```

실행 결과:
| employee_id | full_name    |
|------------|--------------|
| 1          | 김 철수      |
| 2          | 이 미 영희   |
| 3          | 박 민수      |

# 고급 활용 기법

## 1. 다중 조건 처리

```sql
-- 연락처 우선순위와 함께 상태 표시
SELECT 
    employee_id,
    first_name,
    CASE 
        WHEN primary_phone IS NOT NULL THEN '기본 연락처'
        WHEN secondary_phone IS NOT NULL THEN '보조 연락처'
        WHEN emergency_phone IS NOT NULL THEN '비상 연락처'
        ELSE '연락처 없음'
    END as contact_type,
    COALESCE(primary_phone, secondary_phone, emergency_phone, '미등록') as contact_number
FROM 
    employees;
```

## 2. 집계 함수와 함께 사용

```sql
-- 부서별 평균 총 수입 계산
SELECT 
    department_id,
    AVG(salary + COALESCE(bonus, 0)) as avg_total_compensation
FROM 
    employees
GROUP BY 
    department_id;
```

# 주의사항

1. 데이터 타입 일관성
   - COALESCE의 모든 인자는 호환되는 데이터 타입이어야 한다
   - 자동 형변환이 발생할 수 있으므로 주의가 필요하다

2. 성능 고려사항
   - 너무 많은 인자를 사용하면 성능에 영향을 줄 수 있다
   - 인덱스 사용에 영향을 줄 수 있으므로 WHERE 절에서 사용 시 주의가 필요하다

3. NULL 값의 의미
   - NULL의 업무적 의미를 정확히 파악해야 한다
   - 단순히 NULL을 다른 값으로 대체하는 것이 항상 올바른 것은 아니다

# 결론

COALESCE는 다음과 같은 상황에서 특히 유용하다:

1. NULL 값 처리가 필요한 경우
2. 우선순위가 있는 대체 값을 처리할 때
3. 리포트나 화면 출력 시 기본값 처리가 필요한 경우
4. 복잡한 계산식에서 NULL 처리가 필요한 경우

COALESCE를 효과적으로 활용하면 코드를 더 안정적이고 읽기 쉽게 만들 수 있으며, NULL 값 처리로 인한 오류를 예방할 수 있다.