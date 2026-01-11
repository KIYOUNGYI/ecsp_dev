# 9. Standard Configuration Key Names & Values (표준 구성 키 이름 및 값)

> 출처: ocpp-1.6 edition 2.pdf, Chapter 9
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

아래는 이 명세서에서 역할이 표준화된 모든 구성 키 목록이다. 목록은 Feature Profile별로 구분되어 있다. 특정 프로파일 아래에 언급된 필수 구성 키는 충전기가 해당 프로파일을 지원하는 경우에만 지원해야 한다.

boolean 타입의 선택적 구성 키의 경우, 키 목록 없이 GetConfiguration.req에 대한 응답에서 구성 키에 다음 규칙이 적용된다:

- 키가 있으면 충전기는 키로 구성된 기능을 제공하며, 키 값을 설정하여 활성화하거나 비활성화할 수 있다.
- 키가 없으면 충전기는 키로 구성할 수 있는 기능을 제공하지 않는다.

"Accessibility" 속성은 특정 구성 키의 값이 읽기 전용("R")인지 읽기-쓰기("RW")인지를 나타낸다. 키가 읽기 전용인 경우 중앙 시스템은 GetConfiguration을 사용하여 키 값을 읽을 수 있지만 쓸 수는 없다. 접근성이 읽기-쓰기인 경우 중앙 시스템은 ChangeConfiguration을 사용하여 키 값을 쓸 수도 있다.

## 9.1. Core Profile

### 9.1.1. AllowOfflineTxForUnknownId

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | RW |
| **Type** | boolean |
| **Description** | 이 키가 존재하면 충전기는 Unknown Offline Authorization을 지원한다. 이 키가 true 값을 보고하면 Unknown Offline Authorization이 활성화된다. |

### 9.1.2. AuthorizationCacheEnabled

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | RW |
| **Type** | boolean |
| **Description** | 이 키가 존재하면 충전기는 Authorization Cache를 지원한다. 이 키가 true 값을 보고하면 Authorization Cache가 활성화된다. |

### 9.1.3. AuthorizeRemoteTxRequests

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R 또는 RW. 선택은 충전기 구현에 따른다. |
| **Type** | boolean |
| **Description** | RemoteStartTransaction.req 메시지 형식의 트랜잭션 시작 원격 요청이 트랜잭션 시작 로컬 작업처럼 사전에 인증되어야 하는지 여부. |

### 9.1.4. BlinkRepeat

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | times |
| **Description** | 신호를 보낼 때 충전기 조명을 깜박이는 횟수 |

### 9.1.5. ClockAlignedDataInterval

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | seconds |
| **Description** | 클럭 정렬 데이터 간격의 크기(초 단위). 이것은 하루당 균등하게 간격을 둔 집계 간격의 세트 크기(초)이며, 00:00:00(자정)부터 시작한다. 예를 들어, 900(15분) 값은 매일 96개의 15분 간격으로 나누어야 함을 나타낸다.<br>클럭 정렬 데이터가 전송될 때, 해당 간격은 시작 시간과 (선택적) 지속 시간 간격 값으로 식별되며, ISO8601 표준에 따라 표현된다. 모든 "per-period" 데이터(예: 에너지 판독)는 전체 간격(또는 부분 간격 시작 시간 타임스탬프)에 걸쳐 합산(에너지와 같은 measurand의 경우) 또는 평균(다른 값의 경우)되어야 하며, 각 간격이 끝날 때(또는 각 간격의 시작 시 활성화되면) 전송되어야 한다. 각 간격의 기준 시점 판독은 간격 시작 시 시작 시간 타임스탬프로 전송된다.<br>"0" 값(숫자 0)은 관례상 클럭 정렬 데이터를 전송하지 않아야 함을 의미하는 것으로 해석되어야 한다. |

### 9.1.6. ConnectionTimeOut

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | seconds |
| **Description** | 'Preparing' 상태 시작부터 EV 운전자가 충전 케이블 커넥터를 적절한 소켓에 (올바르게) 삽입하지 못해 진행 중인 트랜잭션이 자동으로 취소될 때까지의 간격. 충전기는 원래 상태로 돌아가야 하며(SHALL), 아마도 'Available' 상태일 것이다. |

