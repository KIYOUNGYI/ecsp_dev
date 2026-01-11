# 3. Connection (연결)

> 출처: ocpp-j-1.6-specification.pdf, Chapter 3
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

OCPP-J를 사용하는 충전기와 중앙 시스템 간의 연결에서 중앙 시스템은 WebSocket 서버로 작동하고 충전기는 WebSocket 클라이언트로 작동한다.

## 3.1. Client request (클라이언트 요청)

연결을 설정하기 위해 충전기는 [RFC6455] 섹션 4 "Opening Handshake"에 설명된 대로 WebSocket 연결을 시작한다.

OCPP-J는 URL 및 WebSocket 하위 프로토콜에 추가 제약 조건을 부과하며, 이는 다음 두 섹션 3.1.1 및 3.1.2에 자세히 설명되어 있다.

### 3.1.1. The connection URL (연결 URL)

WebSocket 연결을 시작하려면 충전기는 연결할 URL([RFC3986])이 필요하다. 이 URL은 이후 "연결 URL"이라고 한다. 이 연결 URL은 충전기에 특정된다. 충전기의 연결 URL에는 충전기 ID가 포함되어 있어 중앙 시스템이 WebSocket 연결이 어떤 충전기에 속하는지 알 수 있다.

OCPP-J를 지원하는 중앙 시스템은 최소 하나의 OCPP-J 엔드포인트 URL을 제공해야 하며(MUST), 충전기는 이로부터 연결 URL을 파생해야 한다(SHOULD). 이 OCPP-J 엔드포인트 URL은 "ws" 또는 "wss" 스킴을 가진 모든 URL일 수 있다. 충전기가 OCPP-J 엔드포인트 URL을 얻는 방법은 이 문서의 범위를 벗어난다.

연결 URL을 파생하기 위해 충전기는 OCPP-J 엔드포인트 URL을 수정하여 경로에 먼저 '/' (U+002F SOLIDUS)를 추가한 다음 충전기를 고유하게 식별하는 문자열을 추가한다. 이 고유 식별 문자열은 [RFC3986]에 설명된 대로 필요에 따라 퍼센트 인코딩되어야 한다.

**예시 1**: ID가 "CP001"인 충전기가 OCPP-J 엔드포인트 URL "ws://centralsystem.example.com/ocpp"를 가진 중앙 시스템에 연결하는 경우 다음과 같은 연결 URL이 생성된다:

```
ws://centralsystem.example.com/ocpp/CP001
```

**예시 2**: ID가 "RDAM 123"인 충전기가 OCPP-J 엔드포인트 URL "wss://centralsystem.example.com/ocppj"를 가진 중앙 시스템에 연결하는 경우 다음과 같은 URL이 생성된다:

```
wss://centralsystem.example.com/ocppj/RDAM%20123
```

### 3.1.2. OCPP version (OCPP 버전)

정확한 OCPP 버전은 Sec-Websocket-Protocol 필드에 지정되어야 한다(MUST). 이는 다음 값 중 하나여야 한다(SHOULD):

**Table 1: OCPP Versions**

| OCPP 버전 | WebSocket 하위 프로토콜 이름 |
|-----------|---------------------------|
| 1.2 | ocpp1.2 |
| 1.5 | ocpp1.5 |
| 1.6 | ocpp1.6 |
| 2.0 | ocpp2.0 |

OCPP 1.2, 1.5 및 2.0은 공식 WebSocket 하위 프로토콜 이름 값이다. 이들은 IANA에 등록되어 있다.

OCPP 1.2 및 1.5가 목록에 있음을 참고하라. JSON over WebSocket 솔루션은 실제 메시지 내용과 독립적이므로 이 솔루션은 이전 OCPP 버전에도 사용할 수 있다. 이러한 경우 구현은 상호 운용성을 위해 SOAP 기반 솔루션에 대한 지원도 유지하는 것이 바람직하다는 점을 유념하라.

OCPP-J 엔드포인트 URL 문자열의 일부로 OCPP 버전을 포함하는 것이 좋은 관행으로 간주된다. 물론 동일한 OCPP-J 엔드포인트 URL에서 여러 프로토콜 버전을 처리할 수 있는 웹 서비스를 실행하는 경우에는 필요하지 않다.

### 3.1.3. Example of an opening HTTP request (초기 HTTP 요청 예시)

다음은 OCPP-J 연결 핸드셰이크의 초기 HTTP 요청 예시이다:

```http
GET /webServices/ocpp/CP3211 HTTP/1.1
Host: some.server.com:33033
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: x3JJHMbDL1EzLkh9GBhXDw==
Sec-WebSocket-Protocol: ocpp1.6, ocpp1.5
Sec-WebSocket-Version: 13
```

굵은 부분은 모든 WebSocket 핸드셰이크 요청에서 찾을 수 있으며, 다른 부분은 이 예시에 특정된다.

이 예시에서 중앙 시스템의 OCPP-J 엔드포인트 URL은 "ws://some.server.com:33033/webServices/ocpp"이다. 충전기의 고유 식별자는 "CP3211"이므로 요청할 경로는 "webServices/ocpp/CP3211"이 된다.

Sec-WebSocket-Protocol 헤더를 통해 충전기는 OCPP1.6J와 OCPP1.5J를 사용할 수 있으며, 전자를 선호한다는 것을 나타낸다.

이 예시의 다른 헤더들은 HTTP 및 WebSocket 프로토콜의 일부이며, 타사 WebSocket 라이브러리 위에 OCPP-J를 구현하는 사람들과는 관련이 없다. 이러한 헤더의 역할은 [RFC2616] 및 [RFC6455]에 설명되어 있다.

## 3.2. Server response (서버 응답)

충전기의 요청을 받으면 중앙 시스템은 [RFC6455]에 설명된 대로 응답으로 핸드셰이크를 완료해야 한다.

다음 OCPP-J 특정 조건이 적용된다:

- 중앙 시스템이 URL 경로의 충전기 식별자를 인식하지 못하는 경우, [RFC6455]에 설명된 대로 상태 404의 HTTP 응답을 보내고 WebSocket 연결을 중단해야 한다(SHOULD).
- 중앙 시스템이 클라이언트가 제공하는 하위 프로토콜 중 하나를 사용하는 것에 동의하지 않는 경우, Sec-WebSocket-Protocol 헤더 없이 응답으로 WebSocket 핸드셰이크를 완료한 다음 즉시 WebSocket 연결을 닫아야 한다(MUST).

따라서 중앙 시스템이 위의 예시 요청을 수락하고 충전기와 OCPP 1.6J 사용에 동의하는 경우, 중앙 시스템의 응답은 다음과 같다:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=
Sec-WebSocket-Protocol: ocpp1.6
```

굵은 부분은 모든 WebSocket 핸드셰이크 응답에서 찾을 수 있으며, 다른 부분은 이 예시에 특정된다.

Sec-WebSocket-Accept 헤더의 역할은 [RFC6455]에 설명되어 있다.

Sec-WebSocket-Protocol 헤더는 서버가 이 연결에서 OCPP1.6J를 사용할 것임을 나타낸다.

## 3.3. More information (추가 정보)

WebSocket 핸드셰이크를 직접 구현하는 사람들을 위해 [WS] 및 [WIKIWS]에서 WebSocket 프로토콜에 대한 추가 정보를 제공한다.
