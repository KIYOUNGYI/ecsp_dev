# 4. Operations Initiated by Charge Point (충전기에 의해 시작된 작업)

> 출처: ocpp-1.6 edition 2.pdf, Chapter 4
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

## 4.1. Authorize (인증)

전기차 소유자가 충전을 시작하거나 중지하기 전에, 충전기는 작업을 인증해야 한다. 충전기는 인증 후에만 에너지를 공급해야 한다(SHALL). 트랜잭션을 중지할 때, 충전기는 트랜잭션 중지에 사용된 식별자가 트랜잭션을 시작한 식별자와 다른 경우에만 Authorize.req를 전송해야 한다(SHALL).

Authorize.req는 충전을 위한 식별자의 인증에만 사용되어야 한다(SHOULD).

충전기는 Local Authorization List에 설명된 대로 중앙 시스템을 개입시키지 않고 로컬로 식별자를 인증할 수 있다(MAY). 사용자가 제시한 idTag가 Local Authorization List 또는 Authorization Cache에 없으면, 충전기는 인증을 요청하기 위해 중앙 시스템에 Authorize.req PDU를 전송해야 한다(SHALL). idTag가 Local Authorization List 또는 Authorization Cache에 있으면, 충전기는 중앙 시스템에 Authorize.req PDU를 전송할 수 있다(MAY).

Authorize.req PDU를 수신하면, 중앙 시스템은 Authorize.conf PDU로 응답해야 한다(SHALL). 이 응답 PDU는 중앙 시스템이 idTag를 수락하는지 여부를 나타내야 한다(SHALL). 중앙 시스템이 idTag를 수락하면 응답 PDU는 parentIdTag를 포함할 수 있으며(MAY), 수락을 나타내거나 거부 사유를 나타내는 인증 상태 값을 포함해야 한다(MUST).

충전기가 Authorization Cache를 구현한 경우, Authorize.conf PDU를 수신하면 충전기는 idTag가 Local Authorization List에 없는 경우, Authorization Cache에 설명된 대로 응답의 IdTagInfo 값으로 캐시 항목을 업데이트해야 한다(SHALL).

## 4.2. Boot Notification (부팅 알림)

시작 후, 충전기는 구성 정보(예: 버전, 벤더 등)와 함께 중앙 시스템에 요청을 전송해야 한다(SHALL). 중앙 시스템은 충전기를 수락할지 여부를 나타내는 응답을 해야 한다(SHALL).

충전기는 부팅하거나 재부팅할 때마다 BootNotification.req PDU를 전송해야 한다(SHALL). 물리적 전원 투입/재부팅과 중앙 시스템이 Accepted 또는 Pending을 반환하는 BootNotification의 성공적인 완료 사이에, 충전기는 중앙 시스템에 다른 요청을 전송해서는 안 된다(SHALL NOT). 여기에는 이전부터 충전기에 남아 있는 캐시된 메시지도 포함된다.

중앙 시스템이 상태 Accepted로 BootNotification.conf를 응답하면, 충전기는 응답 PDU의 interval에 따라 하트비트 간격을 조정하며, 제공된 중앙 시스템의 현재 시간과 내부 시계를 동기화하는 것이 권장된다(RECOMMENDED). 중앙 시스템이 Accepted 이외의 것을 반환하면, interval 필드의 값은 다음 BootNotification 요청을 보내기 전의 최소 대기 시간을 나타낸다. 해당 interval 값이 0이면, 충전기는 중앙 시스템에 요청을 과도하게 보내는 것을 피하는 방식으로 자체적으로 대기 간격을 선택한다. TriggerMessage.req로 요청받지 않는 한, 충전기는 그보다 일찍 BootNotification.req를 전송해서는 안 된다(SHOULD NOT).

중앙 시스템이 상태 Rejected를 반환하면, 충전기는 앞서 언급한 재시도 간격이 만료될 때까지 중앙 시스템에 OCPP 메시지를 전송해서는 안 된다(SHALL NOT). 이 간격 동안 충전기는 중앙 시스템에서 더 이상 연결할 수 없을 수 있다. 예를 들어 통신 채널을 닫거나 통신 하드웨어를 종료할 수 있다(MAY). 또한 중앙 시스템은 예를 들어 시스템 리소스를 확보하기 위해 통신 채널을 닫을 수 있다(MAY). Rejected 상태인 동안, 충전기는 중앙 시스템이 시작한 메시지에 응답해서는 안 된다(SHALL NOT). 중앙 시스템은 어떤 메시지도 시작해서는 안 된다(SHOULD NOT).

