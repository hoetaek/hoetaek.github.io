# 라이트세일 amazon linux 2023으로 인스턴스 시작

# 기본 패키지 설치
```sh
sudo yum install gcc libzstd-devel util-linux-user vim-enhanced htop git zsh cronie -y
```
- `gcc`: GNU Compiler Collection의 약자로, 프로그래밍 언어를 위한 컴파일러 모음
- `libzstd-devel`: Zstandard 압축 알고리즘을 위한 개발 라이브러리 - 프로그래밍에서 데이터 압축과 관련된 작업을 할 때 필요
- `util-linux-user`: 리눅스 시스템에서 다양한 유틸리티를 제공하는 패키지 - 나중에 zsh 변경을 위해 필요
- `vim-enhanced`: Vim 텍스트 에디터의 확장 버전 - 텍스트 파일을 편집할 때 다양한 고급 기능을 제공
- `htop`: 시스템의 프로세스와 리소스 사용 현황을 시각적으로 보여주는 도구
- `git`: 소스 코드 버전 관리 시스템 - 프로그래밍 프로젝트에서 코드의 버전을 관리하는 데 사용
- `zsh`: Z Shell의 약자로, 리눅스와 유닉스 시스템에서 사용되는 강력한 커맨드 라인 쉘
- `cronie`: cron 작업을 관리하는 프로그램 - 주기적인 작업을 자동으로 실행하게 설정할 때 사용
## Shell 설정
### oh-my-zsh 설치
```shell
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```
### shell plugin 설치
#### zsh-syntax-highlighting
```shell
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```
#### zsh-autosuggestions
```shell
git clone https://github.com/zsh-users/zsh-autosuggestions $ZSH_CUSTOM/plugins/zsh-autosuggestions
```

### .zshrc 수정
```
plugins=(
        git
        zsh-syntax-highlighting
        zsh-autosuggestions
)
```
# Python 설치
```sh
sudo dnf install python3.11 -y
sudo dnf install python3.11-pip -y
```

파이썬 3.11을 기본으로 설정하지 않기
- 아래는 기본으로 설정하는 코드이지만 실행시키지 않기
```sh
sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.9 1 sudo update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 2
```
```sh
sudo update-alternatives --config python3
```
- 3.11이 기본이 되면 dnf가 망가진다 대신 가상환경으로 해야 한다

## Python Virtualenv
```sh
python3.11 -m venv venv
source venv/bin/activate
python3 --version
deactivate
python3 --version
```

# nvm, nodejs, npm 설치
## nvm 설치
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```
## nodejs 설치
```shell
nvm install node
```

```
nvm use node
```
# nginx 설치

## 디렉토리 권한 설정
```shell
sudo chmod 705 /home/ec2-user/
```
- 권한 설정을 해야 ec2-user 하위 디렉토리를 읽어 웹페이지로 보여줄 수 있음
## nginx 설치 및 설정
```shell
sudo yum install nginx -y
```

### nginx 자동 시작 설정
```shell
sudo systemctl enable nginx
```

### nginx 설정
```sh
sudo vi /etc/nginx/nginx.conf
```
**/etc/nginx/nginx.conf**
- error_log 레벨을 notice에서 기본 값인 error로 변경
	- `error`: 오류 메시지를 기록합니다. → 아무 표시가 없을 때 기본 값
	- `warn`: 경고 메시지를 기록합니다.
	- `notice`: 중요한 이벤트 메시지를 기록합니다.
	- `info`: 상세한 정보를 기록합니다.
	- `debug`: 디버그 정보를 기록합니다.
- upstream backend 설정
- ```upstream backend {
	server localhost:8000;
	}```
- `server_tokens	off;`
- `client_max_body_size 100m;`
- server 부분부터 아래 내용 삭제
```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    keepalive_timeout   65;
    types_hash_max_size 4096;

    server_tokens	off;

	client_max_body_size 100m;
	
	upstream backend {
		server localhost:8000;
	}

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
}
```

# Project 설정하기
## git clone
### ssh key 만들기
[출처 링크](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent#generating-a-new-ssh-key)
```sh
ssh-keygen -t ed25519 -C "이메일"
```
### troubleshooting
- github와 연결이 안될 시에 ssh-agent에 추가하기
```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/istat-key
```
- `~/.ssh/config`
```
Host github.com
  User git
  Hostname github.com 
  IdentityFile ~/.ssh/id_ed25519 
  AddKeysToAgent yes
