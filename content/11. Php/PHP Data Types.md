---
publish: true
tags:
---
# Scalar Types (스칼라 타입)

## String
문자열을 나타내는 타입으로, 텍스트 데이터를 처리한다.
```php
// 문자열 생성
$str = "Hello";
$str2 = 'World';
$str3 = "Hello {$str2}"; // 변수 보간

// 주요 메서드
strlen($str);          // 길이
strtoupper($str);      // 대문자
strtolower($str);      // 소문자
substr($str, 0, 3);    // 부분 문자열
explode(' ', $str);    // 문자열 분할
trim($str);           // 공백 제거
str_contains($str, 'el'); // 포함 여부 확인
```

## Integer & Float
정수와 부동소수점 숫자를 나타낸다.
```php
// 정수
$int = 42;
$hex = 0x2A;      // 16진수
$oct = 0o52;      // 8진수
$bin = 0b101010;  // 2진수

// 실수
$float = 42.123;
$scientific = 1.2e3;

// 주요 함수
is_int($int);           // 정수 여부
is_float($float);       // 실수 여부
round($float, 2);       // 반올림
ceil($float);          // 올림
floor($float);         // 내림
```

## Boolean
true 또는 false 값을 가진다.
```php
$bool = true;
$bool2 = false;

// 불리언 변환
$result = (bool) "";     // false
$result2 = (bool) "0";   // false
$result3 = (bool) [];    // false
$result4 = (bool) "1";   // true
```

# Compound Types (복합 타입)

## Array
PHP에서 가장 유용한 데이터 타입 중 하나로, 순서가 있는 맵이다.

```php
// 배열 생성
$arr = [1, 2, 3];
$assoc = ['name' => 'John', 'age' => 30];

// 배열 조작
array_push($arr, 4);         // 끝에 추가
array_pop($arr);            // 끝에서 제거
array_unshift($arr, 0);     // 앞에 추가
array_shift($arr);          // 앞에서 제거

// 배열 검색
in_array(2, $arr);          // 값 존재 여부
array_key_exists('name', $assoc); // 키 존재 여부
array_search(2, $arr);      // 값의 키 찾기

// 배열 변환
$mapped = array_map(fn($x) => $x * 2, $arr);
$filtered = array_filter($arr, fn($x) => $x > 1);
$reduced = array_reduce($arr, fn($carry, $item) => $carry + $item, 0);

// 배열 정렬
sort($arr);                // 값으로 정렬
asort($assoc);            // 값으로 정렬 (연관배열)
ksort($assoc);            // 키로 정렬
rsort($arr);              // 역순 정렬

// 유용한 배열 팁
array_chunk($arr, 2);     // 배열 분할
array_merge($arr1, $arr2); // 배열 합치기
array_diff($arr1, $arr2);  // 차집합
array_intersect($arr1, $arr2); // 교집합
array_unique($arr);       // 중복 제거
```

## WeakMap
참조가 해제되면 자동으로 가비지 컬렉팅되는 객체 맵이다.
```php
$map = new WeakMap();
$obj = new stdClass();

$map[$obj] = 'data';
// $obj = null; // 참조 해제시 자동 메모리 정리
```

# 특수 타입

## Null
명시적으로 값이 없음을 나타낸다.
```php
$var = null;
is_null($var);        // null 체크
$value = $var ?? 'default'; // null 병합 연산자
```

## Resource
외부 자원의 참조를 나타낸다.
```php
$file = fopen('file.txt', 'r');
is_resource($file);    // resource 체크
fclose($file);        // 자원 해제
```

# 타입 변환
```php
// 명시적 변환
$int = (int) "123";
$str = (string) 123;
$array = (array) $object;

// 타입 검사
$type = gettype($variable);
is_string($variable);
is_array($variable);
is_object($variable);
```

# 데이터 타입 비교
| 타입       | 특징         | 주요 용도       |
| -------- | ---------- | ----------- |
| String   | 텍스트 처리     | 문자열 데이터 처리  |
| Integer  | 정수 연산      | 수치 계산       |
| Float    | 부동소수점 연산   | 실수 계산       |
| Array    | 순차/연관 배열   | 데이터 컬렉션     |
| WeakMap  | 약한 참조 객체 맵 | 메모리 효율적인 캐싱 |
| Resource | 외부 자원 참조   | 파일, DB 연결 등 |
| Null     | 값 없음 표현    | 초기화, 조건 처리  |