중앙 시스템은 또한 Pending 등록 상태를 반환하여 중앙 시스템이 충전기를 수락하기 전에 충전기에서 특정 정보를 검색하거나 설정하려고 함을 나타낼 수 있다(MAY). 중앙 시스템이 Pending 상태를 반환하면, 통신 채널은 충전기나 중앙 시스템에 의해 닫혀서는 안 된다(SHOULD NOT). 중앙 시스템은 충전기에서 정보를 검색하거나 구성을 변경하기 위해 요청 메시지를 보낼 수 있다(MAY). 충전기는 이러한 메시지에 응답해야 한다(SHOULD). 충전기는 중앙 시스템이 TriggerMessage.req 요청으로 지시하지 않는 한 중앙 시스템에 요청 메시지를 보내서는 안 된다(SHALL NOT).

Pending 상태인 동안, 다음 중앙 시스템 시작 메시지는 허용되지 않는다:
- RemoteStartTransaction.req
- RemoteStopTransaction.req

### 4.2.1. Transactions before being accepted by a Central System (중앙 시스템에 수락되기 전 트랜잭션)

충전기 운영자는 충전기가 중앙 시스템에 수락되기 전에 트랜잭션을 수락하도록 충전기를 구성하도록 선택할 수 있다(MAY). 이러한 동작을 구현하려는 당사자는 해당 트랜잭션이 중앙 시스템에 전달될 수 있을지 불확실하다는 것을 인식해야 한다.

재시작 후(예: 원격 재설정 명령, 정전, 펌웨어 업데이트, 소프트웨어 오류 등으로 인해) 충전기는 다시 중앙 시스템에 연결해야 하며(MUST) BootNotification 요청을 전송해야 한다(SHALL). 충전기가 중앙 시스템으로부터 BootNotification.conf를 받지 못하고, 올바르게 사전 설정된 내장 비휘발성 실시간 시계 하드웨어가 없는 경우, 충전기는 유효한 날짜/시간 설정이 없을 수 있어 나중에 트랜잭션의 날짜/시간을 결정하는 것이 불가능할 수 있다.

또한 (예: 구성 오류로 인해) 중앙 시스템이 장기간 또는 무기한으로 Accepted 이외의 상태를 나타내는 경우도 있을 수 있다.

일반적으로 충전기가 (현재 연결 설정, URL 등을 사용하여) 중앙 시스템에 의해 이전에 수락된 적이 없는 경우, 사용자를 인증할 수 없고 실행 중인 트랜잭션이 프로비저닝 프로세스와 충돌할 수 있으므로 충전기에서 모든 충전 서비스를 거부하는 것이 일반적으로 권장된다.

## 4.3. Data Transfer (데이터 전송)

충전기가 OCPP에서 지원하지 않는 기능에 대한 정보를 중앙 시스템에 전송해야 하는 경우, DataTransfer.req PDU를 사용해야 한다(SHALL).

요청의 vendorId는 중앙 시스템에 알려져 있어야 하며(SHOULD) 벤더별 구현을 고유하게 식별해야 한다. VendorId는 역방향 DNS 네임스페이스의 값이어야 하며(SHOULD), 역순으로 된 이름의 최상위 계층은 벤더 조직의 공개적으로 등록된 기본 DNS 이름에 해당해야 한다.

선택적으로, 요청 PDU의 messageId는 특정 메시지 또는 구현을 나타내는 데 사용될 수 있다(MAY).

요청 및 응답 PDU 모두의 데이터 길이는 정의되지 않았으며 관련된 모든 당사자가 합의해야 한다.

요청 수신자가 특정 vendorId에 대한 구현이 없는 경우 'UnknownVendor' 상태를 반환해야 하며(SHALL) data 요소는 존재하지 않아야 한다(SHALL NOT). messageId 불일치의 경우(사용된 경우) 수신자는 'UnknownMessageId' 상태를 반환해야 한다(SHALL). 다른 모든 경우에 'Accepted' 또는 'Rejected' 상태의 사용과 data 요소는 관련 당사자 간의 벤더별 합의의 일부이다.

## 4.4. Diagnostics Status Notification (진단 상태 알림)

