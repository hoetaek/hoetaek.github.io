---
title: AWS
publish: true
tags:
---
# Cloudwatch
## cw agent
### logs
파일 위치
`/opt/aws/amazon-cloudwatch-agent/logs`

로그 읽는 법
```sh
tail -f /opt/aws/amazon-cloudwatch-agent/logs/amazon-cloudwatch-agent.log
```

# Cloudfront
## cache key
CloudFront의 캐시 키(Cache Key)는 캐시에서 특정 객체를 식별하는 데 사용되는 고유 식별자이다. 캐시 키를 구성하는 요소는 요청 URL, HTTP 메서드, 헤더, 쿠키, 쿼리 문자열 파라미터 등이 있다. 캐시 키는 CloudFront가 어떤 버전의 객체를 제공할지 결정하는 데 중요한 역할을 한다.

기본적으로 CloudFront의 캐시 키는 다음과 같은 요소로 구성되어 있다:
1. **도메인 이름**: 요청이 온 도메인 이름.
2. **경로**: 요청된 URL의 경로 부분.
3. **쿼리 문자열**: URL에 포함된 쿼리 문자열 파라미터.

예를 들어, https://example.com/images/picture.jpg?size=large라는 URL 요청이 있다면, 기본 캐시 키는 example.com/images/picture.jpg?size=large가 된다.

하지만, CloudFront에서는 캐시 키를 사용자 지정할 수 있다. 이를 통해 특정 헤더, 쿠키 또는 쿼리 문자열 파라미터를 포함하거나 제외하여 캐시 효율성을 최적화할 수 있다.
### 사용자 지정 캐시 키 구성 요소

1. **쿼리 문자열(Query String Parameters)**:
	• 특정 쿼리 문자열 파라미터를 포함하거나 제외할 수 있다.
	• 예: size 파라미터만 캐시 키에 포함하려면 해당 파라미터를 지정할 수 있다.
2. **헤더(Headers)**:
	• 특정 HTTP 헤더를 캐시 키에 포함하거나 제외할 수 있다.
	• 예: User-Agent 헤더를 포함하여 캐시 키를 생성하면, 브라우저 유형별로 다른 캐시 객체를 유지할 수 있다.
3. **쿠키(Cookies)**:
	• 특정 쿠키를 캐시 키에 포함하거나 제외할 수 있다.
	• 예: session-id 쿠키를 캐시 키에 포함하여 사용자별로 다른 캐시 객체를 유지할 수 있다.
4. **프로토콜(Protocol)**:
	• HTTP와 HTTPS 요청을 구분하여 캐시 키를 생성할 수 있다.
	• 예: HTTP 요청과 HTTPS 요청을 별도의 캐시 키로 처리하여 각 프로토콜별로 캐시 객체를 유지할 수 있다.

> [!important] s3의 파일 이름 설정하여 받기
> cache key에 별도로 설정하지 않으면 모든 query string은 무시된다
> 따라서 s3의 응답 헤더를 설정하는 response-content-disposition을 s3에 전달하려면 해당 값을 cache key의 query parameter에 설정해야 한다

# S3
## Header 설정

S3에서 특정 쿼리 문자열 파라미터를 통해 HTTP 헤더에 영향을 미치는 경우가 있다. 이러한 쿼리 문자열 파라미터는 요청 시 지정되며, 응답의 HTTP 헤더에 반영된다. 대표적인 예로 response-content-disposition을 포함한 몇 가지 파라미터가 있다. 이들 파라미터를 통해 S3 객체를 요청할 때 응답의 HTTP 헤더를 제어할 수 있다. 주요 파라미터와 그 역할은 다음과 같다:

1. **response-content-type**:
	• 이 파라미터를 사용하여 응답의 Content-Type 헤더를 지정할 수 있다.
	• 예: https://bucket-name.s3.amazonaws.com/object-key?response-content-type=application/pdf
	• 위의 URL을 통해 요청하면 응답의 Content-Type 헤더는 application/pdf로 설정된다.

2. **response-content-language**:
	• 이 파라미터를 사용하여 응답의 Content-Language 헤더를 지정할 수 있다.
	• 예: https://bucket-name.s3.amazonaws.com/object-key?response-content-language=en-US
	• 위의 URL을 통해 요청하면 응답의 Content-Language 헤더는 en-US로 설정된다.

3. **response-expires**:
	• 이 파라미터를 사용하여 응답의 Expires 헤더를 지정할 수 있다.
	• 예: https://bucket-name.s3.amazonaws.com/object-key?response-expires=Wed%2C%2021%20Oct%202020%2007%3A28%3A00%20GMT
	• 위의 URL을 통해 요청하면 응답의 Expires 헤더는 Wed, 21 Oct 2020 07:28:00 GMT로 설정된다.

4. **response-cache-control**:
	• 이 파라미터를 사용하여 응답의 Cache-Control 헤더를 지정할 수 있다.
	• 예: https://bucket-name.s3.amazonaws.com/object-key?response-cache-control=no-cache
	• 위의 URL을 통해 요청하면 응답의 Cache-Control 헤더는 no-cache로 설정된다.

5. **response-content-disposition**:
	• 이 파라미터를 사용하여 응답의 Content-Disposition 헤더를 지정할 수 있다. 이는 브라우저에서 파일을 다운로드할 때 사용자가 볼 수 있는 파일 이름을 지정하는 데 유용하다.
	• 예: https://bucket-name.s3.amazonaws.com/object-key?response-content-disposition=attachment%3B%20filename%3D%22example.pdf%22
	• 위의 URL을 통해 요청하면 응답의 Content-Disposition 헤더는 attachment; filename="example.pdf"로 설정된다.

6. **response-content-encoding**:
	• 이 파라미터를 사용하여 응답의 Content-Encoding 헤더를 지정할 수 있다.
	• 예: https://bucket-name.s3.amazonaws.com/object-key?response-content-encoding=gzip
	• 위의 URL을 통해 요청하면 응답의 Content-Encoding 헤더는 gzip으로 설정된다.
	