---
date: 2024-11-13
publish: false
tags:
---
# zsh 스크립트 기본 구조

## 기본 형식

```zsh
#!/bin/zsh

# 스크립트 설명
# 작성자: 
# 최종 수정일: 

# 옵션 설정
setopt EXTENDED_GLOB  # 확장 글로브 패턴 사용
setopt ERR_EXIT      # 오류 발생 시 스크립트 종료
setopt PIPE_FAIL     # 파이프의 모든 명령어 상태 확인

# 변수 선언
local_var="hello"    # 로컬 변수
export GLOBAL="world"  # 환경 변수
```

# 변수와 데이터 타입

## 1. 변수 선언과 사용

```zsh
# 기본 변수 선언
name="John"
age=25

# 변수 사용
echo $name         # 기본 사용
echo ${name}       # 중괄호 사용 (추천)
echo "${name}"     # 따옴표 사용 (공백이 있을 때 추천)

# readonly 변수
readonly MAX_COUNT=100

# 배열 선언
fruits=(apple banana orange)
echo ${fruits[1]}     # apple (zsh는 1부터 인덱싱)
echo ${fruits[*]}     # 모든 요소 출력
echo ${#fruits[@]}    # 배열 길이

# 연관 배열 (해시)
declare -A user
user[name]="John"
user[age]=25
echo ${user[name]}    # John
```

## 2. 특수 변수

```zsh
echo $0        # 스크립트 이름
echo $1        # 첫 번째 인자
echo $@        # 모든 인자 (배열)
echo $#        # 인자 개수
echo $?        # 이전 명령어의 종료 상태
echo $$        # 현재 프로세스 ID
echo $!        # 마지막 백그라운드 프로세스 ID
echo $PWD      # 현재 작업 디렉토리
echo $RANDOM   # 랜덤 숫자
```

# 조건문과 반복문

## 1. if 문

```zsh
# 기본 if 문
if [[ ${age} -gt 18 ]]; then
    echo "성인입니다"
elif [[ ${age} -gt 13 ]]; then
    echo "청소년입니다"
else
    echo "어린이입니다"
fi

# 문자열 비교
if [[ "${name}" = "John" ]]; then
    echo "Hello John!"
fi

# 파일 테스트
if [[ -f "file.txt" ]]; then
    echo "파일이 존재합니다"
fi

# 다중 조건
if [[ -f "file.txt" && ${age} -gt 18 ]]; then
    echo "조건이 모두 참입니다"
fi
```

## 2. case 문

```zsh
case ${fruit} in
    "apple")
        echo "사과입니다"
        ;;
    "banana"|"orange")
        echo "바나나 또는 오렌지입니다"
        ;;
    *)
        echo "기타 과일입니다"
        ;;
esac
```

## 3. 반복문

```zsh
# for 반복문
for i in {1..5}; do
    echo $i
done

# 배열 순회
for fruit in ${fruits[@]}; do
    echo ${fruit}
done

# while 반복문
count=1
while [[ ${count} -le 5 ]]; do
    echo ${count}
    ((count++))
done

# until 반복문
count=1
until [[ ${count} -gt 5 ]]; do
    echo ${count}
    ((count++))
done
```

# 함수

## 1. 함수 정의와 사용

```zsh
# 기본 함수
greeting() {
    echo "Hello, ${1}!"
}

# 함수 호출
greeting "John"

# 반환값이 있는 함수
get_sum() {
    local num1=$1
    local num2=$2
    echo $((num1 + num2))
}

result=$(get_sum 5 3)
echo ${result}  # 8

# 함수 내에서 지역 변수 사용
process_data() {
    local temp_var="임시 데이터"
    echo ${temp_var}
}
```

# 입출력 처리

## 1. 사용자 입력 받기

```zsh
# read 명령어 사용
echo "이름을 입력하세요: "
read name

# -p 옵션으로 프롬프트 표시
read -p "나이를 입력하세요: " age

# 비밀번호 입력 (-s 옵션)
read -s -p "비밀번호를 입력하세요: " password
echo  # 줄바꿈

# 여러 값 입력
read -p "이름과 나이를 입력하세요: " name age
```

## 2. 파일 입출력

```zsh
# 파일 읽기
while IFS= read -r line; do
    echo ${line}
done < "input.txt"

# 파일 쓰기
echo "새로운 내용" > output.txt     # 덮어쓰기
echo "추가 내용" >> output.txt      # 추가하기
```

