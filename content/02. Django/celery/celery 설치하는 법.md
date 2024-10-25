---
title: 설치하는 법
draft: true
tags:
---
## 설치하는 방법
https://docs.celeryq.dev/en/stable/django/first-steps-with-django.html#using-celery-with-django
### celery.py
프로젝트 폴더 내에 파일들을 만들어 celery 설정을 한다
먼저 celery.py를 만들어주고 아래와 같이 입력한다. 편의상 프로젝트의 이름을 my_project라고 하겠다
```python 
# my_project/my_project/celery.py

import os

from celery import Celery

os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'my_project.settings')

app = Celery('my_project')

app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django apps.
app.autodiscover_tasks()
```
1. DJANGO_SETTINGS_MODULE을 settings 파일로 설정을 한다. 만약 파일명이 다르다면 변경해야 한다
2. 편의상 celery의 namespace를 프로젝트명으로 설정한다
3. config_from_object는 settings 파일에서 celery 관련 설정을 받아온다. 이때 비효율적으로 모든 설정을 보기보다는 CELERY로 시작하는 변수들만 살펴본다 [celery config](https://docs.celeryq.dev/en/stable/userguide/configuration.html#configuration)
4. 마지막으로 자동으로 작업을 검색할 수 있도록 autodiscover_tasks를 해준다
### 프로젝트 전역으로 설정하기
```python
# my_project/my_project/__init__.py

from .celery import app as celery_app

__all__ = ('celery_app',)
```
\__all\__ 은 패키지에서 `from module import *`할 때 무엇을 import하도록 명시적으로 허용해줄지 나타내는 특별한 변수이다
celery 내의 app이 wsgi 등과 충돌하지 않도록 celery_app으로 import하고 모든 app에서 사용할 수 있도록 한다 
### settings 수정
redis를 message broker로 설정하기
```python
CELERY_BROKER_URL = "redis://redis:6379/0"
```
