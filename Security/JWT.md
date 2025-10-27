# JWT (JSON Web Token)

* JSON Web Token의 약자
* 웹 애플리케이션에서 사용자별 아래 3가지 사항을 위해 사용되는 표준 토큰 형식(RFC 7519)
    - **인증(Authentication)**
    - **권한 부여(Authorization)**
    - **정보 교환(Information Exchange)**

<br>
<br>

## 1. 기본 개념
* JWT는 서버와 클라이언트가 신뢰할 수 있는 방식으로 정보를 주고받기 위한 **서명된(signed)** 토큰
* JSON 객체를 Base64 URL-safe로 인코딩하고, 디지털 서명(Signature)을 덧붙여 데이터의 **무결성(Integrity)** 과 **인증(Authentication)** 을 보장.
* Python에서는 다음 라이브러리를 주로 사용.
    - `PyJWT`
    - `python-jose`
    - `Authlib`

<br>

### 1.1. JWT의 핵심 인증 원리

```text
Client → 로그인 요청 → Server → JWT 생성/발급
Client → API 요청 시 Authorization 헤더로 전송
Server → JWT 서명·만료 검증 → 요청 처리 or 401 응답
```

1.  **로그인 요청**: 사용자가 아이디/비밀번호 등으로 로그인 요청.
2.  **자격 증명 확인**: 서버는 사용자의 자격 증명 확인.
3.  **JWT 생성 및 발급**: 인증이 성공하면, 서버는 사용자의 정보(Payload)와 비밀키를 이용해 **서명된 JWT**를 생성하여 클라이언트에 전달.
4.  **API 요청 시 JWT 사용**: 클라이언트는 이후 서버에 API를 요청할 때마다 HTTP `Authorization` 헤더에 JWT를 `Bearer <token>` 형태로 실어 보냄.
5.  **토큰 검증**: 서버는 요청을 받을 때마다 이 토큰을 검증(verify)하여 사용자 신원을 확인하고 요청 처리.
    - 서버는 자신이 발급한 토큰인지 확인하기 위해, 수신된 JWT의 Header와 Payload를 동일한 비밀키로 다시 서명하여 생성된 Signature가 기존 Signature와 일치하는지 검증.
    - 토큰이 위변조되지 않았고, 만료되지 않았다면 요청을 허용.

<br>

### 1.2. JWT의 대표 사용 예
*   **OAuth2 기반 인증 시스템**: Google, Facebook 등 소셜 로그인에서 Access Token 발급 시 사용.
*   **OpenID Connect (OIDC)**: OAuth2 위에서 동작하는 인증 계층으로, ID Token으로 JWT를 사용.
*   **API Gateway 인증**: Microservices Architecture(MSA) 환경에서 API Gateway가 각 서비스에 대한 요청을 인증할 때 사용.
*   **Single Sign-On (SSO)**: 한 번의 로그인으로 여러 연관된 서비스를 이용할 수 있도록 인증 정보를 공유할 때 사용.

<br>
<br>

## 2. JWT의 구조
```text
xxxxx.yyyyy.zzzzz


Header.Payload.Signature
   ↓       ↓          ↓
  alg,typ  user data   서명 (secret key)
```
|구분| 이름| 설명|
| :---: | :--- | :--- |
| xxxxx | **Header**  | 토큰의 유형(JWT)과 서명 알고리즘(`HS256`, `RS256` 등)을 명시.|
| yyyyy | **Payload** | 실제로 담을 정보인 **클레임(Claim)** 세트를 포함. 예: 사용자 ID, 권한, 만료 시간 등.|
| zzzzz | **Signature** | Header와 Payload를 비밀키로 서명한 값. 토큰의 위변조 여부를 확인하는 데 사용.|

<br>

### 2.1. `HS256`과 `RS256`의 차이