### 9.1.7. ConnectorPhaseRotation

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | CSL |
| **Description** | 커넥터의 전기 미터(또는 없는 경우 그리드 연결)에 대한 커넥터별 위상 회전. 커넥터별 가능한 값은 다음과 같다:<br>`NotApplicable` (단상 또는 DC 충전기의 경우)<br>`Unknown` (아직 알려지지 않음)<br>`RST` (Standard Reference Phasing)<br>`RTS` (Reversed Reference Phasing)<br>`SRT` (Reversed 240 degree rotation)<br>`STR` (Standard 120 degree rotation)<br>`TRS` (Standard 240 degree rotation)<br>`TSR` (Reversed 120 degree rotation)<br><br>R은 위상 1(L1), S는 위상 2(L2), T는 위상 3(L3)으로 식별될 수 있다.<br><br>알려진 경우 충전기는 인덱스 번호 Zero(0)를 사용하여 그리드 연결과 메인 에너지미터 간의 위상 회전도 보고할 수 있다(MAY).<br><br>값은 CSL로 보고되며 다음과 같이 형식화된다: 0.RST, 1.RST, 2.RTS |

### 9.1.8. ConnectorPhaseRotationMaxLength

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | ConnectorPhaseRotation 구성 키의 최대 항목 수. |

### 9.1.9. GetConfigurationMaxKeys

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | GetConfiguration.req PDU에서 요청된 구성 키의 최대 개수. |

### 9.1.10. HeartbeatInterval

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | seconds |
| **Description** | 중앙 시스템과의 비활성 간격(OCPP 교환 없음) 후 충전기가 Heartbeat.req PDU를 전송해야 하는 시간. |

### 9.1.11. LightIntensity

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | % |
| **Description** | 충전기 조명을 비출 때 최대 강도의 백분율 |

### 9.1.12. LocalAuthorizeOffline

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | boolean |
| **Description** | 충전기가 오프라인일 때 로컬로 인증된 식별자에 대한 트랜잭션을 시작할지 여부. |

### 9.1.13. LocalPreAuthorize

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | boolean |
| **Description** | 충전기가 온라인일 때 중앙 시스템의 Authorize.conf를 기다리거나 요청하지 않고 로컬로 인증된 식별자에 대한 트랜잭션을 시작할지 여부. |

### 9.1.14. MaxEnergyOnInvalidId

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | Wh |
| **Description** | 트랜잭션 시작 후 중앙 시스템에 의해 식별자가 무효화될 때 Wh 단위로 전달되는 최대 에너지. |

### 9.1.15. MeterValuesAlignedData

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | CSL |
| **Description** | MeterValues.req PDU에 포함될 클럭 정렬 measurand(들), ClockAlignedDataInterval 초마다. 해당하는 경우 Measurand는 선택적 phase와 결합된다. 예: Voltage L1<br>기본값: "Energy.Active.Import.Register" |

### 9.1.16. MeterValuesAlignedDataMaxLength

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | MeterValuesAlignedData 구성 키의 최대 항목 수. |

### 9.1.17. MeterValuesSampledData

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | CSL |
| **Description** | MeterValues.req PDU에 포함될 샘플링된 measurand(들), MeterValueSampleInterval 초마다. 해당하는 경우 Measurand는 선택적 phase와 결합된다. 예: Voltage L1<br>기본값: "Energy.Active.Import.Register" |

### 9.1.18. MeterValuesSampledDataMaxLength

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | MeterValuesSampledData 구성 키의 최대 항목 수. |

### 9.1.19. MeterValueSampleInterval

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | seconds |
| **Description** | 미터링(또는 기타) 데이터의 샘플링 간격으로, "MeterValues" PDU로 전송하도록 의도된다. 충전 세션 데이터(ConnectorId>0)의 경우 샘플은 충전 트랜잭션 시작부터 이 간격으로 주기적으로 획득 및 전송된다.<br>"0" 값(숫자 0)은 관례상 샘플링된 데이터를 전송하지 않아야 함을 의미하는 것으로 해석되어야 한다. |

### 9.1.20. MinimumStatusDuration

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | seconds |
| **Description** | StatusNotification.req PDU가 중앙 시스템에 전송되기 전에 충전기 또는 커넥터 상태가 안정적인 최소 지속 시간. |

### 9.1.21. NumberOfConnectors

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | 이 충전기의 물리적 충전 커넥터 수. |

### 9.1.22. ResetRetries

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | times |
| **Description** | 충전기의 실패한 리셋을 재시도하는 횟수. |

### 9.1.23. StopTransactionOnEVSideDisconnect

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | boolean |
| **Description** | true로 설정되면, 충전기는 케이블이 EV에서 분리될 때 트랜잭션을 관리적으로 중지해야 한다(SHALL). |

