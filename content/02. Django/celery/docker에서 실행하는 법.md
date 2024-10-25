---
title: 무제 파일
draft: true
tags:
---
```yml
celery_worker:  
  build:  
    context: ./backend
    dockerfile: Dockerfile  
    args:  
      - DJANGO_SETTINGS_MODULE=my_project.settings
  command: celery -A my_project worker --loglevel=info  
  depends_on:  
    mysql:  
      condition: service_healthy  
    redis:  
      condition: service_healthy  
  environment:  
    - CELERY_BROKER_URL=redis://redis:6379/0  
  volumes:  
    - ./backend:/app
```

my_project라는 장고 프로젝트를 가정하고 작성했다. 장고 프로젝트의 Dockerfile을 그대로 사용하며, Dockerfile에는 app이라는 WORKDIR을 사용한다