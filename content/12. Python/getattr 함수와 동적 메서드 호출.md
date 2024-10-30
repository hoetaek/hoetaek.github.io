---
title: getattr 함수와 동적 메서드 호출
publish: false
tags:
---
# getattr 기본 개념

getattr 함수는 객체의 속성이나 메서드를 문자열 이름으로 동적으로 접근할 수 있게 해주는 내장 함수다.

```python
# 기본 문법
getattr(객체, '속성_이름', 기본값)
```

# 기본 사용법

## 속성 접근
```python
class Person:
    def __init__(self):
        self.name = "John"
        self.age = 30

person = Person()

# 일반적인 접근
print(person.name)  # "John"

# getattr 사용
print(getattr(person, 'name'))  # "John"
print(getattr(person, 'age'))   # 30

# 존재하지 않는 속성에 대한 기본값 설정
print(getattr(person, 'address', 'Not Found'))  # "Not Found"
```

## 메서드 호출
```python
class Calculator:
    def add(self, x, y):
        return x + y
    
    def subtract(self, x, y):
        return x - y

calc = Calculator()

# 동적 메서드 호출
method_name = 'add'
result = getattr(calc, method_name)(10, 5)  # 15
```

# 고급 활용 사례

## 동적 메서드 라우팅
```python
class APIHandler:
    def get_users(self):
        return "Getting users"
    
    def get_posts(self):
        return "Getting posts"
    
    def handle_request(self, resource_type):
        method_name = f'get_{resource_type}'
        handler = getattr(self, method_name, None)
        
        if handler is None:
            raise ValueError(f"Unknown resource type: {resource_type}")
            
        return handler()

handler = APIHandler()
print(handler.handle_request('users'))  # "Getting users"
print(handler.handle_request('posts'))  # "Getting posts"
```

## 커맨드 패턴 구현
```python
class CommandExecutor:
    def command_start(self):
        return "Starting..."
    
    def command_stop(self):
        return "Stopping..."
    
    def command_restart(self):
        return "Restarting..."
    
    def execute(self, command):
        method_name = f'command_{command}'
        if not hasattr(self, method_name):
            return f"Unknown command: {command}"
        
        return getattr(self, method_name)()

executor = CommandExecutor()
commands = ['start', 'stop', 'restart', 'unknown']

for cmd in commands:
    print(f"Executing {cmd}: {executor.execute(cmd)}")
```

## 플러그인 시스템 구현
```python
class PluginManager:
    def __init__(self):
        self.plugins = {}
    
    def register_plugin(self, name, plugin_class):
        self.plugins[name] = plugin_class()
    
    def execute_plugin_method(self, plugin_name, method_name, *args, **kwargs):
        plugin = self.plugins.get(plugin_name)
        if plugin is None:
            raise ValueError(f"Plugin not found: {plugin_name}")
            
        method = getattr(plugin, method_name, None)
        if method is None:
            raise ValueError(f"Method not found: {method_name}")
            
        return method(*args, **kwargs)

# 플러그인 예시
class ImagePlugin:
    def resize(self, width, height):
        return f"Resizing image to {width}x{height}"
    
    def rotate(self, degrees):
        return f"Rotating image by {degrees} degrees"

# 사용 예시
manager = PluginManager()
manager.register_plugin('image', ImagePlugin)
print(manager.execute_plugin_method('image', 'resize', 800, 600))
```

# 성능 최적화

## 메서드 캐싱
```python
class OptimizedHandler:
    def __init__(self):
        self._method_cache = {}
    
    def get_method(self, method_name):
        if method_name not in self._method_cache:
            self._method_cache[method_name] = getattr(self, method_name)
        return self._method_cache[method_name]
    
    def execute(self, method_name, *args):
        return self.get_method(method_name)(*args)
```

# 실용적인 예제

## 데이터 변환기
```python
class DataTransformer:
    def transform_to_int(self, value):
        return int(value)
    
    def transform_to_float(self, value):
        return float(value)
    
    def transform_to_str(self, value):
        return str(value)
    
    def transform(self, value, target_type):
        method_name = f'transform_to_{target_type}'
        transformer = getattr(self, method_name, None)
        
        if transformer is None:
            raise ValueError(f"Unsupported type: {target_type}")
            
        return transformer(value)

transformer = DataTransformer()
data = ["123", "456.789", 789]
types = ["int", "float", "str"]

for value, target_type in zip(data, types):
    result = transformer.transform(value, target_type)
    print(f"Transformed {value} to {target_type}: {result} ({type(result)})")
```

# 주의사항

## 1. 보안 고려사항
```python
class User:
    def __init__(self):
        self.public_data = "Public"
        self._private_data = "Private"
    
    def get_safe_attribute(self, attr_name):
        if attr_name.startswith('_'):
            raise AttributeError("Cannot access private attributes")
        return getattr(self, attr_name)
```

## 2. 예외 처리
```python
def safe_getattr(obj, attr_name, default=None):
    try:
        return getattr(obj, attr_name)
    except (AttributeError, TypeError) as e:
        logging.warning(f"Error accessing {attr_name}: {e}")
        return default
```

# 디버깅 팁

## 속성 존재 여부 확인
```python
class DebugHelper:
    @staticmethod
    def check_attributes(obj, *attributes):
        results = {}
        for attr in attributes:
            exists = hasattr(obj, attr)
            value = safe_getattr(obj, attr, 'Not accessible')
            results[attr] = {
                'exists': exists,
                'value': value,
                'type': type(value) if exists else None
            }
        return results
```

# 결론

getattr 함수는 Python의 동적 특성을 활용하는 강력한 도구다. 플러그인 시스템, 커맨드 패턴, 동적 라우팅 등 다양한 설계 패턴을 구현할 때 유용하게 사용할 수 있다. 다만, 보안과 성능을 고려하여 적절히 사용해야 하며, 가능한 경우 정적 접근 방식을 우선적으로 고려하는 것이 좋다.