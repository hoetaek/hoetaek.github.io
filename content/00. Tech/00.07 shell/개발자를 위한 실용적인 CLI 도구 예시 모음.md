---
date: 2024-11-13
publish: false
tags:
---
# 실제 사용 사례로 알아보는 CLI 도구

개발자 개인이 일상적으로 마주하는 반복 작업들을 자동화하는 실용적인 예시들을 살펴보자.

# 1. 프로젝트 관리 도구

## 작업 브랜치 생성 자동화

```bash
# ~/.local/bin/ticket
#!/bin/bash

# Jira 티켓 번호로 브랜치 생성하고 필요한 설정까지 자동화
create_branch() {
    local ticket_no=$1
    local branch_type=${2:-"feature"}  # feature/fix/hotfix
    local branch_name="$branch_type/$ticket_no"
    
    # 최신 develop 브랜치 가져오기
    git checkout develop
    git pull origin develop
    
    # 브랜치 생성 및 이동
    git checkout -b "$branch_name"
    
    # 자주 사용하는 파일 자동 생성
    if [[ "$branch_type" == "feature" ]]; then
        touch "src/features/$ticket_no.ts"
        touch "test/features/$ticket_no.test.ts"
    fi
    
    echo "🎉 Created and switched to branch: $branch_name"
}

# 사용법: ticket PROJ-123 [feature|fix|hotfix]
create_branch "$1" "$2"
```

## 프로젝트 생성 자동화

```bash
# ~/.local/bin/create-next
#!/bin/bash

# Next.js 프로젝트 생성 및 자주 사용하는 설정 자동화
create_next_project() {
    local project_name=$1
    
    # Next.js 프로젝트 생성
    npx create-next-app@latest $project_name --typescript --tailwind --eslint
    cd $project_name
    
    # 자주 사용하는 패키지 설치
    npm install @tanstack/react-query axios dayjs
    
    # 프로젝트 구조 생성
    mkdir -p src/{components,hooks,utils,types,apis,constants}
    
    # .env 파일 생성
    cat > .env.local << EOF
NEXT_PUBLIC_API_URL=http://localhost:3000/api
EOF
    
    # types/index.ts 생성
    cat > src/types/index.ts << EOF
export interface ApiResponse<T> {
  data: T;
  status: number;
  message: string;
}
EOF
    
    echo "🚀 Project setup complete!"
}

create_next_project "$1"
```

# 2. 개발 작업 자동화

## API 테스트 도구

```bash
# ~/.local/bin/api
#!/bin/bash

# 자주 사용하는 API 엔드포인트 테스트 자동화
CONFIG_FILE="$HOME/.config/api-test/config.json"

test_api() {
    local endpoint=$1
    local method=${2:-"GET"}
    local data=$3
    
    # 설정된 base URL 가져오기
    local base_url=$(jq -r '.base_url' "$CONFIG_FILE")
    local token=$(jq -r '.token' "$CONFIG_FILE")
    
    # curl 요청 보내기
    curl -X "$method" \
         -H "Authorization: Bearer $token" \
         -H "Content-Type: application/json" \
         -d "$data" \
         "$base_url$endpoint" | jq .
}

# 사용법: api /users GET
# 사용법: api /users/1 POST '{"name": "John"}'
test_api "$@"
```

## 데이터베이스 작업 자동화

```bash
# ~/.local/bin/db
#!/bin/bash

# 데이터베이스 관련 작업 자동화
DB_CONFIG="$HOME/.config/db/config.json"

case $1 in
    "backup")
        # 로컬 DB 백업
        pg_dump -U postgres my_db > "backup_$(date +%Y%m%d).sql"
        ;;
    "restore")
        # 백업 복원
        psql -U postgres my_db < "$2"
        ;;
    "reset")
        # 개발 DB 초기화
        psql -U postgres -c "DROP DATABASE my_db"
        psql -U postgres -c "CREATE DATABASE my_db"
        ;;
esac
```

# 3. 생산성 도구

## 문서 템플릿 생성기

```bash
# ~/.local/bin/readme
#!/bin/bash

# README.md 템플릿 생성
create_readme() {
    local project_name=${1:-$(basename $(pwd))}
    
    cat > README.md << EOF
# $project_name

## 개요

## 시작하기

### 필수 조건

### 설치 방법

\`\`\`bash
npm install
\`\`\`

### 개발 서버 실행

\`\`\`bash
npm run dev
\`\`\`

## 테스트

\`\`\`bash
npm test
\`\`\`

## 배포

## 기술 스택

## 프로젝트 구조

## 기여 방법

## 라이선스
EOF

    echo "📝 README.md created!"
}

create_readme "$1"
```

## 작업 환경 설정 자동화

```bash
# ~/.local/bin/workspace
#!/bin/bash

# 작업 환경 설정 자동화 (터미널, 앱 실행 등)
setup_workspace() {
    local project_path=$1
    
    # VS Code 실행
    code "$project_path"
    
    # 터미널 분할 및 명령어 실행
    osascript <<EOF
tell application "iTerm"
    tell current window
        create tab with default profile
        tell current session
            write text "cd $project_path"
            write text "npm run dev"
        end tell
        create tab with default profile
        tell current session
            write text "cd $project_path"
            write text "git status"
        end tell
    end tell
end tell
EOF

    # Docker 컨테이너 실행
    cd "$project_path" && docker-compose up -d
}

# 사용법: workspace ~/projects/my-app
setup_workspace "$1"
```

