---
title: JavaScript에서 배열 요소 제거 시 인덱스 처리하기
publish: false
tags:
---
배열에서 요소를 제거할 때 `splice(i--, 1)` 패턴은 매우 유용하다. 이 패턴을 통해 배열 순회 중 요소를 안전하게 제거할 수 있다.

# 기본 개념

## splice 메서드
```javascript
array.splice(startIndex, deleteCount);
// startIndex: 제거를 시작할 위치
// deleteCount: 제거할 요소의 개수
```

## 후위 감소 연산자(--)
```javascript
let i = 5;
i--; // i를 사용한 후 1 감소
--i; // i를 1 감소시킨 후 사용
```

# 문제 상황
```javascript
const arr = ['A', 'B', 'C', 'D'];

// 잘못된 방법
for (let i = 0; i < arr.length; i++) {
    if (arr[i] === 'B') {
        arr.splice(i, 1); // 'B' 제거
        // 문제: 다음 요소를 건너뛰게 됨
    }
}
```

# 해결 방법: splice(i--, 1)

## 동작 원리
```javascript
const arr = ['A', 'B', 'C', 'D'];

for (let i = 0; i < arr.length; i++) {
    if (arr[i] === 'B') {
        arr.splice(i--, 1);
        // 1. splice(i, 1) 실행
        // 2. i 값 감소
        // 3. 다음 반복에서 i++ 실행
    }
}
```

## 단계별 실행 과정
```javascript
const arr = ['A', 'B', 'C', 'D'];
let i = 1; // 'B'의 위치

// 1단계: splice(1, 1) 실행
// ['A', 'B', 'C', 'D'] => ['A', 'C', 'D']

// 2단계: i-- 실행
// i가 1에서 0으로 감소

// 3단계: 다음 반복의 i++ 실행
// i가 0에서 1로 증가하여 'C' 검사 가능
```

# 실제 활용 예시

## 특정 조건의 요소 모두 제거
```javascript
const numbers = [1, 2, 3, 2, 4, 2, 5];

// 모든 2 제거하기
for (let i = 0; i < numbers.length; i++) {
    if (numbers[i] === 2) {
        numbers.splice(i--, 1);
    }
}
console.log(numbers); // [1, 3, 4, 5]
```

## 객체 배열에서 사용
```javascript
const users = [
    { id: 1, name: "John" },
    { id: 2, name: "Jane" },
    { id: 3, name: "John" }
];

// 이름이 "John"인 모든 사용자 제거
for (let i = 0; i < users.length; i++) {
    if (users[i].name === "John") {
        users.splice(i--, 1);
    }
}
console.log(users); // [{ id: 2, name: "Jane" }]
```

# 주의사항

1. **배열 크기 변화**
   - splice는 배열의 길이를 변경함
   - 순회 중인 배열 수정 시 주의 필요

2. **성능 고려**
   - 대규모 배열의 경우 filter 사용 고려
   - splice는 배열 재구성 비용이 듦

3. **대안적 방법**
```javascript
// filter를 사용한 방법
const numbers = [1, 2, 3, 2, 4, 2, 5];
const filtered = numbers.filter(num => num !== 2);
```