---
title: Docker Compose
draft: true
tags:
---
# Volume
## Bind mounts
내 파일시스템과 도커의 파일 시스템을 연동시키는 방법이다. 아래와 같이 나타내면 사용할 수 있다.

```yml
volumes:
    - (호스트 파일 시스템):(도커의 파일 시스템)
```
## command

# exec vs. run
https://docs.docker.com/reference/cli/docker/compose/exec/
https://docs.docker.com/reference/cli/docker/compose/run/

```sh
docker compose run --rm <service_name> <command>
```
docker compose exec는 기존에 있는 컨테이너 안에 원하는 명령어를 돌리는 것이다. docker compose run은 새로운 컨테이너를 만들어서 명령어를 돌리는 것이다. 
> [!info] 컨테이너 바로 삭제하기
> `docker compose run --rm <service_name> <command>`를 사용하면 실행 후 바로 컨테이너를 삭제해줄 수 있다
# Compose
https://docs.docker.com/reference/cli/docker/compose/
기본 compose 파일이 docker-compose.yml에서 compose.yml로 바뀌었다. 물론 docker-compose.yml도 여전히 자동으로 찾아서 실행해준다

## celery
celery도 docker 서비스로 따로 돌려주어야 장고와 함께 사용할 수 있다
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
