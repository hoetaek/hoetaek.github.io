`sudo yum install -y docker`
https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-docker.html

## docker compose
```bash
sudo mkdir -p /usr/local/lib/docker/cli-plugins/
sudo curl -SL "https://github.com/docker/compose/releases/latest/download/docker-compose-linux-$(uname -m)" -o /usr/local/lib/docker/cli-plugins/docker-compose
sudo chmod +x /usr/local/lib/docker/cli-plugins/docker-compose
```
# 스크립트 개요
# 패키지 설치
```sh
#!/bin/bash  
yum update  
yum -y install docker  
service docker start  
usermod -a -G docker ec2-user  
chkconfig docker on  
pip3 install docker-compose
yum install git
reboot
```

이 스크립트는 Amazon Linux EC2 인스턴스에 Docker 환경을 구성하는 초기 설정 스크립트입니다.

# 스크립트 분석

```bash
#!/bin/bash
```
- 이 스크립트가 bash 셸로 실행되어야 함을 지정합니다
- 셔뱅(shebang) 라인이라고 부릅니다

```bash
yum update
```
- 시스템의 패키지 목록을 최신으로 업데이트합니다
- 보안 업데이트와 버그 수정사항을 포함합니다

```bash
yum -y install docker
```
- Docker 패키지를 설치합니다
- `-y` 옵션: 설치 중 나오는 모든 확인 메시지에 자동으로 'yes' 응답

```bash
service docker start
```
- Docker 데몬을 시작합니다
- 컨테이너를 실행할 수 있도록 Docker 서비스를 활성화합니다

```bash
usermod -a -G docker ec2-user
```
- `ec2-user`를 docker 그룹에 추가합니다
- sudo 없이도 Docker 명령어를 실행할 수 있게 됩니다
- `-a`: 기존 그룹 유지
- `-G`: 추가할 그룹 지정

```bash
chkconfig docker on
```
- 시스템 부팅 시 Docker 서비스가 자동으로 시작되도록 설정합니다
- 시스템 재시작 후에도 Docker가 자동으로 실행됩니다

```bash
pip3 install docker-compose
```
- Python pip를 사용하여 Docker Compose를 설치합니다
- 여러 Docker 컨테이너를 정의하고 실행하는 도구입니다

```bash
yum install git
```
- Git 버전 관리 시스템을 설치합니다
- 소스 코드 관리와 배포에 사용됩니다

```bash
reboot
```
- 시스템을 재시작합니다
- 모든 변경사항을 적용하기 위함입니다

# 사용 방법

1. 스크립트 생성
```bash
vim setup-docker.sh
# 스크립트 내용 붙여넣기
```

2. 실행 권한 부여
```bash
chmod +x setup-docker.sh
```

3. 스크립트 실행
```bash
sudo ./setup-docker.sh
```

# 주의사항

1. 권한
   - root 권한(sudo)으로 실행해야 합니다
   - 시스템 수준의 변경을 수행하기 때문입니다

2. 네트워크
   - 안정적인 인터넷 연결이 필요합니다
   - 패키지 다운로드와 설치를 위해서입니다

3. 재시작
   - 스크립트 마지막에 시스템이 재시작됩니다
   - 중요한 작업은 미리 저장해야 합니다

# 검증 방법

스크립트 실행 후 설치 확인:
```bash
# Docker 버전 확인
docker --version

# Docker 서비스 상태 확인
service docker status

# Docker Compose 버전 확인
docker-compose --version

# 사용자 그룹 확인
groups ec2-user
```

# 활용 예시

EC2 인스턴스 시작 시 User Data로 사용:
```bash
#!/bin/bash
# 로그 기록 추가
exec > >(tee /var/log/user-data.log|logger -t user-data -s 2>/dev/console) 2>&1

# 기존 스크립트 내용
yum update
yum -y install docker
...
```