충전기는 진단 업로드 상태에 대해 중앙 시스템에 알리기 위해 알림을 보낸다. 충전기는 진단 업로드가 진행 중이거나 성공적으로 완료되었거나 실패했음을 중앙 시스템에 알리기 위해 DiagnosticsStatusNotification.req PDU를 전송해야 한다(SHALL). 충전기는 진단 업로드 중이 아닐 때 Diagnostics Status Notification에 대한 TriggerMessage를 수신한 후에만 Idle 상태를 전송해야 한다(SHALL).

DiagnosticsStatusNotification.req PDU를 수신하면, 중앙 시스템은 DiagnosticsStatusNotification.conf로 응답해야 한다(SHALL).

## 4.5. Firmware Status Notification (펌웨어 상태 알림)

충전기는 펌웨어 업데이트 진행 상황에 대해 중앙 시스템에 알리기 위해 알림을 보낸다. 충전기는 펌웨어 업데이트의 다운로드 및 설치 진행 상황을 중앙 시스템에 알리기 위해 FirmwareStatusNotification.req PDU를 전송해야 한다(SHALL). 충전기는 펌웨어를 다운로드/설치하는 중이 아닐 때 Firmware Status Notification에 대한 TriggerMessage를 수신한 후에만 Idle 상태를 전송해야 한다(SHALL).

FirmwareStatusNotification.req PDU를 수신하면, 중앙 시스템은 FirmwareStatusNotification.conf로 응답해야 한다(SHALL).

FirmwareStatusNotification.req PDU는 중앙 시스템이 FirmwareUpdate.req PDU로 시작한 업데이트 프로세스의 상태로 중앙 시스템을 업데이트하기 위해 전송되어야 한다(SHALL).

## 4.6. Heartbeat (하트비트)

충전기가 여전히 연결되어 있음을 중앙 시스템에 알리기 위해, 충전기는 구성 가능한 시간 간격 후에 하트비트를 전송한다.

충전기는 중앙 시스템이 충전기가 여전히 살아 있음을 알 수 있도록 Heartbeat.req PDU를 전송해야 한다(SHALL).

Heartbeat.req PDU를 수신하면, 중앙 시스템은 Heartbeat.conf로 응답해야 한다(SHALL). 응답 PDU는 중앙 시스템의 현재 시간을 포함해야 하며(SHALL), 충전기가 내부 시계를 동기화하는 데 사용하는 것이 권장된다(RECOMMENDED).

충전기는 구성된 하트비트 간격 내에 다른 PDU를 중앙 시스템에 전송한 경우 Heartbeat.req PDU 전송을 건너뛸 수 있다(MAY). 이는 중앙 시스템이 PDU를 수신할 때마다 Heartbeat.req PDU를 받았을 때와 동일한 방식으로 충전기의 가용성을 가정해야 함을 의미한다(SHOULD).

## 4.7. Meter Values (미터 값)

충전기는 전기 미터 또는 기타 센서/변환기 하드웨어를 샘플링하여 미터 값에 대한 추가 정보를 제공할 수 있다(MAY). 미터 값을 언제 전송할지는 충전기가 결정한다. 이는 ChangeConfiguration.req 메시지를 사용하여 데이터 획득 간격을 구성하고 획득 및 보고할 데이터를 지정하여 설정할 수 있다.

충전기는 미터 값을 오프로드하기 위해 MeterValues.req PDU를 전송해야 한다(SHALL). 요청 PDU는 각 샘플에 대해 다음을 포함해야 한다(SHALL):

1. 샘플이 채취된 커넥터의 id. connectorId가 0이면 전체 충전기와 연관된다. connectorId가 0이고 Measurand가 에너지 관련인 경우, 샘플은 주 에너지 미터에서 채취되어야 한다(SHOULD).

2. 이러한 값이 관련된 트랜잭션의 transactionId(해당하는 경우). 진행 중인 트랜잭션이 없거나 값이 주 미터에서 가져온 경우, transaction id는 생략될 수 있다.

3. MeterValue 타입의 하나 이상의 meterValue 요소, 각각 특정 시점에 채취된 하나 이상의 데이터 값 세트를 나타낸다.

각 MeterValue 요소는 타임스탬프와 모두 동일한 시점에 캡처된 하나 이상의 개별 sampledValue 요소 세트를 포함한다. 각 sampledValue 요소는 단일 값 데이터를 포함한다. 각 sampledValue의 특성은 선택적 measurand, context, location, unit, phase 및 format 필드에 의해 결정된다.

선택적 **measurand** 필드는 측정/보고되는 값의 유형을 지정한다.

선택적 **context** 필드는 판독을 트리거하는 이유/이벤트를 지정한다.