```
- github에 연결되는지 확인
- `ssh -T git@github.com`
### deploy key 등록 후 git clone
```sh
git clone git@github.com:프로젝트.git
```

## nginx
### nginx 설정 파일 추가
```sh
sudo cp nginx/prod.conf /etc/nginx/conf.d/prod.conf
```
### nginx 재시작
```
sudo systemctl restart nginx
```
### nginx 상태 확인
```sh
systemctl status nginx.service
```
# DB 설정
## postgresql 설치 후 DB 만들기
```sh
sudo dnf install postgresql15.x86_64 postgresql15-server -y
```
```sh
sudo postgresql-setup --initdb
sudo systemctl start postgresql  
sudo systemctl enable postgresql  
sudo systemctl status postgresql
```
## postgresql에 접속
```sh
sudo -u postgres psql
```
- 또는 postgres 비밀번호 바꾸는 방법 사용
```sh
# Change the ssh user password:  
sudo passwd postgres  
  
# Log in using the Postgres system account:  
su - postgres  
  
# Now, change the admin database password:  
psql -c "ALTER USER postgres WITH PASSWORD 'your-password';"  
exit
```
- 아래와 같이 유저를 만들고 데이터베이스 만들기
```
ALTER USER postgres WITH PASSWORD 'some_user';
CREATE USER some_user WITH PASSWORD 'user_password';
CREATE DATABASE database_name;
GRANT ALL PRIVILEGES ON DATABASE istat TO some_user;
ALTER DATABASE database_name OWNER TO some_user;
```
- hba_file 찾기
```
sudo -u postgres psql -t -P format=unaligned -c 'show hba_file';
```
- IPv4, IPv6 local connections의 method를 ident에서 md5로 수정
- postgresql 재시작
```
sudo systemctl restart postgresql
```
## .env 수정
```sh
cp project/.env.sample project/.env
```

-  django secret key 변경
	[secret key 만들러 가기](https://djecrety.ir)
- Allowed Hosts에 ip 주소 추가
```
ALLOWED_HOST= #ip주소

# Django Database Settings
DB_NAME=database_name
DB_USER=some_user
DB_PASSWORD=user_password
DB_HOST=localhost
```
## 라이브러리 설치
```sh
source venv/bin/activate

pip install -r requirements.txt
```
## DB migration
```sh
python manage.py migrate
```
## django 실행하기
script 실행권한 설정하기
```
chmod +x scripts/start_prod.sh
```
script 실행하기
```
scripts/start_prod.sh
```

script는 아래와 같다
```sh
python manage.py collectstatic --no-input

python manage.py makemigrations

python manage.py migrate

python manage.py compilemessages

gunicorn istat_back.wsgi -w 3 -b 0.0.0.0:8000 --error-logfile /var/log/django-error.log --access-logfile /var/log/django-access.log --timeout 100
```
# Frontend 설정하기
## 패키지 설치하기
```sh
npm install
```
## 빌드하기
```sh
npm run build
```
- ip 주소로 들어가서 확인하기

# Github Action 설정하기
- ip, user, pem 파일 필요
- ip는 instance의 ip 주소이다
- user는 ec2-user
- ssh_key는 pem 파일의 값이다

## 에러 해결
### Ident에서 md5로 수정
`FATAL:  Ident authentication failed`

- hba_file 찾기
```
sudo -u postgres psql -t -P format=unaligned -c 'show hba_file';
```
- IPv4, IPv6 local connections의 method를 ident에서 md5로 수정
- postgresql 재시작
	- `sudo systemctl restart postgresql`
### db의 Owner 설정
`permission denied for schema public`
- `ALTER DATABASE database_name OWNER TO some_user;`

### 폰트 못 읽을 때
```python
import shutil
import matplotlib

shutil.rmtree(matplotlib.get_cachedir())
```
한 후에 파일 삭제