# 4. 일상적인 개발 작업 자동화

## PR 준비 도구

```bash
# ~/.local/bin/pr
#!/bin/bash

# PR 준비 작업 자동화
prepare_pr() {
    # 현재 브랜치 이름 가져오기
    local current_branch=$(git branch --show-current)
    
    # develop 브랜치 최신화
    git fetch origin develop
    
    # 충돌 확인
    git merge-base --is-ancestor origin/develop HEAD || {
        echo "⚠️ 충돌이 있을 수 있습니다. develop 브랜치를 먼저 merge 하세요."
        exit 1
    }
    
    # lint 검사
    npm run lint
    
    # 테스트 실행
    npm run test
    
    # 변경된 파일 목록 출력
    echo "📝 변경된 파일:"
    git diff --name-only develop
    
    # PR 템플릿 생성
    cat > .github/pull_request_template.md << EOF
## 변경 사항
- 

## 테스트
- [ ] 단위 테스트 완료
- [ ] E2E 테스트 완료

## 스크린샷

## 관련 이슈
- $current_branch
EOF
}

prepare_pr
```

## 로그 분석 도구

```bash
# ~/.local/bin/logs
#!/bin/bash

# 로그 파일 분석 도구
analyze_logs() {
    local log_file=$1
    local search_term=$2
    
    case $search_term in
        "error")
            # 에러 로그만 추출
            grep -i "error" "$log_file" | tail -n 50
            ;;
        "slow")
            # 느린 쿼리 추출
            grep "duration" "$log_file" | awk '$NF > 1000'
            ;;
        "auth")
            # 인증 관련 로그 추출
            grep -i "auth" "$log_file" | tail -n 50
            ;;
    esac
}

# 사용법: logs app.log error
analyze_logs "$@"
```

# 설정 예시

## zsh 별칭 설정

```bash
# ~/.local/scripts/aliases.sh

# Git 관련
alias gs="git status"
alias gp="git push origin HEAD"
alias gpl="git pull origin HEAD"
alias nah="git reset --hard && git clean -df"

# Docker 관련
alias dc="docker-compose"
alias dcu="docker-compose up -d"
alias dcd="docker-compose down"
alias dcl="docker-compose logs -f"

# NPM 관련
alias ni="npm install"
alias nr="npm run"
alias nrd="npm run dev"
alias nrb="npm run build"
```

## 자주 사용하는 함수

```bash
# ~/.local/scripts/functions.sh

# Node.js 버전 변경
nvm-switch() {
    local version=$1
    export NVM_DIR="$HOME/.nvm"
    [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
    nvm use $version
}

# GitHub PR 페이지 열기
pr-open() {
    local repo_url=$(git remote get-url origin)
    local branch=$(git branch --show-current)
    open "${repo_url/git@github.com:/https://github.com/}/compare/develop...$branch"
}

# 포트 kill
killport() {
    lsof -i tcp:$1 | awk 'NR!=1 {print $2}' | xargs kill -9
}
```

# 실용적인 팁

1. **자주 사용하는 명령어 분석하기**
   ```bash
   # 히스토리에서 가장 많이 사용한 명령어 추출
   history | awk '{print $2}' | sort | uniq -c | sort -rn | head -n 10
   ```

2. **프로젝트별 설정 관리**
   ```bash
   # .envrc 파일 사용 (direnv 설치 필요)
   echo 'export PROJECT_API_KEY="xxx"' > .envrc
   direnv allow
   ```

3. **자동 업데이트 확인**
   ```bash
   # ~/.zshrc에 추가
   if [ $(date +%w) -eq 1 ]; then  # 월요일마다 체크
      update-scripts --check
   fi
   ```

# 실용적인 시나리오

1. **새 프로젝트 시작 시**
   ```bash
   # 한 번의 명령어로 모든 설정 완료
   workspace-init my-project
   cd my-project
   readme  # README.md 생성
   create-next  # Next.js 프로젝트 설정
   git init
   ticket PROJ-123 feature  # 작업 브랜치 생성
   ```

2. **일일 개발 시작 시**
   ```bash
   workspace ~/projects/current-project  # 작업 환경 설정
   gpl  # 최신 코드 가져오기
   nrd  # 개발 서버 실행
   ```

3. **PR 생성 전**
   ```bash
   pr  # PR 준비 작업 수행
   pr-open  # GitHub PR 페이지 열기
   ```

이러한 도구들은 개발자 개인의 생산성을 크게 향상시킬 수 있습니다. 처음에는 간단한 스크립트부터 시작하여, 필요에 따라 점진적으로 기능을 추가하는 것을 추천합니다. 또한, 이러한 스크립트들을 Git 저장소에서 관리하면 다른 컴퓨터에서도 동일한 환경을 쉽게 구성할 수 있습니다.