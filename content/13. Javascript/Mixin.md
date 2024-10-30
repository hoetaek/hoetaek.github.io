---
title: JavaScript Mixin으로 코드 재사용하기
publish: false
tags:
---
Mixin은 상속을 사용하지 않고도 클래스에 메서드를 추가할 수 있는 방법이다. 특히 다중 상속이 필요한 상황에서 유용하게 사용할 수 있다.

# Mixin의 개념
- 상속 없이 메서드를 클래스에 추가하는 방법
- 하나의 클래스에 여러 Mixin을 조합할 수 있음
- 코드 재사용성을 높이는 유연한 방법

# 기본 사용법

## Object.assign을 사용한 방법
```javascript
// Mixin 정의
const sayHiMixin = {
    sayHi() {
        console.log(`Hello ${this.name}`);
    },
    sayBye() {
        console.log(`Bye ${this.name}`);
    }
};

class User {
    constructor(name) {
        this.name = name;
    }
}

// Mixin 적용
Object.assign(User.prototype, sayHiMixin);

// 사용
const user = new User("John");
user.sayHi(); // "Hello John"
```

# 고급 활용

## 이벤트 처리 Mixin 예시
```javascript
const eventMixin = {
    on(eventName, handler) {
        if (!this._eventHandlers) this._eventHandlers = {};
        if (!this._eventHandlers[eventName]) {
            this._eventHandlers[eventName] = [];
        }
        this._eventHandlers[eventName].push(handler);
    },

    off(eventName, handler) {
        const handlers = this._eventHandlers?.[eventName];
        if (!handlers) return;
        for (let i = 0; i < handlers.length; i++) {
            if (handlers[i] === handler) {
                handlers.splice(i--, 1);
            }
        }
    },

    trigger(eventName, ...args) {
        if (!this._eventHandlers?.[eventName]) return;
        this._eventHandlers[eventName].forEach(handler => handler.apply(this, args));
    }
};

// 사용 예시
class Menu {
    choose(value) {
        this.trigger("select", value);
    }
}

Object.assign(Menu.prototype, eventMixin);

// 동작 확인
const menu = new Menu();
menu.on("select", value => console.log(`선택된 값: ${value}`));
menu.choose("123"); // "선택된 값: 123"
```

splice 코드 관련 참고자료 [[JavaScript에서 배열 요소 제거 시 인덱스 처리하기]]
apply 관련 참고자료 [[JavaScript Function.prototype.apply 이해하기]]
# 실전 활용 패턴

## 여러 Mixin 조합하기
```javascript
const timestampMixin = {
    getTimestamp() {
        return new Date().getTime();
    }
};

const loggingMixin = {
    log(message) {
        console.log(`[${this.getTimestamp()}] ${message}`);
    }
};

class Logger { }

// 여러 Mixin 순차적 적용
Object.assign(Logger.prototype, timestampMixin, loggingMixin);

const logger = new Logger();
logger.log("테스트 메시지"); // "[1635831234567] 테스트 메시지"
```

# 주의사항과 모범 사례

## 주의사항
1. **이름 충돌** 
   - 여러 Mixin의 메서드 이름이 충돌할 수 있음
   - 마지막으로 적용된 Mixin의 메서드가 이전 메서드를 덮어씀

## 모범 사례
1. **명확한 책임**
   - 각 Mixin은 단일 책임을 가지도록 설계
   - 관련된 기능들을 하나의 Mixin으로 그룹화

2. **네이밍 규칙**
   - Mixin 이름에 용도를 명확히 표현
   - 메서드 이름 충돌을 피하기 위한 접두사 사용

```javascript
const ajaxMixin = {
    ajax_get() { /* ... */ },
    ajax_post() { /* ... */ }
};

const validationMixin = {
    validate_email() { /* ... */ },
    validate_phone() { /* ... */ }
};
```

**참고자료**
- JavaScript.info: https://ko.javascript.info/mixins