|구분|약어|설명|특징|
|---|---|---|---|
|`HS256`|HMAC with SHA-256|**대칭키** 기반 알고리즘|하나의 비밀키를 서명과 검증에 모두 사용하므로, 서버 내에서만 인증할 때 적합|
|`RS256`|RSA with SHA-256|**비대칭키(공개키/개인키)** 기반 알고리즘. 개인키로 서명하고, 공개키로 검증|보안성이 높아 외부 서비스 간 인증(OAuth 등)이나, 서명 주체와 검증 주체가 다를 때 주로 사용|

<br>

### 2.2. 클레임(Claim)
* JWT에 담긴 정보 조각으로 세 가지 종류로 구분

|구분|설명|예|
|---|---|---|
|**등록된 클레임(Registered Claims)**|`iss`(발급자), `exp`(만료 시간), `sub`(주체), `iat`(발급 시간) 등 표준으로 정의된 키||
|**공개 클레임(Public Claims)**|충돌 방지를 위해 URI 형태로 정의된 사용자 지정 키|`"https://example.com/my_claim": true`|
|**비공개 클레임(Private Claims)**|서비스 내부에서만 쓰이는 임의의 키|`user_id`, `role` 등|


<br>

### 2.3. 예시
*   클라이언트는 API 호출 시 다음과 같이 JWT를 헤더에 포함시킵니다:
    ```http
    Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyX2lkIjoxMjMsInJvbGUiOiJhZG1pbiIsImV4cCI6MTczMDI4MTYwMH0.q1y9TzJcIh6UHsV0hrTkQ4W4YJ6Xz8I2-l03t4JX1Xc
    ```
*   위 토큰의 각 부분은 다음과 같이 디코딩될 수 있음.
    ```json
    // Header
    {
        "alg": "HS256",
        "typ": "JWT"
    }

    // Payload
    {
        "user_id": 123,
        "role": "admin",
        "exp": 1730281600 // 만료 시간 (Unix Timestamp)
    }

    // Signature
    // HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
    ```

<br>
<br>

## 3. 토큰의 수명 관리: Access Token vs Refresh Token
JWT는 한 번 발급되면 서버에서 강제로 만료시키기 어려움.
- 만약 Access Token이 탈취되면 유효기간 동안 악의적인 사용이 가능. 
- 이를 보완하기 위해 **Refresh Token**을 함께 사용하는 전략이 일반적.

<br>

### 3.1. Access Token

|구분|내용|
|:---:|---|
|**역할**|실제 API 요청에 대한 인증 및 권한 부여에 사용.|
|**수명**|**짧게** 설정 (예: 15분 ~ 1시간). 탈취되더라도 피해를 최소화하기 위함.|
|**저장 위치**|클라이언트의 메모리(변수)에 저장하는 것이 가장 안전하며, `localStorage`나 `sessionStorage`는 XSS 공격에 취약할 수 있음.|

<br>

### 3.2. Refresh Token

|구분|내용|
|:---:|---|
|**역할**|Access Token이 만료되었을 때, 새로운 Access Token을 발급받기 위해 사용|
|**수명**|**길게** 설정(예: 7일 ~ 30일), 사용자가 자주 로그인해야 하는 불편함을 줄여줌|
|**저장 위치**|**안전한 곳**에 저장. 웹 환경에서는 `HttpOnly`, `Secure`, `SameSite=Strict` 옵션을 적용한 쿠키에 저장하는 것이 권장. 이를 통해 XSS 공격으로 토큰을 탈취하는 것을 방지하고, CSRF(Cross-Site Request Forgery) 공격도 완화할 수 있음. (단, 완벽한 CSRF 방어를 위해서는 Anti-CSRF 토큰 패턴 병행 권장.)|
|**보안**|Refresh Token은 탈취되면 장기간 시스템 접근 권한을 주는 것과 같으므로, 서버는 발급된 Refresh Token을 DB에 저장하고 관리해야 함|

<br>

