---
title: docker로 dev 환경에서 실행하는 법
draft: false
tags:
---
> [!warning] settings 파일 확인
> manage.py에서 settings 파일이 알맞게 환경변수로 설정되어 있는지 확인해야한다
```sh
#!/bin/sh

python manage.py makemigrations
python manage.py migrate --no-input
python manage.py runserver 0.0.0.0:8000
```

docker-compose.yml에서 위 파일을 실행할 수 있도록 추가해주어야 한다. runserver만 하면 migration이 되지 않기 때문이다

```yml
version: "3.9"

services:
  db:
    container_name: my_project_db
    image: postgres:14
    volumes:
      - postgres_data:/var/lib/postgresql/data/
    env_file:
      - ".env"
    ports:
       - 5432:5432

  backend:
    container_name: my_project
    restart: always
    build:
      context: ./my_project
    command: /scripts/start_server.sh
    volumes:
      - ./my_project:/app
    env_file:
      - ".env"
    ports:
      - 8000:8000
    depends_on:
      - db

volumes:
  postgres_data:
```

마지막으로 django에 대한 이미지 정보를 담고 있는 Dockerfile에서 해당 스크립트에 대한 실행 권한을 주어야 한다
```Dockerfile
FROM python:3.12  
  
WORKDIR /app
  
COPY requirements.txt ./  
RUN pip install -r requirements.txt  
  
COPY ./scripts /scripts
RUN chmod -R +x /scripts

COPY .. .
```
