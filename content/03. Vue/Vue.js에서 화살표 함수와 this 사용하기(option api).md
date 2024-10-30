---
title: 무제 파일
publish: true
tags:
---
Vue.js에서는 this 바인딩이 특히 중요한데, Vue 인스턴스의 속성과 메서드에 접근하기 위해서다.

# Vue 메서드에서의 this

## 1. 기본 메서드 정의
```javascript
export default {
    data() {
        return {
            message: 'Hello'
        }
    },
    // 잘못된 방법
    methods: {
        greet: () => {
            // 화살표 함수에서는 this가 Vue 인스턴스를 가리키지 않음
            console.log(this.message) // undefined
        }
    },
    // 올바른 방법
    methods: {
        greet() {
            // 일반 함수에서는 this가 Vue 인스턴스를 가리킴
            console.log(this.message) // 'Hello'
        }
    }
}
```

## 2. 계산된 속성 (Computed)
```javascript
export default {
    data() {
        return {
            firstName: 'John',
            lastName: 'Doe'
        }
    },
    computed: {
        // 잘못된 방법
        fullName: () => {
            // this가 Vue 인스턴스를 가리키지 않음
            return `${this.firstName} ${this.lastName}`
        },
        // 올바른 방법
        fullName() {
            return `${this.firstName} ${this.lastName}`
        }
    }
}
```

# 콜백에서의 this

## 1. 비동기 처리
```javascript
export default {
    data() {
        return {
            users: []
        }
    },
    methods: {
        // 올바른 방법
        async fetchUsers() {
            try {
                const response = await axios.get('/api/users')
                this.users = response.data
            } catch (error) {
                console.error(error)
            }
        },
        // 화살표 함수가 유용한 경우
        processUsers() {
            this.users.forEach((user) => {
                // 여기서는 화살표 함수가 적절함
                console.log(`${this.message}: ${user.name}`)
            })
        }
    }
}
```

## 2. 이벤트 핸들러
```javascript
export default {
    methods: {
        // 템플릿의 @click 이벤트에 사용될 메서드
        handleClick() {
            // Vue 메서드로는 일반 함수 사용
            this.doSomething()
        },
        processData() {
            // 내부 콜백으로는 화살표 함수 사용이 적절
            setTimeout(() => {
                this.updateData()
            }, 1000)
        }
    }
}
```

# 라이프사이클 훅

```javascript
export default {
    // 잘못된 방법
    created: () => {
        // this가 Vue 인스턴스를 가리키지 않음
        this.fetchData()
    },
    // 올바른 방법
    created() {
        // this가 Vue 인스턴스를 올바르게 가리킴
        this.fetchData()
    }
}
```

# Composition API에서의 차이점

```javascript
import { ref, onMounted } from 'vue'

// Composition API에서는 this 바인딩에 대해 덜 신경써도 됨
export default {
    setup() {
        const message = ref('Hello')
        
        // 화살표 함수 사용 가능
        const greet = () => {
            console.log(message.value)
        }

        onMounted(() => {
            greet()
        })

        return {
            message,
            greet
        }
    }
}
```

# 주의해야 할 상황

## 1. watch에서의 사용
```javascript
export default {
    data() {
        return {
            searchText: ''
        }
    },
    // 잘못된 방법
    watch: {
        searchText: () => {
            // this가 Vue 인스턴스를 가리키지 않음
            this.doSearch()
        },
        // 올바른 방법
        searchText(newVal, oldVal) {
            this.doSearch()
        }
    }
}
```

## 2. Props 변경 감지
```javascript
export default {
    props: ['userData'],
    watch: {
        // 올바른 방법
        userData: {
            handler(newVal) {
                this.processUserData(newVal)
            },
            deep: true
        }
    }
}
```

# 권장 사항

1. **기본 규칙**
   - Vue 인스턴스 메서드는 일반 함수로 정의
   - 내부 콜백에는 화살표 함수 사용
   - 라이프사이클 훅은 일반 함수로 정의

2. **Composition API 사용 시**
   - setup() 내부에서는 화살표 함수 자유롭게 사용 가능
   - this 의존성이 줄어들어 더 유연한 함수 정의 가능

3. **메서드와 계산된 속성**
   - methods와 computed 속성은 일반 함수로 정의
   - 내부 로직에서 비동기 콜백이 필요한 경우 화살표 함수 사용