# 4. RPC framework (RPC 프레임워크)

> 출처: ocpp-j-1.6-specification.pdf, Chapter 4
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

## 4.1. Introduction (소개)

WebSocket은 전이중 연결이며, 단순히 데이터가 들어가고 나올 수 있는 파이프이며 입력과 출력 사이에 명확한 관계가 없다. WebSocket 프로토콜 자체는 메시지를 요청과 응답으로 연관시키는 방법을 제공하지 않는다. 이러한 요청/응답 관계를 인코딩하려면 WebSocket 위에 작은 프로토콜이 필요하다. 이 문제는 WebSocket의 더 많은 사용 사례에서 발생하므로 이를 해결하기 위한 기존 스키마가 있다. 가장 널리 사용되는 것은 WAMP([WAMP] 참조)이지만 해당 프레임워크의 현재 버전에서는 RPC를 대칭적으로 처리하는 것이 WAMP 호환이 아니다. 필요한 프레임워크가 매우 간단하기 때문에 WAMP에서 영감을 받아 필요하지 않은 것은 생략하고 누락된 것을 추가하여 자체 프레임워크를 정의하기로 했다.

기본적으로 우리가 필요한 것은 매우 간단하다: 메시지(CALL)를 보내고 응답(CALLRESULT)을 받거나 메시지가 제대로 처리될 수 없는 이유에 대한 설명(CALLERROR)을 받아야 한다. 향후 호환성을 위해 이러한 메시지의 번호 매기기를 WAMP와 동기화할 것이다. 실제 OCPP 메시지는 최소한 메시지 유형, 고유 메시지 ID 및 페이로드(OCPP 메시지 자체)를 포함하는 래퍼에 넣어진다.

### 4.1.1. Synchronicity (동기성)

충전기 또는 중앙 시스템은 이전에 보낸 모든 CALL 메시지가 응답되었거나 시간 초과되지 않은 한 상대방에게 CALL 메시지를 보내서는 안 된다(SHOULD NOT). CALL 메시지의 메시지 ID를 가진 CALLERROR 또는 CALLRESULT 메시지를 받았을 때 CALL 메시지가 응답된 것이다.

다음의 경우 CALL 메시지가 시간 초과된 것이다:
- 응답되지 않았고,
- 메시지가 전송된 이후 구현 종속적인 시간 초과 간격이 경과했다.

구현은 이 시간 초과 간격을 자유롭게 선택할 수 있다. 상대방과 통신하는 데 사용되는 네트워크 종류를 고려하는 것이 권장된다(RECOMMENDED). 모바일 네트워크는 일반적으로 고정 회선보다 최악의 경우 왕복 시간이 훨씬 길다.

> **참고** | 위의 요구 사항은 충전기 또는 중앙 시스템이 CALLERROR 또는 CALLRESULT를 기다리는 동안 상대방으로부터 CALL 메시지를 받을 수 있다는 것을 배제하지 않는다. 양쪽의 CALL 메시지가 항상 서로 교차할 수 있기 때문에 이러한 상황을 방지하기 어렵다.

### 4.1.2. Character encoding (문자 인코딩)

래퍼와 페이로드로 구성된 전체 메시지는 UTF-8([RFC3629] 참조) 문자 인코딩으로 인코딩된 유효한 JSON이어야 한다(MUST).

모든 유효한 US-ASCII 텍스트는 유효한 UTF-8이기도 하므로, 시스템이 US-ASCII 텍스트만 전송하는 경우 전송하는 모든 메시지는 UTF-8 요구 사항을 준수한다. 충전기 또는 중앙 시스템은 자연어 텍스트를 전송할 때만 US-ASCII에 없는 문자를 사용해야 한다(SHOULD). 이러한 자연어 텍스트의 예는 OCPP 2.0의 LocalizedText 타입의 텍스트이다.

### 4.1.3. The message type (메시지 타입)

