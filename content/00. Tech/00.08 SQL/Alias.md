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

SQL Alias는 테이블이나 컬럼에 임시로 다른 이름을 부여하는 기능이다. 마치 사람에게 별명을 붙이는 것처럼, 데이터베이스의 객체에 별칭을 지정하여 더 쉽게 참조할 수 있게 한다. 실생활의 예로는 긴 회사명 대신 약자를 사용하는 것과 비슷하다.

# 필요한 배경 지식

Alias를 제대로 이해하고 활용하기 위해서는 다음 개념들을 알고 있어야 한다:

1. SQL의 기본 문법 구조
2. SELECT 문의 구성 요소
3. 테이블 간의 관계와 JOIN

# Alias의 종류와 기본 문법

## 테이블 Alias
테이블 이름에 별칭을 부여하는 방식이다. 특히 JOIN 구문에서 유용하다.

```sql
-- 기본 문법 (AS 키워드 사용)
SELECT * FROM employees AS e;

-- AS 키워드 생략 가능
SELECT * FROM employees e;
```

## 컬럼 Alias
컬럼에 별칭을 지정하는 방식이다. 결과셋의 컬럼 이름을 보기 좋게 변경할 수 있다.

```sql
-- 기본 문법 (AS 키워드 사용)
SELECT 
    first_name AS 이름,
    salary AS 급여;

-- AS 키워드 생략 가능
SELECT 
    first_name 이름,
    salary 급여;

-- 공백이나 특수문자가 필요한 경우 큰따옴표 사용
SELECT 
    first_name AS "직원 이름",
    salary AS "월급(원)";
```

# 실제 데이터로 보는 Alias 활용

## 테이블 구조

```sql
-- 직원 테이블
CREATE TABLE employees (
    employee_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    department_id INT
);

-- 부서 테이블
CREATE TABLE departments (
    department_id INT PRIMARY KEY,
    department_name VARCHAR(100),
    location VARCHAR(100)
);
```

### employees 테이블 데이터
| employee_id | first_name | last_name | department_id |
|------------|------------|-----------|---------------|
| 1          | 김         | 철수      | 10            |
| 2          | 이         | 영희      | 20            |
| 3          | 박         | 민수      | 10            |

### departments 테이블 데이터
| department_id | department_name | location |
|--------------|-----------------|----------|
| 10           | 개발팀          | 서울     |
| 20           | 영업팀          | 부산     |

# Alias 활용 사례

## 1. 단순 컬럼 별칭

```sql
SELECT 
    e.first_name AS 이름,
    e.last_name AS 성,
    e.department_id AS 부서번호
FROM 
    employees e
WHERE 
    e.department_id = 10;
```

### 실행 결과
| 이름 | 성   | 부서번호 |
|------|------|----------|
| 김   | 철수 | 10       |
| 박   | 민수 | 10       |

## 2. JOIN에서의 Alias 활용

```sql
SELECT 
    e.first_name AS 이름,
    e.last_name AS 성,
    d.department_name AS 부서명,
    d.location AS 근무지
FROM 
    employees e
    INNER JOIN departments d ON e.department_id = d.department_id
ORDER BY 
    e.employee_id;
```

### 실행 결과
| 이름 | 성   | 부서명 | 근무지 |
|------|------|--------|--------|
| 김   | 철수 | 개발팀 | 서울   |
| 이   | 영희 | 영업팀 | 부산   |
| 박   | 민수 | 개발팀 | 서울   |

## 3. 복잡한 표현식에서의 Alias

```sql
SELECT 
    e.first_name || ' ' || e.last_name AS 전체이름,
    CASE 
        WHEN d.location = '서울' THEN '본사'
        ELSE '지사'
    END AS 근무지구분,
    d.department_name AS 부서명
FROM 
    employees e
    LEFT JOIN departments d ON e.department_id = d.department_id;
```

# 고급 활용법

## 1. 서브쿼리에서의 Alias 활용

```sql
SELECT 
    d.department_name AS 부서명,
    emp_count.인원수
FROM 
    departments d
    LEFT JOIN (
        SELECT 
            department_id,
            COUNT(*) AS 인원수
        FROM 
            employees
        GROUP BY 
            department_id
    ) emp_count ON d.department_id = emp_count.department_id;
```

## 2. 중첩 JOIN에서의 Alias

```sql
SELECT 
    e.first_name AS 직원이름,
    m.first_name AS 관리자이름,
    d.department_name AS 부서명
FROM 
    employees e
    LEFT JOIN employees m ON e.manager_id = m.employee_id
    LEFT JOIN departments d ON e.department_id = d.department_id;
```

# 주의사항

1. 명명 규칙
   - 의미 있는 별칭을 사용한다
   - 너무 짧거나 모호한 별칭은 피한다
   - 일관된 명명 규칙을 따른다

2. 가독성
   - 적절한 들여쓰기를 사용한다
   - 복잡한 쿼리에서는 주석을 추가한다
   - 일관된 대소문자 사용 규칙을 따른다

3. 기술적 제약
   - 특수문자나 공백이 포함된 별칭은 큰따옴표로 묶어야 한다
   - 데이터베이스의 예약어는 별칭으로 사용하지 않는다
   - 별칭의 길이 제한을 고려한다

# 모범 사례와 안티 패턴

## 좋은 예시
```sql
-- 의미 있는 별칭 사용
SELECT 
    emp.first_name AS 이름,
    dept.department_name AS 부서명
FROM 
    employees emp
    JOIN departments dept ON emp.department_id = dept.department_id;
```

## 나쁜 예시
```sql
-- 모호하고 의미 없는 별칭 사용
SELECT 
    a.first_name AS fn,
    b.department_name AS dn
FROM 
    employees a
    JOIN departments b ON a.department_id = b.department_id;
```

# 결론

SQL Alias는 단순한 기능이지만 적절히 활용하면 다음과 같은 이점이 있다:

1. 코드의 가독성이 향상된다
2. 유지보수가 용이해진다
3. 복잡한 쿼리를 더 쉽게 작성할 수 있다
4. SQL 문의 길이를 줄일 수 있다

Alias를 사용할 때는 항상 명확성과 일관성을 유지하는 것이 중요하며, 팀 내에서 합의된 명명 규칙을 따르는 것이 바람직하다.