### 3.3. 토큰 재발급 흐름
1.  클라이언트는 Access Token으로 API를 요청.
2.  서버는 Access Token이 만료되었음을 확인하고 `401 Unauthorized` 오류를 반환.
3.  클라이언트는 이 오류를 감지하고, 보관하고 있던 Refresh Token을 첨부하여 서버의 토큰 재발급 API(` /refresh` 등)에 새로운 Access Token을 요청.
4.  서버는 전달받은 Refresh Token이 유효한지 DB에서 확인한 후, 새로운 Access Token(과 경우에 따라 새로운 Refresh Token)을 발급.
5.  클라이언트는 새로 발급받은 Access Token으로 이전에 실패했던 API 요청을 다시 시도.

*   **예외 처리**: 만료된 토큰(`ExpiredSignatureError`)이나 위조된 토큰(`InvalidSignatureError`)에 대한 명확한 예외 처리가 필수적. Refresh Token마저 만료되었다면, 사용자는 다시 로그인해야 함.

<br>
<br>

## 4. JWT의 장점과 단점
### 4.1. 장점
| 장점| 설명|
| :---: | :--- |
| **Stateless (무상태성)** | 서버가 세션을 저장하지 않아도 되므로, 서버의 부담이 줄고 확장성(Scale-out)에 유리|
| **빠른 인증**| 토큰만 검증하면 되므로, 매번 DB를 조회하여 사용자 정보를 확인할 필요가 없어 인증 속도가 빠름 (단, Refresh Token 검증 시 DB 조회 필요) |
| **Cross-domain 지원** | 쿠키 기반 세션과 달리, 도메인이 다른 환경(Web, Mobile, 다른 서비스 등)에서도 동일하게 인증을 처리할 수 있어 유연함|
| **높은 확장성**| 여러 서버로 구성된 분산 환경(MSA 등)에서 인증 상태를 쉽게 공유할 수 있음|

<br>

### 4.2. 단점
1.  **토큰 탈취 시 위험**
    > JWT는 **Stateless** 특성상 한 번 발급되면 서버에서 해당 토큰을 무효화하기 어려움. 따라서 토큰이 탈취되면 유효기간(`exp`)이 만료될 때까지 악용될 수 있음. 이를 보완하기 위해 Access Token의 유효기간을 짧게 설정하거나, 별도의 **토큰 블랙리스트**를 구현해야 함.

2.  **토큰 크기가 상대적으로 큼**
    > Payload에 담는 정보가 많아질수록 토큰의 크기가 커짐. 매 요청마다 헤더에 포함되므로, 세션 쿠키 방식에 비해 네트워크 트래픽에 약간의 부담을 줄 수 있음.

3.  **Payload 정보 노출**
    > Payload는 Base64로 인코딩되었을 뿐, 암호화된 것이 아니므로 누구나 디코딩하여 내용을 볼 수 있습니다. 따라서 주민등록번호, 비밀번호 등 민감한 정보는 절대로 Payload에 담아서는 안 됩니다. 만약 Payload를 암호화해야 한다면, JWE(JSON Web Encryption)를 적용할 수 있으나, 구현 복잡도가 크게 증가합니다.

<br>
<br>

## 5. 정리

| 항목|설명|
| --- | --- |
| **정의**| JSON 기반의 서명된 인증 토큰 (RFC 7519)|
| **주요 사용처** | 로그인 유지, API 접근 제어, 서비스 간 인증(OAuth2, OIDC)|
| **핵심 구성**| Header + Payload + Signature|
| **장점**| 서버 상태 비유지(stateless), 빠른 인증, 높은 확장성, Cross-domain 지원 |
| **단점**| 토큰 탈취 시 무효화 어려움, 토큰 크기 문제, Payload 정보 노출|

<br>
<br>

## 6. JWT 사용 시 보안 고려 사항

