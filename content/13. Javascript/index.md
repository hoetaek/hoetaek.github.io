---
title: javascript
---
# NPM
## nvm
- nvm을 이용하면 디렉토리마다 node의 버전을 따로 관리할 수 있다
- 예시: `nvm use 16`
설치하는 방법
- [링크](https://github.com/nvm-sh/nvm)
- `curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash`
# Data type
##  Set
Set의 특정 인덱스 값을 읽으려면?
- Set은 순서가 보장이 안되므로 인덱스로 값을 읽지 못한다
- 대신에 array로 변환한 후에 읽어야 한다
Set에 값이 포함되어 있는지 알 수 있는 방법은?
- .has()를 이용한다
map 메소드를 사용하고 싶다면?
- 새로운 Array에 복사를 하고 사용한다
- 예시: \[...mySet].map(x => x * 3)
## Object
값의 개수 구하는 법은?
- Object.keys()를 이용한다
- Object.getOwnPropertyNames()를 이용한다
- for ... in을 이용하여 값을 increment하여 구한다
key에 포함되어 있는지 어떻게 알 수 있을까?
- in 키워드를 사용한다(element in Object)
# Mixin
- 자바스크립트에서 상속 외에도 다른 class와 결합할 수 있는 방법
## 사용 방법
- Object.assign을 사용한다
```javascript
class User extends Person { 
	// ... 
	} 

Object.assign(User.prototype, sayHiMixin);
```
## 출처(참고문헌)
- https://ko.javascript.info/mixins
