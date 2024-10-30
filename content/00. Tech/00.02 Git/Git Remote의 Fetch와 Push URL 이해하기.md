---
publish: true
tags:
---
# 기본 개념

## Remote URL 구조
```sh
# 일반적인 경우 (fetch와 push가 동일)
origin  https://github.com/myusername/project.git (fetch)
origin  https://github.com/myusername/project.git (push)

# fetch와 push가 다른 경우
origin  https://github.com/myusername/project.git (fetch)
origin  git@github.com:myusername/project.git (push)
```

# Fetch와 Push URL이 다른 경우

## 1. Protocol 차이
```sh
# HTTPS로 fetch, SSH로 push
origin  https://github.com/myusername/project.git (fetch)
origin  git@github.com:myusername/project.git (push)
```
- 읽기는 HTTPS로 하고 쓰기는 SSH로 하는 경우
- Security policy상 특정 protocol만 허용하는 경우

## 2. Mirroring 설정
```sh
# 서로 다른 repository로 fetch와 push
origin  https://github.com/original/project.git (fetch)
origin  https://github.com/mirror/project.git (push)
```
- 한 repository의 내용을 다른 repository로 mirroring할 때 사용
- Backup이나 synchronization 목적으로 활용

# URL 설정 방법

## 기본 설정
```sh
# remote 추가 시 하나의 URL만 지정
git remote add origin https://github.com/myusername/project.git
```

## 개별 URL 설정
```sh
# fetch와 push URL을 따로 설정
git remote add origin https://github.com/myusername/project.git
git remote set-url --push origin git@github.com:myusername/project.git
```

## URL 확인
```sh
# 모든 remote URL 확인
git remote -v

# 특정 remote의 상세 정보 확인
git remote show origin
```

# 활용 사례

## 1. Security Policy 대응
```sh
# 회사 network에서:
# - HTTPS는 proxy를 통해 허용
# - SSH는 직접 접근 허용
origin  https://github.com/company/project.git (fetch)
origin  git@github.com:company/project.git (push)
```

## 2. Repository Mirroring
```sh
# GitHub -> GitLab mirroring
origin  https://github.com/original/project.git (fetch)
origin  https://gitlab.com/mirror/project.git (push)
```

## 3. Read/Write 권한 분리
```sh
# Public 읽기용 URL과 authenticated 쓰기용 URL 분리
origin  https://public.github.com/project.git (fetch)
origin  https://private-token@github.com/project.git (push)
```

# 주의사항

1. **Security 고려사항**
   - HTTPS는 username/password 또는 token 인증 필요
   - SSH는 key 기반 인증 사용
   - 각 protocol의 security 특성 이해 필요

2. **Network 제한**
   - 기업 환경에서 특정 protocol이 block될 수 있음
   - Proxy 설정이 필요할 수 있음

3. **Authentication 관리**
   - HTTPS: credentials manager 사용
   - SSH: key 관리 필요

4. **URL 변경 시 주의점**
   ```sh
   # URL 변경 전 기존 설정 backup
   git remote -v > remote_backup.txt
   
   # URL 변경
   git remote set-url origin new-url
   # 또는
   git remote set-url --push origin new-push-url
   ```

# 실제 설정 예시

## Multiple Repository Mirroring
```sh
# 원본 repository 설정
git remote add origin https://github.com/original/project.git
git remote set-url --push origin git@github.com:mirror/project.git

# 추가 mirror repository 설정
git remote add gitlab https://gitlab.com/mirror/project.git
git push gitlab --mirror
```

## Hybrid 접근
```sh
# 일반적인 fetch는 HTTPS로
git remote add origin https://github.com/myusername/project.git

# push는 SSH로 설정
git remote set-url --push origin git@github.com:myusername/project.git

# CI/CD system용 추가 remote 설정
git remote add ci https://ci-token@github.com/myusername/project.git
```