---
date: 2024-11-13
publish: false
tags:
---
> [!info]
> [[개발자를 위한 실용적인 CLI 도구 예시 모음]]
# CLI 도구 환경 구성하기

macOS와 zsh 환경에서 커스텀 CLI 도구를 만들고 관리하는 방법을 알아보자.

## 기본 환경 설정

### 1. 디렉토리 구조 설정

```bash
# CLI 도구를 위한 기본 디렉토리 구조 생성
mkdir -p ~/.local/bin          # 실행 파일들이 위치할 디렉토리
mkdir -p ~/.local/scripts      # 스크립트 소스 코드
mkdir -p ~/.local/completions  # zsh completion 스크립트
```

### 2. zsh 설정 파일 구성

```bash
# ~/.zshrc 파일에 추가할 내용

# PATH 설정
export PATH="$HOME/.local/bin:$PATH"

# 자동 완성 설정
fpath=(~/.local/completions $fpath)
autoload -Uz compinit && compinit

# 별칭 및 함수를 별도 파일로 관리
source ~/.local/scripts/aliases.sh
source ~/.local/scripts/functions.sh
```

## 스크립트 관리 시스템 구축

### 1. 스크립트 템플릿 시스템

```bash
# ~/.local/scripts/template.sh
#!/bin/bash

# 기본 템플릿 생성 스크립트
create_script() {
    local script_name=$1
    local script_path="$HOME/.local/bin/$script_name"
    
    # 스크립트 생성
    cat > "$script_path" << 'EOF'
#!/bin/bash

VERSION="1.0.0"
SCRIPT_NAME="$script_name"

# 도움말
show_help() {
    cat << EOF
사용법: $SCRIPT_NAME [options]

옵션:
    -h, --help      도움말 표시
    -v, --version   버전 정보 표시
EOF
}

# 메인 로직
main() {
    case "$1" in
        -h|--help)
            show_help
            ;;
        -v|--version)
            echo "$SCRIPT_NAME version $VERSION"
            ;;
        *)
            echo "명령어를 실행합니다."
            ;;
    esac
}

main "$@"
EOF

    # 실행 권한 부여
    chmod +x "$script_path"
    echo "스크립트가 생성되었습니다: $script_path"
}
```

### 2. zsh 자동 완성 설정

```bash
# ~/.local/completions/_example
#compdef example

_example() {
    local -a commands
    commands=(
        'start:시작 명령어'
        'stop:중지 명령어'
        'restart:재시작 명령어'
    )

    _describe 'command' commands
}

_example "$@"
```

## 실용적인 CLI 도구 예시

### 1. 프로젝트 생성 도구

```bash
# ~/.local/bin/create-project
#!/bin/bash

# 프로젝트 템플릿 생성 도구
PROJECT_TYPES=("node" "python" "react" "go")

create_node_project() {
    npm init -y
    echo "node_modules/" > .gitignore
    mkdir src test
    # 기본 설정 파일 생성
}

create_python_project() {
    python -m venv venv
    echo "venv/" > .gitignore
    mkdir src tests
    # requirements.txt 등 생성
}

show_help() {
    echo "사용법: create-project <프로젝트타입> <프로젝트이름>"
    echo "지원 타입: ${PROJECT_TYPES[*]}"
}

main() {
    local type=$1
    local name=$2

    if [[ ! " ${PROJECT_TYPES[@]} " =~ " ${type} " ]]; then
        show_help
        exit 1
    fi

    mkdir -p "$name"
    cd "$name"

    case $type in
        "node")
            create_node_project
            ;;
        "python")
            create_python_project
            ;;
    esac
}

main "$@"
```

### 2. 개발 환경 관리 도구

```bash
# ~/.local/bin/dev
#!/bin/bash

# 개발 환경 관리 도구
case $1 in
    "docker")
        case $2 in
            "start")
                docker-compose up -d
                ;;
            "stop")
                docker-compose down
                ;;
        esac
        ;;
    "db")
        case $2 in
            "migrate")
                # 데이터베이스 마이그레이션
                ;;
            "reset")
                # 데이터베이스 초기화
                ;;
        esac
        ;;
esac
```

## 고급 기능 구현

### 1. 로깅 시스템

