---
date: 2024-11-28
publish: false
tags:
---
# 파이프라인 변수의 이해

소프트웨어 개발에서 레시피를 생각해보자. 같은 요리법이라도 재료의 양이나 조리 시간을 조절하여 다양한 결과물을 만들 수 있는 것처럼, AWS CodePipeline의 변수는 동일한 파이프라인 구조를 유지하면서 다양한 환경과 상황에 맞춰 유연하게 배포 프로세스를 조정할 수 있게 해준다.

## 변수의 종류

CodePipeline의 변수는 크게 두 가지 카테고리로 나눌 수 있다:

1. 정의된 변수(Defined Variables)
   - 사전에 정의되어 있어 이름이나 값을 변경할 수 없다
   - 예: GitHub 소스 액션의 CommitId, BranchName 등
   - 파이프라인 전역 변수인 ExecutionId도 여기에 포함된다

2. 사용자 구성 변수(User-Configured Variables)
   - CloudFormation, CodeBuild, Lambda 액션에서만 사용 가능
   - 사용자가 직접 키와 값을 정의할 수 있다

# 주요 액션별 사용 가능한 변수

## 1. CloudFormation 변수
CloudFormation의 경우 다음 액션에서만 변수를 출력할 수 있다:
- CREATE_UPDATE
- REPLACE_ON_FAILURE
- ALWAYS_REPLACE
- CHANGE_SET_EXECUTE

```yaml
Outputs:
  Version:
    Value: "12"
    Description: "Application version number"
```

## 2. CodeBuild 변수
buildspec.yml에서 환경 변수를 export하여 파이프라인에서 사용할 수 있다:

```yaml
env:
  exported-variables:
    - BUILD_VERSION
    - DEPLOY_ENV
phases:
  build:
    commands:
      - export BUILD_VERSION="1.0.0"
      - export DEPLOY_ENV="production"
```

## 3. Lambda 변수
Lambda 함수에서 putJobSuccessResult API 호출 시 변수를 정의할 수 있다:

```javascript
const AWS = require('aws-sdk');
const codepipeline = new AWS.CodePipeline();

exports.handler = async (event) => {
  await codepipeline.putJobSuccessResult({
    jobId: event.CodePipeline.job.id,
    outputVariables: {
      MY_VARIABLE: "custom value",
      ENVIRONMENT: "production"
    }
  }).promise();
};
```

# 변수 사용을 위한 실전 가이드

## 1. 변수 네임스페이스 설정
변수를 출력하는 액션에서는 반드시 Variable namespace를 설정해야 한다. 이는 UI에서 액션 설정 시 하단에서 설정할 수 있다.

## 2. 변수 참조 구문
다운스트림 액션에서 변수를 사용할 때는 다음 구문을 사용한다:
```
#{네임스페이스.변수키}
```

예시:
- 소스 변수 참조: `#{SourceVariables.CommitId}`
- CloudFormation 출력 참조: `#{CFNOutput.StackName}`

## 3. 실제 사용 예시

### GitHub 소스 변수를 CodeBuild에서 사용
CodeBuild 액션의 환경 변수 설정:
```yaml
Environment Variables:
  Name: BRANCH_NAME
  Value: #{SourceVariables.BranchName}
```

### 수동 승인 단계에서 동적 URL 생성
```
Review URL: https://github.com/#{SourceVariables.RepositoryName}/commit/#{SourceVariables.CommitId}
```

# 변수 검증 및 디버깅

## 1. 변수 출력 확인 방법
1. 파이프라인 실행 기록(History) 확인
2. 특정 실행(Execution) 선택
3. 원하는 액션 선택
4. 'Output Variables' 섹션에서 사용 가능한 변수 목록 확인

## 2. 일반적인 문제 해결
- 네임스페이스가 설정되어 있는지 확인
- 변수 참조 구문이 정확한지 확인
- 액션 순서가 올바른지 확인 (상위 액션의 변수만 사용 가능)

# 보안 고려사항

1. 민감한 정보는 변수로 직접 노출하지 않기
2. 환경별 변수 접근 권한 설정
3. 변수 값의 정기적인 감사

# 결론

CodePipeline의 변수 시스템을 효과적으로 활용하면 다음과 같은 이점을 얻을 수 있다:

1. 동적이고 유연한 파이프라인 구성
2. 환경별 설정 관리 용이
3. 자동화된 배포 프로세스 구현
4. 디버깅과 모니터링 효율화

주의할 점은 모든 액션이 변수를 생성할 수 있는 것은 아니지만, 모든 액션에서 변수를 사용할 수 있다는 것이다. 따라서 파이프라인 설계 시 변수의 흐름을 신중하게 고려해야 한다.