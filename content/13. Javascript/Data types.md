---
title: Data types
publish: true
tags:
---
JavaScript의 데이터 타입들의 특징과 활용법을 알아보자.

## Primitive Types (원시 타입)

### String
문자열을 나타내는 타입으로, 텍스트 데이터를 다룬다.

```javascript
// 문자열 생성
const str = "Hello";
const str2 = 'World';
const str3 = `Hello ${str2}`; // 템플릿 리터럴

// 주요 메서드
str.length; // 길이
str.toUpperCase(); // 대문자
str.toLowerCase(); // 소문자
str.substring(0, 3); // 부분 문자열
str.split(' '); // 문자열 분할
str.trim(); // 공백 제거
str.includes('el'); // 포함 여부 확인
```

### Number
숫자를 나타내는 타입으로, 정수와 실수를 모두 포함한다.

```javascript
// 숫자 생성
const num = 42;
const float = 42.123;

// 주요 메서드와 프로퍼티
Number.isInteger(num); // 정수 여부
Number.isFinite(num); // 유한수 여부
num.toFixed(2); // 소수점 자릿수 지정
Number.MAX_SAFE_INTEGER; // 최대 안전 정수
Number.MIN_SAFE_INTEGER; // 최소 안전 정수
```

### Boolean
true 또는 false 값을 가지는 논리 타입이다.

```javascript
const bool = true;
const bool2 = false;

// 흔한 불리언 변환
!!0; // false
!!1; // true
!!{}; // true
!!''; // false
```

### Null & Undefined
값이 없음을 나타내는 특별한 타입이다.
```javascript
let val = null; // 명시적으로 값이 없음
let val2; // undefined, 값이 할당되지 않음

// 확인 방법
val === null; // true
typeof val2 === 'undefined'; // true
```

## Reference Types (참조 타입)

### Array
순서가 있는 데이터 컬렉션을 나타낸다.

```javascript
// 배열 생성
const arr = [1, 2, 3];
const arr2 = new Array(3); // 길이가 3인 빈 배열

// 주요 메서드
arr.push(4); // 끝에 추가
arr.pop(); // 끝에서 제거
arr.shift(); // 앞에서 제거
arr.unshift(0); // 앞에 추가
arr.includes(2); // 포함 여부
arr.indexOf(2); // 인덱스 찾기

// 배열 변형
arr.map(x => x * 2); // 각 요소 변환
arr.filter(x => x > 1); // 조건에 맞는 요소만
arr.reduce((sum, x) => sum + x, 0); // 누적 계산
arr.slice(1, 3); // 부분 배열
arr.splice(1, 1); // 요소 삭제/추가
```

### Set
중복을 허용하지 않는 값의 컬렉션이다.

```javascript
// Set 생성과 조작
const mySet = new Set([1, 2, 3]);
mySet.add(4);
mySet.delete(1);
mySet.has(2);
mySet.size;

// 배열 변환이 필요한 작업
[...mySet].map(x => x * 3);
Array.from(mySet);
```

### Map
키-값 쌍의 컬렉션으로, Object와 달리 키가 어떤 타입이든 될 수 있다.

```javascript
// Map 생성과 조작
const myMap = new Map();
myMap.set('key', 'value');
myMap.set(42, 'number key');
myMap.get('key');
myMap.has('key');
myMap.delete('key');
myMap.size;

// 순회
myMap.forEach((value, key) => console.log(key, value));
for (const [key, value] of myMap) {
    console.log(key, value);
}
```

### Object
키-값 쌍을 저장하는 기본적인 자료구조다.

```javascript
// 객체 생성과 조작
const obj = {
    name: 'John',
    age: 30
};

// 속성 접근
obj.name;
obj['age'];

// 존재 여부 확인
'name' in obj;
obj.hasOwnProperty('age');

// 객체 조작
Object.keys(obj); // 키 배열
Object.values(obj); // 값 배열
Object.entries(obj); // 키-값 쌍 배열
Object.freeze(obj); // 불변 객체로 만들기
```

## 타입 변환

### 명시적 변환
```javascript
// 문자열로 변환
String(123); // "123"
(123).toString(); // "123"

// 숫자로 변환
Number("123"); // 123
parseInt("123", 10); // 123
parseFloat("123.45"); // 123.45

// 불리언으로 변환
Boolean(""); // false
Boolean(1); // true
```

### 암시적 변환
```javascript
"123" + 456; // "123456"
"123" - 0; // 123
!!0; // false
if ("") { /* 실행되지 않음 */ }
```

## 타입별 특징 비교

| 타입      | 특징             | 주요 용도    |
| ------- | -------------- | -------- |
| String  | 불변, 인덱스 접근 가능  | 텍스트 처리   |
| Number  | 64비트 부동소수점     | 수치 계산    |
| Boolean | true/false     | 조건문      |
| Array   | 순서 있는 컬렉션      | 데이터 리스트  |
| Set     | 중복 없는 컬렉션      | 고유값 관리   |
| Map     | 키-값 쌍(키 타입 자유) | 복잡한 매핑   |
| Object  | 키-값 쌍(문자열 키)   | 구조화된 데이터 |