| 구분| 약점| 해결책|
| :---: | --- | --- |
| **HTTPS 필수 사용**| JWT는 서명되었지만 암호화되지 않았으므로, 중간자 공격(MITM)으로 토큰이 탈취될 수 있음. | 따라서 통신 전 구간에 **HTTPS(TLS/SSL)**를 적용하여 전송 계층을 암호화해야 함. |
| **강력한 비밀키 사용 및 관리**| `HS256` 알고리즘의 비밀키나 `RS256`의 개인키가 유출되면 모든 토큰이 위조될 수 있음. | 키는 충분히 길고 복잡하게 설정하고, 코드에 하드코딩하지 말고 환경 변수나 Secret Manager 등을 통해 안전하게 관리해야 함.|
| **서명 알고리즘 `none` 취약점 방지** | 공격자가 Header의 `alg` 필드를 `none`으로 변경하여 서명 검증을 우회하는 공격 존재 | 서버 측에서는 토큰 검증 시 **허용할 알고리즘 목록(Whitelist)**(예: `['HS256', 'RS256']`)을 명시하여, `none`이나 예상치 못한 알고리즘을 거부해야 함.|
| **안전한 토큰 저장 위치**| **Access Token**은 JavaScript로 접근할 수 없는 곳이 이상적이지만, 현실적으로는 메모리에 저장하는 것이 좋음.| **Refresh Token**은 `HttpOnly`, `Secure`, `SameSite=Strict` 쿠키에 저장하여 XSS와 CSRF 공격으로부터 보호해야 함. `localStorage`는 XSS에 매우 취약하므로 민감한 토큰 저장소로 부적합함. |
| **토큰 폐기(Revocation) 전략 수립**  | 사용자가 로그아웃하거나 비밀번호를 변경했을 때, 기존에 발급된 토큰을 무효화해야 함. | 이를 위해 무효화할 토큰 목록을 관리하는 **블랙리스트(Denylist)**를 Redis와 같은 빠른 저장소에 구현하거나, Refresh Token을 DB에서 삭제하는 등의 전략 필요. |
| **토큰 로테이션(Token Rotation)**| 구현의 복잡성 및 경쟁 상태(Race Condition) 발생 가능성 | Refresh Token을 사용하여 새로운 Access Token을 발급받을 때, 기존 Refresh Token을 무효화하고 **새로운 Refresh Token을 함께 발급**하는 방식. 만약 탈취된 Refresh Token이 사용되더라도, 한 번만 유효하므로 보안성을 높일 수 있음.|
| **민감한 정보는 Payload에 금지**| Payload는 누구나 디코딩할 수 있음| 비밀번호, 개인 식별 정보 등 민감한 데이터를 절대 포함해서는 안 됨|

<br>

### 6.1. 서버 검증 체크리스트 (요약)
- 알고리즘 화이트리스트 강제: `algorithms=["HS256"]` 또는 `["RS256"]`
- 발급자/대상자 검증: `iss`/`aud` 일치 여부 확인
- 시간 클레임 검증: `exp`/`nbf`/`iat` (시계 오차 `leeway` 적용)
- 키 로테이션: 헤더 `kid` 기반으로 올바른 공개키 선택(RS256)
- 예외 처리: 만료/서명불일치/형식오류 → 401/403 일관 응답


<br>
<br>

## 7. JWT 사용을 재고해야 하는 경우
JWT는 많은 장점을 가지고 있지만, 모든 상황에 적합한 만능 해결책은 아님.

*   **단순한 서버-클라이언트 구조일 때**: 단일 서버 환경이고, 상태를 유지하는 것이 복잡하지 않다면 전통적인 **세션/쿠키 방식**이 더 간단하고 효과적일 수 있음. 세션은 서버에서 직접 제어 가능하므로 특정 사용자를 강제 로그아웃시키는 등의 관리가 용이함.
*   **토큰을 즉시 무효화해야 할 때**: 사용자의 권한이 실시간으로 변경되거나, 관리자가 특정 사용자를 즉시 시스템에서 제외해야 하는 등 강력한 중앙 통제가 필요한 시스템에서는 Stateless한 JWT보다 Stateful한 세션 방식이 더 적합할 수 있음. (JWT로도 블랙리스트를 통해 구현할 수 있지만, 이는 JWT의 Stateless 장점을 일부 희생하는 것임.)
