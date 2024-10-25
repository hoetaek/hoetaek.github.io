---
title: Linux
---
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
# 로그
## 연속해서 보는 법
```sh
tail -f path/to/file.log
```
