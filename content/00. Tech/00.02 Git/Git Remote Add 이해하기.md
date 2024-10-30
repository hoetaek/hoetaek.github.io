---
publish: true
tags:
---
# Remote Add 기본 개념

`git remote add`는 다음 형식을 가진다:
```sh
git remote add <name> <url>
```
여기서 `<name>`은 remote repository에 대해 우리가 지정하는 별칭이다.

# 일반적으로 사용되는 Remote 이름들

1. **origin**
   - 가장 일반적으로 사용되는 이름
   - Convention일 뿐, 다른 이름 사용 가능
   ```sh
   git remote add origin https://github.com/username/repo.git
   ```

2. **upstream**
   - Fork한 원본 repository를 지칭할 때 주로 사용
   - 역시 Convention일 뿐, 다른 이름 사용 가능
   ```sh
   git remote add upstream https://github.com/original/repo.git
   ```

3. **custom name**
   - 어떤 이름이든 사용 가능
   ```sh
   git remote add myrepo https://github.com/username/repo.git
   git remote add backup https://github.com/username/backup.git
   git remote add ci https://github.com/username/repo.git
   ```

# Remote 관련 주요 명령어

```sh
# Remote 목록 확인
git remote

# Remote 상세 정보 확인
git remote -v

# Remote 추가
git remote add <name> <url>

# Remote URL 변경
git remote set-url <name> <new-url>

# Remote 제거
git remote remove <name>

# Remote 이름 변경
git remote rename <old-name> <new-name>
```

# 실제 사용 예시

```sh
# 여러 개의 remote 추가
git remote add origin https://github.com/username/repo.git
git remote add backup https://github.com/username/backup.git
git remote add testing https://github.com/username/testing.git

# 결과 확인
git remote -v
origin  https://github.com/username/repo.git (fetch)
origin  https://github.com/username/repo.git (push)
backup  https://github.com/username/backup.git (fetch)
backup  https://github.com/username/backup.git (push)
testing https://github.com/username/testing.git (fetch)
testing https://github.com/username/testing.git (push)
```

즉, `origin`, `upstream`, `ci` 등은 단순히 관례적으로 사용되는 이름일 뿐, git remote add 명령어에서 어떤 이름이든 사용할 수 있다.