선택적 **location** 필드는 측정이 이루어지는 위치를 지정한다(예: Inlet, Outlet).

선택적 **phase** 필드는 값이 적용되는 전기 설비의 상 또는 상들을 지정한다. 충전기는 전기 미터(또는 없는 경우 그리드 연결) 관점에서 모든 상 번호 종속 값을 보고해야 한다(SHALL).

> **참고:** phase 필드는 모든 Measurand에 적용되는 것은 아니다.

> **참고:** 두 개의 측정값(Current.Offered 및 Power.Offered)은 엄밀히 말하면 측정된 값이 아니다. 이들은 EV에 제공되는 최대 전류/전력량을 나타내며 스마트 충전 애플리케이션에서 사용하기 위한 것이다.

개별 커넥터 상 회전 정보의 경우, 중앙 시스템은 GetConfiguration을 통해 충전기의 ConnectorPhaseRotation 구성 키를 조회할 수 있다(MAY). 충전기는 그리드 연결에 대한 상 회전을 보고해야 한다(SHALL). 커넥터당 가능한 값은 다음과 같다:

NotApplicable, Unknown, RST, RTS, SRT, STR, TRS 및 TSR. 자세한 내용은 Standard Configuration Key Names & Values 섹션을 참조하라.

**실험적(EXPERIMENTAL)** 선택적 **format** 필드는 데이터가 단순 숫자 값("Raw")으로 정상(기본) 형식으로 표현되는지, 또는 16진수 데이터로 표현되는 불투명한 디지털 서명된 바이너리 데이터 블록인 "SignedData"로 표현되는지를 지정한다. 이 실험적 필드는 더 성숙한 대안 솔루션이 제공될 때 이후 버전에서 더 이상 사용되지 않고 제거될 수 있다.

이전 버전과의 호환성을 유지하기 위해, sampledValue 요소의 모든 선택적 필드의 기본값은 추가 필드가 없는 값이 Wh(와트시) 단위의 능동 수입 에너지의 레지스터 판독값으로 해석되도록 설정되어 있다.

MeterValues.req PDU를 수신하면, 중앙 시스템은 MeterValues.conf로 응답해야 한다(SHALL).

중앙 시스템이 수신한 MeterValues.req에 포함된 데이터에 대해 정상성 검사를 적용할 가능성이 높다. 이러한 정상성 검사의 결과가 중앙 시스템이 MeterValues.conf로 응답하지 않도록 해서는 안 된다(SHOULD NOT). MeterValues.conf로 응답하지 않으면 Error responses to transaction-related messages에 지정된 대로 충전기가 동일한 메시지를 다시 시도하게 될 뿐이다.

## 4.8. Start Transaction (트랜잭션 시작)

충전기는 시작된 트랜잭션에 대해 알리기 위해 중앙 시스템에 StartTransaction.req PDU를 전송해야 한다(SHALL). 이 트랜잭션이 예약을 종료하는 경우(Reserve Now 작업 참조), StartTransaction.req는 reservationId를 포함해야 한다(MUST).

StartTransaction.req PDU를 수신하면, 중앙 시스템은 StartTransaction.conf PDU로 응답해야 한다(SHOULD). 이 응답 PDU는 transaction id와 인증 상태 값을 포함해야 한다(MUST).

중앙 시스템은 StartTransaction.req PDU의 식별자 유효성을 확인해야 한다(MUST). 왜냐하면 식별자가 오래된 정보를 사용하여 충전기에 의해 로컬로 인증되었을 수 있기 때문이다. 예를 들어, 식별자가 충전기의 Authorization Cache에 추가된 이후 차단되었을 수 있다.

충전기가 Authorization Cache를 구현한 경우, StartTransaction.conf PDU를 수신하면 충전기는 idTag가 Local Authorization List에 없는 경우, Authorization Cache에 설명된 대로 응답의 IdTagInfo 값으로 캐시 항목을 업데이트해야 한다(SHALL).

중앙 시스템이 수신한 StartTransaction.req에 포함된 데이터에 대해 정상성 검사를 적용할 가능성이 높다. 이러한 정상성 검사의 결과가 중앙 시스템이 StartTransaction.conf로 응답하지 않도록 해서는 안 된다(SHOULD NOT). StartTransaction.conf로 응답하지 않으면 Error responses to transaction-related messages에 지정된 대로 충전기가 동일한 메시지를 다시 시도하게 될 뿐이다.

## 4.9. Status Notification (상태 알림)