메시지 타입을 식별하기 위해 다음 메시지 타입 번호 중 하나를 사용해야 한다(MUST).

**Table 2: Message types**

| MessageType | MessageTypeNumber | Direction |
|-------------|-------------------|-----------|
| CALL | 2 | Client-to-Server |
| CALLRESULT | 3 | Server-to-Client |
| CALLERROR | 4 | Server-to-Client |

서버가 이 목록에 없는 메시지 타입 번호를 가진 메시지를 받으면, 메시지 페이로드를 무시해야 한다(SHALL). 각 메시지 타입에는 추가 필수 필드가 있을 수 있다.

### 4.1.4. The message ID (메시지 ID)

메시지 ID는 요청을 식별하는 역할을 한다. CALL 메시지의 메시지 ID는 동일한 WebSocket 연결에서 동일한 발신자가 CALL 메시지에 대해 이전에 사용한 모든 메시지 ID와 달라야 한다(MUST). CALLRESULT 또는 CALLERROR 메시지의 메시지 ID는 CALLRESULT 또는 CALLERROR 메시지가 응답하는 CALL 메시지의 메시지 ID와 같아야 한다(MUST).

**Table 3: Unique Message ID**

| Name | Datatype | Restrictions |
|------|----------|--------------|
| messageId | string | 최대 36자, GUID를 허용하기 위함 |

## 4.2. Message structures for different message types (메시지 타입별 메시지 구조)

> **참고** | 다음 단락에서 충전기 ID가 누락된 것을 발견할 수 있다. ID는 WebSocket 연결 핸드셰이크 중에 교환되며 연결의 속성이다. 모든 메시지는 이 ID에 의해 전송되거나 이 ID로 전달된다. 따라서 각 메시지에서 이를 반복할 필요가 없다.

### 4.2.1. Call

Call은 항상 4개의 요소로 구성된다: 표준 요소인 MessageTypeId와 UniqueId, 상대방에게 요구되는 특정 Action, 그리고 페이로드(Action에 대한 인수). Call의 구문은 다음과 같다:

```json
[<MessageTypeId>, "<UniqueId>", "<Action>", {<Payload>}]
```

**Table 4: Call Fields**

| Field | Meaning |
|-------|---------|
| UniqueId | 요청과 결과를 매칭하는 데 사용될 고유 식별자이다. |
| Action | 원격 프로시저 또는 액션의 이름. 이것은 SOAP 기반 메시지의 Action 필드와 동일한 값을 포함하는 대소문자 구분 문자열이며, 앞의 슬래시는 없다. |
| Payload | Action과 관련된 인수를 포함하는 JSON 객체이다. 페이로드가 없는 경우 JSON은 두 가지 다른 표기법을 허용한다: `null` 또는 빈 객체 `{}`. 사소해 보이지만 빈 객체 표현만 사용하는 것이 좋은 관행으로 간주된다. Null은 일반적으로 정의되지 않은 것을 나타내며, 이는 비어있는 것과 같지 않고, `{}`가 더 짧다. |

예를 들어, BootNotification 요청은 다음과 같을 수 있다:

```json
[2,
 "19223201",
 "BootNotification",
 {"chargePointVendor": "VendorX", "chargePointModel": "SingleSocketCharger"}
]
```

### 4.2.2. CallResult

Call이 올바르게 처리될 수 있는 경우 결과는 일반 CallResult가 된다. OCPP 응답 정의에 포함된 오류 상황은 이 컨텍스트에서 오류로 간주되지 않는다. 이들은 일반 결과이며 수신자에게 바람직하지 않은 결과라도 일반 CallResult로 처리된다.

CallResult는 항상 3개의 요소로 구성된다: 표준 요소인 MessageTypeId와 UniqueId, 그리고 원래 Call의 Action에 대한 응답을 포함하는 페이로드. CallResult의 구문은 다음과 같다:

```json
[<MessageTypeId>, "<UniqueId>", {<Payload>}]
```

