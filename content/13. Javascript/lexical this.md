---
title: lexical this
publish: true
tags:
---
Lexical this는 화살표 함수가 자신만의 this를 생성하지 않고, 자신을 감싸는 외부 스코프의 this를 그대로 사용하는 것을 의미한다.

# 일반 함수 vs 화살표 함수

## 일반 함수의 this
- 호출 방식에 따라 this가 동적으로 결정됨
- 함수가 어떻게 호출되었는지가 중요

```javascript
const user = {
    name: "John",
    greet: function() {
        console.log(`Hello, ${this.name}`);
    }
};

user.greet(); // "Hello, John"
const greet = user.greet;
greet(); // "Hello, undefined" (this가 전역 객체를 가리킴)
```

## 화살표 함수의 this (Lexical this)
- 함수가 선언된 위치의 this를 사용
- 호출 방식과 관계없이 this가 고정됨

```javascript
const user = {
    name: "John",
    greet: () => {
        console.log(`Hello, ${this.name}`);
    }
};

user.greet(); // this는 user가 아닌 외부 스코프의 this를 가리킴
```

# 실제 활용 사례

## 클래스 메서드에서의 콜백
```javascript
// 문제가 있는 코드
class Counter {
    constructor() {
        this.count = 0;
    }
    
    start() {
        setInterval(function() {
            // this가 전역 객체를 가리켜서 의도한 대로 동작하지 않음
            this.count++;
            console.log(this.count);
        }, 1000);
    }
}

// 화살표 함수로 해결
class Counter {
    constructor() {
        this.count = 0;
    }
    
    start() {
        setInterval(() => {
            // this가 Counter 인스턴스를 가리킴
            this.count++;
            console.log(this.count);
        }, 1000);
    }
}
```

## 이벤트 핸들러
```javascript
// 문제가 있는 코드
class App {
    constructor() {
        this.buttonClicks = 0;
        this.button = document.querySelector('button');
        
        this.button.addEventListener('click', function() {
            // this가 버튼 엘리먼트를 가리킴
            this.buttonClicks++; // 에러!
        });
    }
}

// 화살표 함수로 해결
class App {
    constructor() {
        this.buttonClicks = 0;
        this.button = document.querySelector('button');
        
        this.button.addEventListener('click', () => {
            // this가 App 인스턴스를 가리킴
            this.buttonClicks++;
        });
    }
}
```

# Lexical this가 유용한 상황

1. **비동기 콜백**
```javascript
class DataFetcher {
    constructor() {
        this.data = null;
    }
    
    fetch() {
        setTimeout(() => {
            // this가 DataFetcher 인스턴스를 올바르게 가리킴
            this.data = "fetched data";
            console.log(this.data);
        }, 1000);
    }
}
```

2. **메서드 체이닝**
```javascript
class Builder {
    constructor() {
        this.items = [];
    }
    
    addItem(item) {
        // 화살표 함수로 인해 this가 Builder 인스턴스를 가리킴
        process.nextTick(() => {
            this.items.push(item);
            console.log(this.items);
        });
        return this;
    }
}
```

# 주의사항

1. **객체의 메서드로 부적절한 경우**
```javascript
// 잘못된 사용
const obj = {
    name: "John",
    greet: () => {
        console.log(`Hello, ${this.name}`);
    }
};

// 올바른 사용
const obj = {
    name: "John",
    greet() {
        console.log(`Hello, ${this.name}`);
    }
};
```
[[객체 메서드에서 일반 함수와 화살표 함수의 차이]]

2. **프로토타입 메서드로 부적절한 경우**
```javascript
// 피해야 할 사용
Function.prototype.myBind = () => {
    console.log(this); // 잘못된 this 바인딩
};

// 올바른 사용
Function.prototype.myBind = function() {
    console.log(this); // 정상적인 this 바인딩
};
```

# 정리
1. **Lexical this의 핵심**
   - 화살표 함수는 자신의 this를 생성하지 않음
   - 선언된 위치의 this를 그대로 사용
   - 호출 방식에 관계없이 this가 유지됨

2. **사용하면 좋은 경우**
   - 콜백 함수
   - 이벤트 핸들러
   - 비동기 함수
   - 중첩 함수

3. **피해야 할 경우**
   - 객체의 메서드
   - 프로토타입 메서드
   - this를 동적으로 바인딩해야 하는 경우