### 9.1.24. StopTransactionOnInvalidId

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | boolean |
| **Description** | 충전기가 이 트랜잭션에 대한 StartTransaction.conf에서 Accepted가 아닌 인증 상태를 받았을 때 진행 중인 트랜잭션을 중지할지 여부. |

### 9.1.25. StopTxnAlignedData

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | CSL |
| **Description** | 트랜잭션의 모든 ClockAlignedDataInterval마다 StopTransaction.req MeterValues.req PDU의 TransactionData 요소에 포함될 클럭 정렬 주기적 measurand(들). |

### 9.1.26. StopTxnAlignedDataMaxLength

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | StopTxnAlignedData 구성 키의 최대 항목 수. |

### 9.1.27. StopTxnSampledData

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | CSL |
| **Description** | 충전 세션 시작부터 MeterValueSampleInterval 초마다 StopTransaction.req PDU의 TransactionData 요소에 포함될 샘플링된 measurand(들). |

### 9.1.28. StopTxnSampledDataMaxLength

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | StopTxnSampledData 구성 키의 최대 항목 수. |

### 9.1.29. SupportedFeatureProfiles

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R |
| **Type** | CSL |
| **Description** | 지원되는 Feature Profile 목록. 가능한 프로파일 식별자: Core, FirmwareManagement, LocalAuthListManagement, Reservation, SmartCharging 및 RemoteTrigger. |

### 9.1.30. SupportedFeatureProfilesMaxLength

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | SupportedFeatureProfiles 구성 키의 최대 항목 수. |

### 9.1.31. TransactionMessageAttempts

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | times |
| **Description** | 중앙 시스템이 처리에 실패했을 때 충전기가 트랜잭션 관련 메시지를 제출하려고 시도해야 하는 횟수. |

### 9.1.32. TransactionMessageRetryInterval

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | seconds |
| **Description** | 중앙 시스템이 처리에 실패한 트랜잭션 관련 메시지를 재제출하기 전에 충전기가 대기해야 하는 시간. |

### 9.1.33. UnlockConnectorOnEVSideDisconnect

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | boolean |
| **Description** | true로 설정되면, 충전기는 케이블이 EV에서 분리될 때 충전기 측의 케이블 잠금을 해제해야 한다(SHALL). |

### 9.1.34. WebSocketPingInterval

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | RW |
| **Type** | integer |
| **Unit** | seconds |
| **Description** | websocket 구현에만 관련됨. 0은 클라이언트 측 websocket Ping/Pong을 비활성화한다. 이 경우 ping/pong이 없거나 서버가 ping을 시작하고 클라이언트가 Pong으로 응답한다. 양수 값은 ping 간 초 수로 해석된다. 음수 값은 허용되지 않는다. ChangeConfiguration은 REJECTED 결과를 반환할 것으로 예상된다. |

## 9.2. Local Auth List Management Profile

### 9.2.1. LocalAuthListEnabled

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | RW |
| **Type** | boolean |
| **Description** | Local Authorization List가 활성화되어 있는지 여부. |

### 9.2.2. LocalAuthListMaxLength

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | Local Authorization List에 저장할 수 있는 최대 식별 항목 수. |

### 9.2.3. SendLocalListMaxLength

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | 단일 SendLocalList.req에 전송할 수 있는 최대 식별 항목 수. |

## 9.3. Reservation Profile

### 9.3.1. ReserveConnectorZeroSupported

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | R |
| **Type** | boolean |
| **Description** | 이 구성 키가 존재하고 true로 설정되면: 충전기는 커넥터 0에 대한 예약을 지원한다. |

## 9.4. Smart Charging Profile

### 9.4.1. ChargeProfileMaxStackLevel

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | ChargingProfile의 최대 StackLevel. 정의된 숫자는 또한 Charging Profile Purpose별로 설치 가능한 충전 스케줄의 최대 허용 개수를 나타낸다. |

### 9.4.2. ChargingScheduleAllowedChargingRateUnit

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R |
| **Type** | CSL |
| **Description** | ChargingSchedule에서 사용할 수 있는 지원되는 수량 목록. 허용되는 값: 'Current' 및 'Power' |

### 9.4.3. ChargingScheduleMaxPeriods

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | ChargingSchedule당 정의할 수 있는 최대 기간(period) 수. |

### 9.4.4. ConnectorSwitch3to1PhaseSupported

| 속성 | 값 |
|------|-----|
| **Required/optional** | optional |
| **Accessibility** | R |
| **Type** | boolean |
| **Description** | 정의되고 true인 경우, 이 충전기는 트랜잭션 중 3상에서 1상으로의 전환을 지원한다. |