**Table 5: CallResult Fields**

| Field | Meaning |
|-------|---------|
| UniqueId | Call 요청의 ID와 정확히 동일해야 하므로 수신자가 요청과 결과를 매칭할 수 있다. |
| Payload | 실행된 Action의 결과를 포함하는 JSON 객체이다. 페이로드가 없는 경우 JSON은 두 가지 다른 표기법을 허용한다: `null` 또는 빈 객체 `{}`. 사소해 보이지만 빈 객체 표현만 사용하는 것이 좋은 관행으로 간주된다. Null은 일반적으로 정의되지 않은 것을 나타내며, 이는 비어있는 것과 같지 않고, `{}`가 더 짧다. |

예를 들어, BootNotification 응답은 다음과 같을 수 있다:

```json
[3,
 "19223201",
 {"status":"Accepted", "currentTime":"2013-02-01T20:53:32.486Z", "heartbeatInterval":300}
]
```

### 4.2.3. CallError

CallError는 다음 두 가지 상황에서만 사용한다:

1. 메시지 전송 중 오류가 발생했다. 이는 네트워크 문제, 서비스 가용성 문제 등일 수 있다.
2. Call이 수신되었지만 Call의 내용이 적절한 메시지에 대한 요구 사항을 충족하지 않는다. 이는 필수 필드 누락, 동일한 고유 식별자를 가진 기존 Call이 이미 처리 중인 경우, 고유 식별자가 너무 긴 경우 등일 수 있다.

CallError는 항상 5개의 요소로 구성된다: 표준 요소인 MessageTypeId와 UniqueId, errorCode 문자열, errorDescription 문자열, errorDetails 객체. CallError의 구문은 다음과 같다:

```json
[<MessageTypeId>, "<UniqueId>", "<errorCode>", "<errorDescription>", {<errorDetails>}]
```

**Table 6: CallError Fields**

| Field | Meaning |
|-------|---------|
| UniqueId | Call 요청의 ID와 정확히 동일해야 하므로 수신자가 요청과 결과를 매칭할 수 있다. |
| ErrorCode | 이 필드는 아래 ErrorCode 테이블의 문자열을 포함해야 한다. |
| ErrorDescription | 가능하면 채워야 하며, 그렇지 않으면 명확한 빈 문자열 `""`. |
| ErrorDetails | 이 JSON 객체는 정의되지 않은 방식으로 오류 세부 정보를 설명한다. 오류 세부 정보가 없는 경우 빈 객체 `{}`를 채워야 한다(MUST). |

**Table 7: Valid Error Codes**

| Error Code | Description |
|------------|-------------|
| NotImplemented | 요청된 Action이 수신자에게 알려지지 않음 |
| NotSupported | 요청된 Action이 인식되지만 수신자가 지원하지 않음 |
| InternalError | 내부 오류가 발생하여 수신자가 요청된 Action을 성공적으로 처리할 수 없음 |
| ProtocolError | Action에 대한 페이로드가 불완전함 |
| SecurityError | Action 처리 중 보안 문제가 발생하여 수신자가 Action을 성공적으로 완료하지 못하도록 방지함 |
| FormationViolation | Action에 대한 페이로드가 구문적으로 올바르지 않거나 Action에 대한 PDU 구조를 따르지 않음 |
| PropertyConstraintViolation | 페이로드가 구문적으로 올바르지만 최소한 하나의 필드에 잘못된 값이 포함됨 |
| OccurenceConstraintViolation | Action에 대한 페이로드가 구문적으로 올바르지만 최소한 하나의 필드가 발생 제약 조건을 위반함 |
| TypeConstraintViolation | Action에 대한 페이로드가 구문적으로 올바르지만 최소한 하나의 필드가 데이터 타입 제약 조건을 위반함 (예: `"somestring": 12`) |
| GenericError | 이전 오류로 포함되지 않는 기타 오류 |
