---
publish: true
tags:
---
## Origin
- 기본적으로 클론한 원격 저장소를 가리킴
- git clone 시 자동으로 origin이라는 이름으로 설정됨
- 일반적으로 본인이 소유한 원격 저장소를 의미

## Upstream
- 원본 원격 저장소를 가리킴
- fork한 저장소의 원본 저장소를 의미
- 수동으로 추가해야 함

# 실제 사용 예시

## 일반적인 프로젝트 작업
```sh
# 저장소 클론
git clone https://github.com/username/repo.git
# 자동으로 origin 설정됨

# origin 확인
git remote -v
# origin  https://github.com/username/repo.git (fetch)
# origin  https://github.com/username/repo.git (push)
```

## Fork한 프로젝트 작업
```sh
# 1. Github에서 프로젝트 fork

# 2. fork한 저장소 클론
git clone https://github.com/myusername/project.git

# 3. upstream 추가
git remote add upstream https://github.com/original/project.git

# 4. remote 목록 확인
git remote -v
# origin    https://github.com/myusername/project.git (fetch)
# origin    https://github.com/myusername/project.git (push)
# upstream  https://github.com/original/project.git (fetch)
# upstream  https://github.com/original/project.git (push)
```

> [!info] fetch와 push
> fetch와 push url은 언제 다르게 설정하는 걸까?
> [[Git Remote의 Fetch와 Push URL 이해하기]]

> [!info] git remote add
> remote의 이름 convention과 git remote add의 의미에 대해서 알고 싶다면?
> [[Git Remote Add 이해하기]]
# 주요 작업 패턴

## Origin과 작업하기
```sh
# 변경사항 가져오기
git fetch origin
git pull origin main

# 변경사항 업로드
git push origin main
```

## Upstream과 작업하기
```sh
# upstream의 최신 변경사항 가져오기
git fetch upstream

# upstream의 변경사항을 로컬 main 브랜치에 병합
git checkout main
git merge upstream/main

# 병합된 변경사항을 origin에 반영
git push origin main
```

# 일반적인 워크플로우

1. **초기 설정**
   ```sh
   # fork한 저장소 클론
   git clone https://github.com/myusername/project.git
   
   # upstream 추가
   git remote add upstream https://github.com/original/project.git
   ```


1. **기능 개발**
   ```sh
   # 새로운 브랜치 생성
   git checkout -b feature-branch
   
   # 작업 후 커밋
   git add .
   git commit -m "Add new feature"
   
   # origin에 푸시
   git push origin feature-branch
   ```

3. **upstream과 동기화**
   ```sh
   # upstream의 최신 변경사항 가져오기
   git fetch upstream
   git checkout main
   git merge upstream/main
   
   # origin 동기화
   git push origin main
   ```

# 주의사항

1. **권한 관리**
   - origin: 일반적으로 push 권한 있음
   - upstream: 일반적으로 push 권한 없음 (PR로 기여)

2. **동기화 주기**
   - upstream의 변경사항을 자주 동기화하는 것이 좋음
   - 충돌 방지와 최신 코드 유지를 위해 중요

3. **브랜치 관리**
   - main 브랜치는 upstream과 동기화 상태 유지
   - 새로운 기능은 feature 브랜치에서 개발

4. **remote 이름 관례**
   - origin: 관례적으로 사용되는 이름
   - upstream: 관례적으로 사용되는 이름이지만, 다른 이름도 사용 가능