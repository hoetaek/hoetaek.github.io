---
publish: true
tags:
---
Poetry는 Python의 의존성 관리와 패키징을 위한 현대적인 도구다. 이 문서에서는 macOS 환경에서 Poetry를 사용하여 프로젝트 내부에 가상환경을 설정하고 관리하는 방법을 다룬다.

## Poetry 설치

### Homebrew를 사용한 설치 (권장)
```sh
brew install poetry
```

### 공식 설치 스크립트 사용
```sh
curl -sSL https://install.python-poetry.org | python3 -
```

### 설치 확인
```sh
poetry --version
```

## 프로젝트 초기 설정

### 1. 새 프로젝트 생성
```sh
poetry new my-project
cd my-project

# 또는 기존 프로젝트에서 초기화
poetry init
```

### 2. 가상환경 설정
프로젝트 디렉토리 내부에 가상환경을 생성한다.
```sh
# 가상환경을 프로젝트 내부에 생성하도록 설정
poetry config virtualenvs.in-project true

# 가상환경 경로 설정
poetry config virtualenvs.path "./.venv"

# 의존성 설치 및 업데이트
poetry install && poetry update
```

## 의존성 관리

### 1. 패키지 추가 및 제거
```sh
# 패키지 추가
poetry add requests pandas numpy

# 개발 의존성으로 추가
poetry add pytest black isort --dev

# 패키지 제거
poetry remove requests
```

### 2. requirements.txt 변환

#### requirements.txt 생성
```sh
# 기본 생성
poetry export --without-hashes --format=requirements.txt > requirements.txt

# 개발 의존성 포함
poetry export --without-hashes --format=requirements.txt --dev > requirements.txt
```

#### requirements.txt에서 의존성 가져오기
```sh
poetry add $(cat requirements.txt)
```

## VSCode 통합

### 1. Python 익스텐션 설치
Command + Shift + X를 눌러 익스텐션 마켓플레이스를 열고 'Python'을 설치한다.

### 2. 프로젝트 설정
`.vscode/settings.json` 파일을 생성하고 다음 내용을 추가한다:
```json
{
    "python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python",
    "python.formatting.provider": "black",
    "python.linting.enabled": true,
    "python.linting.pylintEnabled": true
}
```

### 3. 인터프리터 선택
1. Command + Shift + P를 눌러 명령 팔레트를 연다
2. "Python: Select Interpreter" 입력
3. `.venv/bin/python`을 선택한다

## 유용한 팁

### 1. PATH 설정
Poetry가 설치되었지만 명령어가 인식되지 않는 경우 `~/.zshrc` 또는 `~/.bash_profile`에 추가:
```sh
export PATH="$HOME/.local/bin:$PATH"
```

### 2. pyenv와 함께 사용
macOS에서 여러 Python 버전을 관리하려면 pyenv와 함께 사용하는 것이 좋다:
```sh
# pyenv 설치
brew install pyenv

# Python 버전 설치
pyenv install 3.9.9

# 프로젝트에서 사용할 Python 버전 지정
pyenv local 3.9.9

# Poetry가 pyenv의 Python을 사용하도록 설정
poetry env use $(pyenv which python)
```

### 3. 자동 완성 설정
zsh에서 Poetry 명령어 자동 완성을 활성화:
```sh
# ~/.zshrc에 추가
poetry completions zsh > ~/.zfunc/_poetry
echo 'fpath+=~/.zfunc' >> ~/.zshrc
echo 'autoload -Uz compinit && compinit' >> ~/.zshrc
```

## 문제 해결

### 1. 가상환경 재설정
```sh
# 가상환경 삭제
poetry env remove python

# 캐시 정리
poetry cache clear . --all

# 의존성 재설치
poetry install
```

### 2. 권한 문제 해결
```sh
# Poetry 캐시 디렉토리 권한 수정
sudo chown -R $(whoami) ~/Library/Caches/pypoetry
```