충전기는 충전기 내의 상태 변경이나 오류에 대해 중앙 시스템에 알리기 위해 중앙 시스템에 알림을 보낸다. 다음 표는 충전기가 중앙 시스템에 StatusNotification.req PDU를 전송할 수 있는(MAY) 이전 상태(왼쪽 열)에서 새 상태(상단 행)로의 변경을 나타낸다.

> **참고:** 이전 OCPP 버전에서 정의된 Occupied 상태는 더 이상 관련이 없다. Occupied 상태는 5개의 새로운 상태로 분할되었다: Preparing, Charging, SuspendedEV, SuspendedEVSE 및 Finishing.

> **참고:** Status Notification에서는 향후 호환성을 위해 Socket 또는 Charge Point 대신 EVSE가 사용된다.

### 상태 전환 테이블

| State From \ To: | 1<br>Available | 2<br>Preparing | 3<br>Charging | 4<br>SuspendedEV | 5<br>SuspendedEVSE | 6<br>Finishing | 7<br>Reserved | 8<br>Unavailable | 9<br>Faulted |
|-----------------|----------------|----------------|---------------|------------------|--------------------|----------------|---------------|------------------|--------------|
| **A - Available** | | A2 | A3 | A4 | A5 | | A7 | A8 | A9 |
| **B - Preparing** | B1 | | B3 | B4 | B5 | B6 | | | B9 |
| **C - Charging** | C1 | | | C4 | C5 | C6 | | C8 | C9 |
| **D - SuspendedEV** | D1 | | D3 | | D5 | D6 | | D8 | D9 |
| **E - SuspendedEVSE** | E1 | | E3 | E4 | | E6 | | E8 | E9 |
| **F - Finishing** | F1 | F2 | | | | | | F8 | F9 |
| **G - Reserved** | G1 | G2 | | | | | | G8 | G9 |
| **H - Unavailable** | H1 | H2 | H3 | H4 | H5 | | | | H9 |
| **I - Faulted** | I1 | I2 | I3 | I4 | I5 | I6 | I7 | I8 | |

### 상태 변경 이벤트 설명

다음 표는 상태 변경을 유발할 수 있는 이벤트를 설명한다:

| 이벤트 | 설명 |
|--------|------|
| **A2** | 사용이 시작됨 (예: 플러그 삽입, 주차 공간 점유 감지, idTag 제시, 시작 버튼 누름, RemoteStartTransaction.req 수신) |
| **A3** | 인증 수단이 없는 충전기에서 가능할 수 있음 |
| **A4** | A3와 유사하지만 EV가 충전을 시작하지 않음 |
| **A5** | A3와 유사하지만 EVSE가 충전을 허용하지 않음 |
| **A7** | 커넥터를 예약하는 Reserve Now 메시지가 수신됨 |
| **A8** | 커넥터를 Unavailable로 설정하는 Change Availability 메시지가 수신됨 |
| **A9** | 추가 충전 작업을 방해하는 오류가 감지됨 |
| **B1** | 의도된 사용이 종료됨 (예: 플러그 제거, 주차 공간 점유 해제, idTag 두 번째 제시, 예상 사용자 동작에 대한 시간 초과(구성 키 ConnectionTimeOut으로 설정)) |
| **B3** | 충전을 위한 모든 전제 조건이 충족되고 충전 프로세스가 시작됨 |
| **B4** | 충전을 위한 모든 전제 조건이 충족되었지만 EV가 충전을 시작하지 않음 |
| **B5** | 충전을 위한 모든 전제 조건이 충족되었지만 EVSE가 충전을 허용하지 않음 |
| **B6** | 시간 초과. 사용이 시작되었지만(예: 플러그 삽입, 주차 공간 점유 감지) 시간 초과 내에 idTag가 제시되지 않음 |
| **B9** | 추가 충전 작업을 방해하는 오류가 감지됨 |
| **C1** | 사용자 동작이 필요하지 않은 상태에서 충전 세션이 종료됨 (예: EV 측에서 고정 케이블이 제거됨) |
| **C4** | EV 요청에 따라 충전이 중지됨 (예: S2가 열림) |
| **C5** | EVSE 요청에 따라 충전이 중지됨 (예: 스마트 충전 제한, StartTransaction.conf의 AuthorizationStatus에 의해 트랜잭션이 무효화됨) |
| **C6** | 사용자 또는 Remote Stop Transaction 메시지에 의해 트랜잭션이 중지되고 추가 사용자 동작이 필요함 (예: 케이블 제거, 주차 공간 이탈) |
| **C8** | 충전 세션이 종료되고, 사용자 동작이 필요하지 않으며, 커넥터가 Unavailable이 되도록 예약됨 |
| **C9** | 추가 충전 작업을 방해하는 오류가 감지됨 |
| **D1** | 사용자 동작이 필요하지 않은 상태에서 충전 세션이 종료됨 |
| **D3** | EV의 요청에 따라 충전이 재개됨 (예: S2가 닫힘) |
| **D5** | EVSE에 의해 충전이 일시중지됨 (예: 스마트 충전 제한으로 인해) |
| **D6** | 트랜잭션이 중지되고 추가 사용자 동작이 필요함 |
| **D8** | 충전 세션이 종료되고, 사용자 동작이 필요하지 않으며, 커넥터가 Unavailable이 되도록 예약됨 |
| **D9** | 추가 충전 작업을 방해하는 오류가 감지됨 |
| **E1** | 사용자 동작이 필요하지 않은 상태에서 충전 세션이 종료됨 |
| **E3** | EVSE 제한이 해제되어 충전이 재개됨 |
| **E4** | EVSE 제한이 해제되었지만 EV가 충전을 시작하지 않음 |
| **E6** | 트랜잭션이 중지되고 추가 사용자 동작이 필요함 |
| **E8** | 충전 세션이 종료되고, 사용자 동작이 필요하지 않으며, 커넥터가 Unavailable이 되도록 예약됨 |
| **E9** | 추가 충전 작업을 방해하는 오류가 감지됨 |
| **F1** | 모든 사용자 동작이 완료됨 |
| **F2** | 사용자가 충전 세션을 재시작함 (예: 케이블 재연결, idTag 다시 제시), 이로써 새 트랜잭션이 생성됨 |
| **F8** | 모든 사용자 동작이 완료되고 커넥터가 Unavailable이 되도록 예약됨 |
| **F9** | 추가 충전 작업을 방해하는 오류가 감지됨 |
| **G1** | 예약이 만료되거나 Cancel Reservation 메시지가 수신됨 |
| **G2** | 예약 식별자가 제시됨 |
| **G8** | 예약이 만료되거나 Cancel Reservation 메시지가 수신되고 커넥터가 Unavailable이 되도록 예약됨 |
| **G9** | 추가 충전 작업을 방해하는 오류가 감지됨 |
| **H1** | Change Availability 메시지에 의해 커넥터가 Available로 설정됨 |
| **H2** | 사용자가 충전기와 상호작용한 후 커넥터가 Available로 설정됨 |
| **H3** | 커넥터가 Available로 설정되고 충전을 시작하기 위한 사용자 동작이 필요하지 않음 |
| **H4** | H3와 유사하지만 EV가 충전을 시작하지 않음 |
| **H5** | H3와 유사하지만 EVSE가 충전을 허용하지 않음 |
| **H9** | 추가 충전 작업을 방해하는 오류가 감지됨 |
| **I1-I8** | 오류가 해결되고 상태가 오류 이전 상태로 돌아감 |

> **참고:** 충전기 커넥터는 위 표에 표시된 9개 상태 중 하나를 가질 수 있다(MAY). ConnectorId 0의 경우, Available, Unavailable 및 Faulted와 같은 제한된 세트만 적용된다. ConnectorId 0의 상태는 개별 커넥터(>0)의 상태와 직접적인 연결이 없다.

> **참고:** 충전이 EV와 EVSE 모두에 의해 일시중지된 경우, SuspendedEVSE 상태가 SuspendedEV 상태보다 우선해야 한다(SHALL).

> **참고:** Change Availability 명령에 의해 충전기 또는 커넥터가 Unavailable 상태로 설정되면, 'Unavailable' 상태는 재부팅 후에도 지속되어야 한다(MUST). 충전기는 다른 목적(예: 펌웨어 업데이트 중 또는 초기 Accepted RegistrationStatus를 기다리는 동안)으로 내부적으로 Unavailable 상태를 사용할 수 있다(MAY).

Occupied 상태가 5개의 새로운 상태(Preparing, Charging, SuspendedEV, SuspendedEVSE 및 Finishing)로 분할됨에 따라, 더 많은 StatusNotification.req PDU가 충전기에서 중앙 시스템으로 전송될 것이다. 예를 들어, 트랜잭션이 시작될 때 커넥터 상태는 Preparing에서 Charging으로 순차적으로 변경되며, 그 사이에 짧은 SuspendedEV 및/또는 SuspendedEVSE가 몇 초 이내에 발생할 수 있다.

