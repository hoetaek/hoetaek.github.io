---
title: django admin에 css 적용시키는 방법
draft: true
tags:
---

#dev/django #dev/docker #dev/nginx

## django

- admin url 설정

```python
from django.urls import path
from django.contrib import admin

from .api import api

urlpatterns = [
	path("admin/", admin.site.urls),
	path("api/", api.urls),
	]
```

- settings.py에 collectstatic의 destination 설정

```python
STATIC_URL = "/static/"

STATIC_ROOT = BASE_DIR / "static"
```

- script 파일에 collectstatic 추가

```sh
python manage.py collectstatic --no-input
```

## docker

### dockerfile

- django project directory에 대해서 권한 유지한채로 복사

```dockerfile
COPY --chown=django-user:django-use . /code
RUN chown -R django-user /code
```

### docker-compose yml

- static volume 설정

  ```yml
  backend:
    volumes:
      - static:/code/static
  ---
  nginx:
    volumes:
      - static:/code/static
  ---
  volumes:
    static:
  ```

## nginx

### dev

```conf
# backend urls

location ~ ^/(admin|api|static) {

proxy_redirect off;

proxy_pass http://backend;

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

proxy_set_header Host $http_host;

}
```

### production

- url 설정하기

```conf
location ~ ^/(admin|api) {

proxy_redirect off;

proxy_pass http://backend;

proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

proxy_set_header Host $http_host;

}
```

- static url 설정하기

```
# static files

location /static {

autoindex off;

alias /code/static/;

# Some basic cache-control for static files to be sent to the browser

location ~* \.(?:ico|css|js|gif|jpe?g|png)$ {
	expires max;
	
	add_header Pragma public;
	
	add_header Cache-Control "pubilc, must-revalidate, proxy-revalidate";
	}
}
```