# 오류 처리

```zsh
# 오류 처리 함수
handle_error() {
    echo "에러 발생: $1" >&2
    exit 1
}

# 오류 처리 예시
if ! command -v git &> /dev/null; then
    handle_error "git이 설치되어 있지 않습니다"
fi

# 명령어 실행 결과 확인
if ! git pull; then
    handle_error "git pull 실패"
fi
```

# 유용한 zsh 기능

## 1. 글로브 패턴

```zsh
# 확장 글로브 패턴
setopt EXTENDED_GLOB

# 모든 .txt 파일 (숨김 파일 포함)
echo **.txt

# .git으로 끝나지 않는 디렉토리
echo *~*.git

# 빈 디렉토리 찾기
echo **/*(/)
```

## 2. 매개변수 확장

```zsh
# 기본값 설정
echo ${name:-"기본값"}

# 대문자 변환
echo ${name:u}

# 소문자 변환
echo ${name:l}

# 문자열 길이
echo ${#name}

# 부분 문자열
echo ${name:0:2}
```

# 실전 예제

## 1. 로그 처리 스크립트

```zsh
#!/bin/zsh

# 로그 파일 처리 스크립트
LOG_FILE="app.log"
OUTPUT_DIR="logs"

process_logs() {
    local date=$1
    local pattern=$2
    
    # 출력 디렉토리 생성
    mkdir -p ${OUTPUT_DIR}
    
    # 로그 검색 및 저장
    grep "${date}" "${LOG_FILE}" | \
        grep -i "${pattern}" > "${OUTPUT_DIR}/${date}_${pattern}.log"
    
    # 결과 확인
    if [[ $? -eq 0 ]]; then
        echo "로그 처리 완료"
    else
        echo "로그 처리 실패" >&2
        return 1
    fi
}

# 매개변수 확인
if [[ $# -lt 2 ]]; then
    echo "사용법: $0 날짜 패턴" >&2
    exit 1
fi

# 함수 호출
process_logs "$1" "$2"
```

## 2. 배포 스크립트

```zsh
#!/bin/zsh

# 간단한 배포 스크립트
DEPLOY_DIR="/var/www/app"
BACKUP_DIR="/var/www/backups"

deploy() {
    local version=$1
    local timestamp=$(date +%Y%m%d_%H%M%S)
    
    # 현재 버전 백업
    echo "백업 생성 중..."
    if [[ -d ${DEPLOY_DIR} ]]; then
        cp -r ${DEPLOY_DIR} ${BACKUP_DIR}/backup_${timestamp}
    fi
    
    # 새 버전 배포
    echo "새 버전 배포 중..."
    git pull origin ${version}
    
    # 의존성 설치
    npm install
    
    # 빌드
    npm run build
    
    echo "배포 완료!"
}

# 오류 처리
set -e

# 매개변수 확인
if [[ $# -ne 1 ]]; then
    echo "사용법: $0 버전" >&2
    exit 1
fi

# 배포 실행
deploy "$1"
```

# 스크립트 디버깅

```zsh
# 디버그 모드 활성화
set -x  # 실행되는 명령어 출력
set -e  # 오류 발생 시 즉시 종료
set -v  # 입력 라인 출력

# 특정 섹션만 디버깅
{
    set -x
    # 디버그할 코드
    set +x
}

# 로그 출력 함수
debug() {
    [[ "${DEBUG}" = "true" ]] && echo "DEBUG: $*" >&2
}
```

# 성능 최적화 팁

1. **서브쉘 사용 최소화**
   ```zsh
   # 좋지 않은 예
   for file in $(ls); do
   
   # 좋은 예
   for file in *; do
   ```

2. **적절한 변수 사용**
   ```zsh
   # 지역 변수 사용
   local temp_var="value"
   
   # 참조 사용
   local -n ref=array
   ```

3. **명령어 그룹화**
   ```zsh
   # 파이프 대신 프로세스 치환 사용
   diff <(sort file1) <(sort file2)
   ```

# 결론

zsh 스크립트는 강력한 기능과 편리한 문법을 제공합니다. 이 가이드에서 다룬 기본 문법과 패턴을 기반으로, 실제 업무에서 유용한 자동화 스크립트를 작성할 수 있습니다. 스크립트 작성 시에는 가독성과 유지보수성을 고려하고, 적절한 오류 처리와 문서화를 포함하는 것이 중요합니다.