전환 횟수를 제한하기 위해, 충전기는 선택적 구성 키 MinimumStatusDuration에 정의된 시간보다 짧은 시간 동안 활성화된 경우 StatusNotification.req 전송을 생략할 수 있다(MAY). 이러한 방식으로, 충전기는 특정 StatusNotification.req PDU를 전송하지 않도록 선택할 수 있다(MAY).

> **참고:** 충전기 제조업체는 MinimumStatusDuration 설정과 별도로 특정 상태 전환에 대한 최소 상태 지속 시간을 구현했을 수 있다(MAY). MinimumStatusDuration에 설정된 시간은 이 기본 지연에 추가된다. MinimumStatusDuration을 0으로 설정하는 것은 제조업체의 기본 최소 상태 지속 시간을 재정의해서는 안 된다(SHALL NOT).

> **참고:** 높은 MinimumStatusDuration 시간을 설정하면 모든 StatusNotification의 전송이 지연될 수 있다. 충전기는 MinimumStatusDuration 시간이 경과한 후에만 StatusNotification.req를 전송하기 때문이다.

충전기는 오류 조건을 중앙 시스템에 알리기 위해 StatusNotification.req PDU를 전송할 수 있다(MAY). 'status' 필드가 Faulted가 아닌 경우, 충전 작업이 여전히 가능하므로 해당 조건은 경고로 간주되어야 한다.

> **참고:** ChargePointErrorCode EVCommunicationError는 Preparing, SuspendedEV, SuspendedEVSE 및 Finishing 상태에서만 사용되어야 하며(SHALL) 경고로 처리되어야 한다.

충전기가 StopTransactionOnEVSideDisconnect를 false로 설정하고, 트랜잭션이 실행 중이며, EV가 EV 측에서 연결 해제되면, 상태 SuspendedEV인 StatusNotification.req가 중앙 시스템으로 전송되어야 하며(SHOULD), 'errorCode' 필드는 'NoError'로 설정되어야 한다. 충전기는 'info' 필드에 추가 정보를 추가하여 중앙 시스템에 일시중지 사유('EV side disconnected')를 알려야 한다(SHOULD). 현재 트랜잭션은 중지되지 않는다.

충전기가 StopTransactionOnEVSideDisconnect를 true로 설정하고, 트랜잭션이 실행 중이며, EV가 EV 측에서 연결 해제되면, 상태 'Finishing'인 StatusNotification.req가 중앙 시스템으로 전송되어야 하며(SHOULD), 'errorCode' 필드는 'NoError'로 설정되어야 한다. 충전기는 'info' 필드에 추가 정보를 추가하여 중앙 시스템에 중지 사유('EV side disconnected')를 알려야 한다(SHOULD). 현재 트랜잭션은 중지된다.

충전기가 오프라인 상태였다가 중앙 시스템에 연결되면, 다음 규칙에 따라 중앙 시스템에 상태를 업데이트한다:

1. 충전기가 오프라인 상태인 동안 상태가 변경된 경우, 충전기는 현재 상태와 함께 StatusNotification.req PDU를 전송해야 한다(SHOULD).

2. 충전기는 오프라인 상태인 동안 발생한 오류를 보고하기 위해 StatusNotification.req PDU를 전송할 수 있다(MAY).

3. 충전기는 오프라인 상태인 동안 발생했으며 충전기 오류나 충전기의 현재 상태를 중앙 시스템에 알리지 않는 과거 상태 변경 이벤트에 대한 StatusNotification.req PDU를 전송해서는 안 된다(SHOULD NOT).

4. StatusNotification.req 메시지는 설명하는 이벤트가 발생한 순서대로 전송되어야 한다(MUST).

StatusNotification.req PDU를 수신하면, 중앙 시스템은 StatusNotification.conf PDU로 응답해야 한다(SHALL).

## 4.10. Stop Transaction (트랜잭션 중지)

트랜잭션이 중지되면, 충전기는 트랜잭션이 중지되었음을 중앙 시스템에 알리기 위해 StopTransaction.req PDU를 전송해야 한다(SHALL).

StopTransaction.req PDU는 트랜잭션 사용에 대한 추가 세부 정보를 제공하기 위해 선택적 TransactionData 요소를 포함할 수 있다(MAY). 선택적 TransactionData 요소는 MeterValues.req PDU의 meterValue 요소와 동일한 데이터 구조를 사용하여 임의 개수의 MeterValue를 포함하는 컨테이너이다(MeterValues 섹션 참조).

