---
title: docker로 dev 환경에서 실행하는 법
publish: true
tags:
---
> [!warning] settings 파일 확인
> manage.py에서 settings 파일이 알맞게 환경변수로 설정되어 있는지 확인해야한다
## env
```.env
ALLOWED_HOSTS=example.com,api.example.com,localhost,127.0.0.1  
CORS_ORIGIN_WHITELIST=http://127.0.0.1:5173,http://localhost:5173
```

## bash file
### dev
```sh
#!/bin/sh

python manage.py makemigrations
python manage.py migrate --no-input
python manage.py runserver 0.0.0.0:8000
```
### production
```sh
#!/bin/sh

python manage.py collectstatic --no-input
python manage.py makemigrations
python manage.py migrate
gunicorn istat_back.wsgi -b 0.0.0.0:8000
```

## docker
### Dockerfile
```Dockerfile
FROM python:3.12



ENV PYTHONDONTWRITEBYTECODE=1 \

PYTHONUNBUFFERED=1

COPY requirements.txt /tmp/requirements.txt
COPY ./scripts /scripts

WORKDIR /app
EXPOSE 8000

RUN python -m venv /py && \
/py/bin/pip install --upgrade pip && \
/py/bin/pip install -r /tmp/requirements.txt && \
rm -rf /root/.cache/ && \
rm -rf /tmp && \
adduser \
--disabled-password \
--no-create-home \
django-user && \
chown -R django-user:django-user . && \
chmod -R +x /scripts



ADD . /app/

RUN chown -R django-user:django-user /app/static

ENV PATH="/scripts:/py/bin:$PATH"

USER django-user
```

### Docker compose
> [!warning] database
> volume을 꼭 설정해야 한다

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
