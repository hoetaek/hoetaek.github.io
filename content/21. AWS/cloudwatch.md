---
title: cloudwatch
publish: true
tags:
---
# CloudWatch Agent 로그 확인하기

## 로그 파일 위치

CloudWatch Agent의 로그 파일은 다음 경로에 위치한다:
```
/opt/aws/amazon-cloudwatch-agent/logs
```

## 로그 확인 방법

실시간으로 로그를 모니터링하기 위해 다음 명령어를 사용한다:
```sh
tail -f /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

이 명령어는:
- 로그 파일의 마지막 부분을 실시간으로 출력
- 새로운 로그가 추가될 때마다 자동으로 화면에 표시
- Ctrl + C를 눌러 모니터링을 중단할 수 있음