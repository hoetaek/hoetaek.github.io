---
title: JavaScript Function.prototype.apply 이해하기
publish: false
tags:
---
apply는 JavaScript에서 함수를 호출할 때 this 컨텍스트와 인자를 지정할 수 있게 해주는 메서드다.

# 기본 문법
```javascript
function.apply(thisArg, [argsArray])
// thisArg: 함수 내부에서 this로 사용될 값
// argsArray: 함수에 전달할 인자 배열
```

# 주요 특징

## this 컨텍스트 지정
```javascript
const person = {
    name: "John",
    greet() {
        console.log(`안녕하세요, ${this.name}입니다!`);
    }
};

person.greet(); // "안녕하세요, John입니다!"

const greet = person.greet;
greet.apply(person); // "안녕하세요, John입니다!"
```

## 배열로 인자 전달
```javascript
function sum(a, b, c) {
    return a + b + c;
}

console.log(sum.apply(null, [1, 2, 3])); // 6
```

# 실제 활용 사례

## 배열에서 최대/최소값 찾기
```javascript
const numbers = [5, 6, 2, 3, 7];

// apply 사용
const max = Math.max.apply(null, numbers);
const min = Math.min.apply(null, numbers);

// 현대적인 방법 (spread 연산자)
const max2 = Math.max(...numbers);
const min2 = Math.min(...numbers);
```

## 객체 메서드 빌리기
```javascript
const computer = {
    name: "MacBook",
    printSpecs(...specs) {
        console.log(`${this.name} 스펙:`, ...specs);
    }
};

const phone = {
    name: "iPhone"
};

// computer의 메서드를 phone에서 사용
computer.printSpecs.apply(phone, ["A15", "6GB RAM"]);
// "iPhone 스펙: A15 6GB RAM"
```

# apply vs call vs bind

## apply

```javascript
function greet(greeting, punctuation) {
    console.log(`${greeting}, ${this.name}${punctuation}`);
}

const person = { name: "John" };
greet.apply(person, ["Hello", "!"]); // 배열로 인자 전달
```

## call

```javascript
// call은 인자를 개별적으로 전달
greet.call(person, "Hello", "!"); 
```

## bind

```javascript
// bind는 새로운 함수를 반환
const boundGreet = greet.bind(person);
boundGreet("Hello", "!");
```

# 현대적인 대안

## Spread 연산자
```javascript
// 기존 apply 사용
Math.max.apply(null, numbers);

// 현대적인 방법
Math.max(...numbers);
```

## arrow function & lexical this
```javascript
class Logger {
    constructor(prefix) {
        this.prefix = prefix;
    }

    log(...args) {
        // 화살표 함수는 상위 스코프의 this를 유지
        const print = () => {
            console.log(this.prefix, ...args);
        };
        print();
    }
}
```

this의 사용에 대한 참고자료 [[lexical this]]
# 주의사항

1. **null/undefined 전달**
   - strict 모드에서는 thisArg가 null/undefined일 때 전역 객체로 변환되지 않음

2. **성능 고려**
   - apply는 인자를 배열로 전달할 때 유용
   - 단순 함수 호출은 일반 호출이 더 빠를 수 있음

3. **화살표 함수**
   - 화살표 함수는 자체 this를 가지지 않아 apply로 this를 바꿀 수 없음
```javascript
const arrowFn = () => console.log(this);
arrowFn.apply({ name: "John" }); // this는 변경되지 않음
```