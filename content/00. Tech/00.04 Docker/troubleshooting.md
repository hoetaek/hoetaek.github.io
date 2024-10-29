---
title: 무제 파일
publish: false
tags:
---
# variable not set
`docker compose up`을 하면 다음과 같은 에러가 뜬다

`WARN[0000] The "fjoo" variable is not set. Defaulting to a blank string.`

> [!info]
> 보통 .env에 있는 값에 문제가 있는 것이다
> 이상한 문자 때문에 중간에 있는 값이 환경변수가 아니라 문자로 인식이 되고 있다
