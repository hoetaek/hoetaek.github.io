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
SQL 집계 함수 완벽 가이드: 데이터 분석의 핵심 도구

# 개념 이해하기

SQL 집계 함수는 여러 행의 데이터를 하나의 결과값으로 종합하는 특별한 함수들이다. 마치 학급의 시험 결과를 정리할 때 전체 평균, 최고점, 최저점 등을 계산하는 것과 같다. 이러한 집계 함수들은 데이터 분석과 리포팅에서 핵심적인 역할을 한다.

# 기본 데이터 구조 이해

실제 예제를 통해 집계 함수를 이해해보자. 다음과 같은 학생 성적 데이터를 사용할 것이다:

```sql
CREATE TABLE student_scores (
    student_id INT PRIMARY KEY,
    student_name VARCHAR(50),
    subject VARCHAR(50),
    score INT,
    class_id INT,
    exam_date DATE
);
```

### 샘플 데이터
| student_id | student_name | subject | score | class_id | exam_date   |
|------------|--------------|---------|--------|-----------|-------------|
| 1          | 김철수       | 수학     | 85     | 101       | 2024-03-01 |
| 2          | 이영희       | 수학     | 92     | 101       | 2024-03-01 |
| 3          | 박민수       | 수학     | 78     | 102       | 2024-03-01 |
| 4          | 정미경       | 수학     | 95     | 102       | 2024-03-01 |
| 5          | 김철수       | 영어     | 88     | 101       | 2024-03-02 |
| 6          | 이영희       | 영어     | 85     | 101       | 2024-03-02 |
| 7          | 박민수       | 영어     | 92     | 102       | 2024-03-02 |
| 8          | 정미경       | 영어     | 89     | 102       | 2024-03-02 |

# 주요 집계 함수 설명

## COUNT(): 행의 개수 세기

COUNT 함수는 세 가지 형태로 사용할 수 있다:

```sql
-- 전체 행 수 계산
SELECT COUNT(*) as total_records
FROM student_scores;

-- NULL이 아닌 특정 컬럼의 값 개수
SELECT COUNT(score) as score_count
FROM student_scores;

-- 중복을 제외한 고유값 개수
SELECT COUNT(DISTINCT subject) as subject_count
FROM student_scores;
```

실행 결과:
```
total_records: 8
score_count: 8
subject_count: 2
```

## SUM(): 합계 계산

```sql
-- 전체 점수 합계
SELECT 
    subject,
    SUM(score) as total_score,
    COUNT(*) as student_count,
    SUM(score) / COUNT(*) as average_score
FROM student_scores
GROUP BY subject;
```

실행 결과:

|subject|total_score|student_count|average_score|
|---|---|---|---|
|수학|350|4|87.50|
|영어|354|4|88.50|

## AVG(): 평균 계산

```sql
-- 과목별, 반별 평균 점수
SELECT 
    subject,
    class_id,
    AVG(score) as avg_score,
    MIN(score) as min_score,
    MAX(score) as max_score
FROM student_scores
GROUP BY subject, class_id
ORDER BY subject, class_id;
```

실행 결과:

|subject|class_id|avg_score|min_score|max_score|
|---|---|---|---|---|
|수학|101|88.50|85|92|
|수학|102|86.50|78|95|
|영어|101|86.50|85|88|
|영어|102|90.50|89|92|

## MIN()과 MAX(): 최솟값과 최댓값

```sql
-- 과목별 최고/최저 점수와 해당 학생
SELECT 
    subject,
    MAX(score) as highest_score,
    MIN(score) as lowest_score,
    (
        SELECT student_name 
        FROM student_scores s2 
        WHERE s2.subject = s1.subject 
        AND s2.score = MAX(s1.score)
        LIMIT 1
    ) as top_student
FROM student_scores s1
GROUP BY subject;
```

실행 결과:

|subject|highest_score|lowest_score|top_student|
|---|---|---|---|
|수학|95|78|정미경|
|영어|92|85|박민수|

## STDDEV()와 VARIANCE(): 표준편차와 분산

```sql
-- 과목별 점수 분포 통계
SELECT 
    subject,
    AVG(score) as mean,
    STDDEV(score) as std_dev,
    VARIANCE(score) as variance
FROM student_scores
GROUP BY subject;
```

# 고급 집계 기법

## 1. HAVING 절과 함께 사용

```sql
-- 평균이 88점 이상인 과목 찾기
SELECT 
    subject,
    AVG(score) as avg_score
FROM student_scores
GROUP BY subject
HAVING AVG(score) >= 88;
```

## 2. 윈도우 함수와 결합

```sql
-- 누적 평균 계산
SELECT 
    student_name,
    subject,
    score,
    AVG(score) OVER (
        PARTITION BY subject 
        ORDER BY exam_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) as running_avg
FROM student_scores;
```

## 3. 조건부 집계

```sql
-- 과목별, 성적 구간별 학생 수 계산
SELECT 
    subject,
    COUNT(CASE WHEN score >= 90 THEN 1 END) as A_grade,
    COUNT(CASE WHEN score >= 80 AND score < 90 THEN 1 END) as B_grade,
    COUNT(CASE WHEN score < 80 THEN 1 END) as C_grade
FROM student_scores
GROUP BY subject;
```

# 주의사항

1. NULL 값 처리
   - COUNT(*)를 제외한 모든 집계 함수는 NULL 값을 무시한다
   - NULL 값 처리를 위해 COALESCE나 IFNULL을 활용해야 할 수 있다

2. 성능 고려사항
   - GROUP BY 사용 시 인덱스 활용을 고려해야 한다
   - 대용량 데이터 집계 시 성능에 주의해야 한다

3. 정확도
   - 금액이나 중요한 수치 계산 시 반올림 처리에 주의해야 한다
   - AVG 함수 사용 시 소수점 자릿수를 적절히 조절해야 한다

# 활용 시나리오

## 1. 성적 분석 리포트

```sql
-- 종합 성적 분석
SELECT 
    subject,
    COUNT(*) as student_count,
    ROUND(AVG(score), 2) as avg_score,
    MIN(score) as min_score,
    MAX(score) as max_score,
    ROUND(STDDEV(score), 2) as std_dev,
    ROUND(AVG(CASE WHEN score >= 90 THEN 1.0 ELSE 0.0 END) * 100, 2) as pass_rate
FROM student_scores
GROUP BY subject;
```

## 2. 시계열 분석

```sql
-- 일자별 평균 점수 추이
SELECT 
    exam_date,
    subject,
    AVG(score) as avg_score,
    COUNT(*) as exam_count
FROM student_scores
GROUP BY exam_date, subject
ORDER BY exam_date, subject;
```

# 결론

SQL 집계 함수는 데이터 분석에서 필수적인 도구이며, 다음과 같은 장점을 제공한다:

1. 대량의 데이터를 의미 있는 정보로 요약할 수 있다
2. 복잡한 통계 분석을 간단한 쿼리로 수행할 수 있다
3. 데이터의 패턴과 트렌드를 파악하는 데 도움이 된다
4. 효율적인 리포팅 시스템 구축이 가능하다

집계 함수를 효과적으로 활용하기 위해서는 각 함수의 특성을 이해하고, 적절한 상황에 맞는 함수를 선택하는 것이 중요하다.