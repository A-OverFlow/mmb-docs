# 무물보 API 문서 (예제)

## 1. 공통

### 1.1. 에러 코드

* HTTP Error Code가 아닌 Custom Error Code 입니다.

|Error Code|Description|
|------|---|
|1000|잘못된 argument|
|1001|데이터 없음|
|1002|알수없는 에러|

---

## 2. 서비스 이름

### 2.0. API 리스트

|HTTP Method|URL|비고|
|------|---|---|
|Method|https://url:port/path|예시|
|POST|https://mumulbo.com/api/v1/member|회원가입|
|GET|https://mumulbo.com/api/v1/member|회원조회|
|POST|https://mumulbo.com/api/v1/login|로그인|
|PUT|https://mumulbo.com/api/v1/login|회원정보변경|

---

### 2.1. API 이름

|HTTP Method|URL|비고|
|------|---|---|
|Method|https://url:port/path|예시|

### 2.1.1. Reqeust

#### 2.1.1.1. Header

|Key|Value|Required|Description|
|------|---|---|---|
|Authorization|Bearer <JWT>|O|토큰|

#### 2.1.1.2. Path Variables

|Key|Description|
|------|---|
|userId|joonsub.lim|

#### 2.1.1.3. Params

|Key|Value|Required|Description|
|------|---|---|---|
|user_id|joonsub.lim|O|사용자 아이디|

#### 2.2.4. Body

|Key|Value|Required|Description|
|------|---|---|---|
|user_id|joonsub.lim|O|사용자 아이디|
|username|임준섭|O|사용자 이름|
|email|joonsub.lim@ahnlab.com|O|이메일|

### 2.1.2. Response

#### 2.1.2.1. Header

|Key|Value|Description|
|------|---|---|
|X-RateLimit-Limit|500|분당 최대 요청량|
|X-RateLimit-Remaining|499|남은 요청량|

#### 2.1.2.2. Body

|Key|Value|Description|
|------|---|---|
|user_id|joonsub.lim|사용자 아이디|
|username|임준섭|사용자 이름|
|email|joonsub.lim@ahnlab.com|이메일|

### 2.1.3. Syntax

#### 2.1.3.1 Request Syntax

``` json
POST https://mumulbo.com/api/v1/member
Content-Type: application/json

{
    "username": "임준섭",
    "email": "joonsub.lim@ahnlab.com",
    "age": 30,
}
```

#### 2.1.3.2. Response Syntax

``` json
{
    "message": "ok"
}
```

---

### 2.2. API 이름

|HTTP Method|URL|비고|
|------|---|---|
|POST|https://mumulbo.com/api/v1/member|회원가입|

### 2.2.1. Request

#### 2.2.1.1. Header

<반복>