StopTransaction.req PDU를 수신하면, 중앙 시스템은 StopTransaction.conf PDU로 응답해야 한다(SHALL).

> **참고:** 중앙 시스템은 트랜잭션 중지를 방지할 수 없다. StopTransaction.req를 수신했음을 충전기에 알릴 수만 있으며(MAY), 트랜잭션을 중지하는 데 사용된 idTag에 대한 정보를 보낼 수 있다(MAY). 이 정보는 구현된 경우 Authorization Cache를 업데이트하는 데 사용되어야 한다(SHOULD).

요청 PDU의 idTag는 충전기 자체가 트랜잭션을 중지해야 하는 경우 생략될 수 있다(MAY). 예를 들어, 충전기가 재설정을 요청받은 경우이다.

트랜잭션이 정상적인 방식으로 종료되면(예: EV 운전자가 트랜잭션을 중지하기 위해 식별자를 제시함), Reason 요소는 생략될 수 있으며(MAY) Reason은 'Local'로 가정되어야 한다(SHOULD). 트랜잭션이 정상적으로 종료되지 않으면, Reason은 올바른 값으로 설정되어야 한다(SHOULD). 정상적인 트랜잭션 종료의 일부로, 충전기는 케이블을 잠금 해제해야 한다(SHALL)(영구적으로 연결되지 않은 경우).

충전기는 EV에서 케이블이 연결 해제될 때 케이블을 잠금 해제할 수 있다(MAY)(영구적으로 연결되지 않은 경우). 지원되는 경우, 이 기능은 구성 키 UnlockConnectorOnEVSideDisconnect에 의해 보고되고 제어된다.

충전기는 EV에서 케이블이 연결 해제될 때 실행 중인 트랜잭션을 중지할 수 있다(MAY). 지원되는 경우, 이 기능은 구성 키 StopTransactionOnEVSideDisconnect에 의해 보고되고 제어된다.

StopTransactionOnEVSideDisconnect가 false로 설정되면, EV에서 케이블이 연결 해제될 때 트랜잭션이 중지되어서는 안 된다(SHALL NOT). EV가 다시 연결되면 에너지 전송이 다시 허용된다. 이 경우 동일한 진행 중인 트랜잭션 중에 다른 EV가 충전하고 연결을 해제하는 것을 방지할 메커니즘이 없다. UnlockConnectorOnEVSideDisconnect를 false로 설정하면, 사용자가 식별자를 제시할 때까지 커넥터는 충전기에서 잠금 상태로 유지되어야 한다(SHALL).

StopTransactionOnEVSideDisconnect를 true로 설정하면, EV에서 케이블이 연결 해제될 때 트랜잭션이 중지되어야 한다(SHALL). EV가 다시 연결되면, 트랜잭션이 중지되고 새 트랜잭션이 시작될 때까지 에너지 전송이 허용되지 않는다. UnlockConnectorOnEVSideDisconnect가 true로 설정되면, 충전기의 커넥터도 잠금 해제된다.

> **참고:** StopTransactionOnEVSideDisconnect가 false로 설정되면, 이는 UnlockConnectorOnEVSideDisconnect보다 우선해야 한다(SHALL). 즉, StopTransactionOnEVSideDisconnect가 false일 때 EV 측에서 케이블이 연결 해제되면 케이블은 항상 잠금 상태로 유지된다.

> **참고:** StopTransactionOnEVSideDisconnect를 true로 설정하면 EV 측에서 잠금되지 않은 케이블을 뽑아 에너지 흐름을 중지하는 방해 행위를 방지할 수 있다.

중앙 시스템이 수신한 StopTransaction.req에 포함된 데이터에 대해 정상성 검사를 적용할 가능성이 높다. 이러한 정상성 검사의 결과가 중앙 시스템이 StopTransaction.conf로 응답하지 않도록 해서는 안 된다(SHOULD NOT). StopTransaction.conf로 응답하지 않으면 Error responses to transaction-related messages에 지정된 대로 충전기가 동일한 메시지를 다시 시도하게 될 뿐이다.

충전기가 Authorization Cache를 구현한 경우, StopTransaction.conf PDU를 수신하면 충전기는 idTag가 Local Authorization List에 없는 경우, Authorization Cache에 설명된 대로 응답의 IdTagInfo 값으로 캐시 항목을 업데이트해야 한다(SHALL).
