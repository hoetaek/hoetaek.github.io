---
title: django에 celery 설정하는 법
publish: true
tags:
---
# 프로젝트 구조 준비
Django 프로젝트의 기본 구조는 다음과 같습니다:
```
my_project/
├── manage.py
├── my_project/
│   ├── __init__.py
│   ├── celery.py     # 새로 생성할 파일
│   ├── settings.py
│   └── urls.py
```

# Celery 설정하기

## Celery 설정 파일 생성
프로젝트 폴더 내에 `celery.py` 파일을 생성하고 아래 내용을 작성합니다:

```python
# my_project/my_project/celery.py
import os
from celery import Celery

# Django settings 모듈 설정
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'my_project.settings')

# Celery 앱 생성
app = Celery('my_project')

# Django settings에서 Celery 설정 가져오기
app.config_from_object('django.conf:settings', namespace='CELERY')

# 등록된 Django 앱에서 tasks.py 자동으로 로드
app.autodiscover_tasks()
```

## 프로젝트 전역 설정
`__init__.py` 파일을 수정하여 Celery 앱을 프로젝트 전역에서 사용할 수 있도록 설정합니다:

```python
# my_project/my_project/__init__.py
from .celery import app as celery_app

__all__ = ('celery_app',)
```

## Message Broker 설정
`settings.py` 파일에 Redis를 Message Broker로 설정합니다:

```python
# my_project/my_project/settings.py
CELERY_BROKER_URL = "redis://redis:6379/0"
```

# 설정 내용 설명

1. `celery.py` 설정:
   - `DJANGO_SETTINGS_MODULE`을 프로젝트의 settings 파일로 지정합니다
   - Celery 앱의 이름을 프로젝트명으로 설정합니다
   - `config_from_object`로 settings 파일에서 'CELERY'로 시작하는 설정을 가져옵니다
   - `autodiscover_tasks`로 각 앱의 tasks.py를 자동으로 찾아 등록합니다

2. `__init__.py` 설정:
   - `__all__` 변수는 `from module import *` 실행 시 import할 항목을 지정합니다
   - Celery app을 `celery_app`으로 이름 지정하여 다른 app들과 충돌을 방지합니다

3. `settings.py` 설정:
   - Redis를 Message Broker로 사용하도록 URL을 지정합니다
   - Redis 서버는 기본적으로 6379 포트를 사용합니다