### 9.4.5. MaxChargingProfilesInstalled

| 속성 | 값 |
|------|-----|
| **Required/optional** | required |
| **Accessibility** | R |
| **Type** | integer |
| **Description** | 한 번에 설치 가능한 최대 Charging Profile 수. |

---

# Appendix A: New in OCPP 1.6 (OCPP 1.6의 새로운 기능)

> 출처: ocpp-1.6 edition 2.pdf, Appendix A
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

OCPP 1.5 [OCPP1.5]와 비교하여 OCPP 1.6에서 다음과 같은 변경 사항이 적용되었다:

- **Smart Charging 추가**
- **전송 프로토콜로 WebSocket을 통한 JSON 바인딩 추가**, 데이터 사용량 감소 및 NAT 라우터를 통한 OCPP 통신 가능, 참조: OCPP JSON Specification
- **ChargePointStatus 열거형에 추가 상태 추가**, CPO 및 최종 사용자에게 충전기의 현재 상태에 대한 더 많은 정보 제공
- **MeterValues.req 구조 변경**, XML 속성 사용 제거, JSON 지원에 필요 (JSON에는 속성 지원 없음)
- **Measurand 열거형에 추가 값 추가**, 충전기 제조업체가 EV의 충전 상태(State of Charge)와 같은 새로운 정보를 중앙 시스템에 전송할 수 있는 가능성 제공
- **TriggerMessage 메시지 추가**, 중앙 시스템이 충전기로부터 정보를 요청할 수 있는 가능성 제공
- **BootNotification.conf에서 사용하는 RegistrationStatus 열거형에 새로운 Pending 멤버 추가**
- **더 많고 명확한 구성 키 추가**, CPO가 충전기에서 다양한 비즈니스 케이스를 구성하는 방법을 더 명확하게 제시
- **메시지와 구성 키를 프로파일로 분할**, OCPP를 점진적으로 또는 부분적으로만 구현하기 쉽게 만듦
- **알려진 모호성 제거** (예: UnlockConnector.req 사용 시기, RemoteStart/Stop 응답 방법, 커넥터 번호 매기기)

## A.1. Updated/New Messages (업데이트/새 메시지)

### BootNotification.req
- IccId와 Imsi를 CiString[]로 변경하여 최대 길이 강제

### BootNotification.conf
- heartbeatInterval을 interval로 변경, interval은 이제 heartbeat 외의 다른 목적으로도 사용됨
- Pending 상태 추가

### ChargePointErrorCode
- 열거값 추가: InternalError, LocalListConfict, UnderVoltage
- 열거값 이름 변경: Mode3Error → EVCommunicationError

### ChargePointStatus
- Occupied 열거값을 더 상세한 값으로 대체: Preparing, Charging, SuspendedEVSE, SuspendedEV, Finishing

### ChargingRateUnitType
- 신규

### ConfigurationStatus
- RebootRequired 열거값 추가

### ClearChargingProfile.req
- 신규

### ClearChargingProfile.conf
- 신규

### DiagnosticsStatus
- Uploading 및 Idle 열거값 추가

### FirmwareStatus
- Downloading, Installing 및 Idle 열거값 추가

### GetCompositeSchedule.req
- 신규

### GetCompositeSchedule.conf
- 신규

### Location
- Cable 및 EV 열거값 추가

### Measurand
- 열거값 추가: Current.Offered, Frequency, Power.Factor, Power.Offered, RPM, SoC

### MeterValues.req
- 복잡한 데이터 구조 개편
- 'phase' 필드 추가

### ReadingContext
- Trigger 및 Other 열거값 추가

### RemoteStartTransaction.req
- ChargingProfile 선택적 필드 추가

### SendLocalList.req
- hash 제거

### SendLocalList.conf
- hash 제거

### SetChargingProfile.req
- 신규

### SetChargingProfile.conf
- 신규

### StatusNotification.req
- 상태 개편
- 새로운 오류 코드
- 커넥터 ID 0은 Available, Unavailable, Faulted 상태만 가질 수 있음

### StopTransaction.req
- 명시적이고 필수적인 중지 사유 추가

### TriggerMessage.req
- 신규

### TriggerMessage.conf
- 신규

### UnlockConnector.conf
- UnlockStatus 열거형 개편

### UnitOfMeasure
- Fahrenheit, K, Percent, VA, kVA 추가
- 이름 변경: Volt → V, Amp → A