```bash
# ~/.local/scripts/logger.sh
#!/bin/bash

# 로그 레벨 설정
LOG_LEVEL=${LOG_LEVEL:-"INFO"}

log() {
    local level=$1
    shift
    local message=$@
    local timestamp=$(date '+%Y-%m-%d %H:%M:%S')
    
    case $LOG_LEVEL in
        "DEBUG")
            [[ "$level" =~ ^(DEBUG|INFO|WARN|ERROR)$ ]] && echo "[$timestamp] $level: $message"
            ;;
        "INFO")
            [[ "$level" =~ ^(INFO|WARN|ERROR)$ ]] && echo "[$timestamp] $level: $message"
            ;;
        "WARN")
            [[ "$level" =~ ^(WARN|ERROR)$ ]] && echo "[$timestamp] $level: $message"
            ;;
        "ERROR")
            [[ "$level" =~ ^(ERROR)$ ]] && echo "[$timestamp] $level: $message"
            ;;
    esac
}
```

### 2. 설정 관리

```bash
# ~/.local/scripts/config.sh
#!/bin/bash

# 설정 파일 관리
CONFIG_DIR="$HOME/.config/scripts"
mkdir -p "$CONFIG_DIR"

load_config() {
    local script_name=$1
    local config_file="$CONFIG_DIR/$script_name.conf"
    
    if [[ -f "$config_file" ]]; then
        source "$config_file"
    fi
}

save_config() {
    local script_name=$1
    local config_file="$CONFIG_DIR/$script_name.conf"
    
    # 설정 저장 로직
    declare -p | grep "^declare -x CONFIG_" > "$config_file"
}
```

## 프로젝트 구조 예시

```
~/.local/
├── bin/                    # 실행 파일
│   ├── create-project
│   ├── dev
│   └── other-scripts
├── scripts/               # 공통 스크립트
│   ├── aliases.sh
│   ├── functions.sh
│   ├── logger.sh
│   └── config.sh
├── completions/          # 자동 완성 스크립트
│   ├── _create-project
│   └── _dev
└── tests/               # 테스트 파일들
    └── test_scripts.sh
```

# 유용한 macOS 특화 기능

## 1. Spotlight 통합

```bash
# ~/.local/bin/spotlight-search
#!/bin/bash

# Spotlight 검색 CLI 도구
mdfind -name "$1" | while read -r file; do
    echo "$file"
done
```

## 2. macOS 알림 통합

```bash
# ~/.local/scripts/notify.sh
#!/bin/bash

notify() {
    local title=$1
    local message=$2
    
    osascript -e "display notification \"$message\" with title \"$title\""
}
```

# 설치 및 업데이트 관리

## 1. 설치 스크립트

```bash
# ~/.local/scripts/install.sh
#!/bin/bash

# 필요한 디렉토리 생성
mkdir -p ~/.local/{bin,scripts,completions,tests}

# zsh 설정 추가
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
echo 'fpath=(~/.local/completions $fpath)' >> ~/.zshrc
echo 'autoload -Uz compinit && compinit' >> ~/.zshrc

# 기본 스크립트 복사
cp ~/.local/scripts/{logger,config,notify}.sh ~/.local/scripts/
```

## 2. 업데이트 관리

```bash
# ~/.local/bin/update-scripts
#!/bin/bash

# 스크립트 업데이트 확인 및 적용
VERSION_FILE="$HOME/.local/scripts/version"
CURRENT_VERSION=$(cat "$VERSION_FILE")

check_updates() {
    # 업데이트 확인 로직
    # git repository나 원격 서버에서 확인
    :
}

backup_scripts() {
    local backup_dir="$HOME/.local/backups/$(date +%Y%m%d)"
    mkdir -p "$backup_dir"
    cp -r ~/.local/{bin,scripts,completions} "$backup_dir/"
}

main() {
    check_updates
    backup_scripts
    # 업데이트 적용
}

main "$@"
```

# 주의사항

1. **권한 관리**
   - 모든 스크립트에 실행 권한 부여 필요
   - 중요한 스크립트는 접근 권한 제한

2. **의존성 관리**
   - 외부 도구 의존성 최소화
   - 필요한 경우 설치 여부 확인 로직 추가

3. **버전 관리**
   - Git 등으로 버전 관리 권장
   - 정기적인 백업 필요

# 결론

macOS와 zsh 환경에서 커스텀 CLI 도구를 만들고 관리하는 것은 개발 생산성을 크게 향상시킬 수 있다. 잘 구성된 디렉토리 구조와 공통 기능을 활용하면, 일관성 있고 관리하기 쉬운 CLI 도구를 만들 수 있다. 시작은 작은 스크립트부터, 점진적으로 기능을 확장해 나가는 것을 추천한다.