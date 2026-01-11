# 5. Operations Initiated by Central System (중앙 시스템에 의해 시작된 작업)

> 출처: ocpp-1.6 edition 2.pdf, Chapter 5
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

## 5.1. Cancel Reservation (예약 취소)

예약을 취소하기 위해 중앙 시스템은 충전기에 CancelReservation.req PDU를 전송해야 한다(SHALL).

충전기가 요청 PDU의 reservationId와 일치하는 예약을 가지고 있으면, 상태 'Accepted'를 반환해야 한다(SHALL). 그렇지 않으면 'Rejected'를 반환해야 한다(SHALL).

## 5.2. Change Availability (가용성 변경)

중앙 시스템은 충전기에 가용성 변경을 요청할 수 있다. 충전기는 충전 중이거나 충전 준비가 되어 있을 때 사용 가능("작동 중")한 것으로 간주된다. 충전기는 충전을 허용하지 않을 때 사용 불가능한 것으로 간주된다. 중앙 시스템은 충전기에 가용성 변경을 요청하기 위해 ChangeAvailability.req PDU를 전송해야 한다(SHALL). 중앙 시스템은 가용성을 available 또는 unavailable로 변경할 수 있다.

ChangeAvailability.req PDU를 수신하면, 충전기는 ChangeAvailability.conf PDU로 응답해야 한다(SHALL). 응답 PDU는 충전기가 요청된 가용성으로 변경할 수 있는지 여부를 나타내야 한다(SHALL). 트랜잭션이 진행 중일 때 충전기는 가용성 상태 'Scheduled'로 응답하여 트랜잭션이 완료된 후에 발생하도록 예약되었음을 나타내야 한다(SHALL).

중앙 시스템이 충전기에 이미 있는 상태로 변경을 요청하는 경우, 충전기는 가용성 상태 'Accepted'로 응답해야 한다(SHALL).

ChangeAvailability.req PDU로 요청된 가용성 변경이 발생하면, 충전기는 해당 섹션에 설명된 대로 StatusNotification.req로 중앙 시스템에 새로운 가용성 상태를 알려야 한다(SHALL).

> **참고:** ChangeAvailability.req가 ConnectorId = 0을 포함하는 경우, 상태 변경은 충전기 및 모든 커넥터에 적용된다.

> **참고:** 지속 상태: 예를 들어 Unavailable로 설정된 커넥터는 재부팅 후에도 유지되어야 한다.

## 5.3. Change Configuration (구성 변경)

중앙 시스템은 충전기에 구성 매개변수 변경을 요청할 수 있다. 이를 위해 중앙 시스템은 ChangeConfiguration.req를 전송해야 한다(SHALL). 이 요청에는 키-값 쌍이 포함되며, "key"는 변경할 구성 설정의 이름이고 "value"는 구성 설정의 새 값을 포함한다.

ChangeConfiguration.req를 수신하면 충전기는 구성에 변경을 적용할 수 있는지 여부를 나타내는 ChangeConfiguration.conf로 응답해야 한다(SHALL). "key" 및 "value"의 내용은 규정되지 않았다. 충전기는 다음 규칙에 따라 ChangeConfiguration.conf의 status 필드를 설정해야 한다(SHALL):

- 변경이 성공적으로 적용되고 변경이 즉시 유효한 경우, 충전기는 상태 'Accepted'로 응답해야 한다(SHALL).
- 변경이 성공적으로 적용되었지만 유효하게 하려면 재부팅이 필요한 경우, 충전기는 상태 'RebootRequired'로 응답해야 한다(SHALL).
- "key"가 충전기에서 지원하는 구성 설정에 해당하지 않으면, 충전기는 상태 'NotSupported'로 응답해야 한다(SHALL).
- 충전기가 구성을 설정하지 않았고 이전 상태 중 어느 것도 적용되지 않으면, 충전기는 상태 'Rejected'로 응답해야 한다(SHALL).

> **참고:** 충전기가 'Rejected' 상태의 ChangeConfiguration.conf로 응답하는 Change Configuration 요청의 예는 범위를 벗어난 값이 있는 요청 및 예상 형식에 맞지 않는 값이 있는 요청이다.

키 값이 CSL로 정의된 경우, CSL의 최대 길이를 항목 수로 나타내는 [KeyName]MaxLength 키가 함께 제공될 수 있다(MAY). 이 키가 설정되지 않은 경우, 1(하나) 항목의 안전한 값을 가정해야 한다(SHOULD).

## 5.4. Clear Cache (캐시 지우기)

중앙 시스템은 충전기에 Authorization Cache를 지우도록 요청할 수 있다. 중앙 시스템은 충전기의 Authorization Cache를 지우기 위해 ClearCache.req PDU를 전송해야 한다(SHALL).

ClearCache.req PDU를 수신하면, 충전기는 ClearCache.conf PDU로 응답해야 한다(SHALL). 응답 PDU는 충전기가 Authorization Cache를 지울 수 있었는지 여부를 나타내야 한다(SHALL).

## 5.5. Clear Charging Profile (충전 프로파일 지우기)

중앙 시스템이 이전에 충전기로 전송한 일부 또는 모든 충전 프로파일을 지우려는 경우, ClearChargingProfile.req PDU를 사용해야 한다(SHALL).

충전기는 요청을 처리할 수 있었는지 여부를 지정하는 ClearChargingProfile.conf PDU로 응답해야 한다(SHALL).

## 5.6. Data Transfer (데이터 전송)

중앙 시스템이 OCPP에서 지원하지 않는 기능에 대한 정보를 충전기로 전송해야 하는 경우, DataTransfer.req PDU를 사용해야 한다(SHALL).

이 작업의 동작은 충전기에 의해 시작된 Data Transfer 작업과 동일하다. 자세한 내용은 Data Transfer를 참조하라.

## 5.7. Get Composite Schedule (복합 스케줄 가져오기)

중앙 시스템은 GetCompositeSchedule.req PDU를 전송하여 충전기에 복합 충전 스케줄을 보고하도록 요청할 수 있다(MAY). GetCompositeSchedule.conf PDU에 보고된 스케줄은 충전기에 있는 모든 활성 스케줄과 가능한 로컬 제한의 계산 결과이다. 로컬 제한이 고려될 수 있다.

GetCompositeSchedule.req를 수신하면, 충전기는 요청 PDU가 수신된 순간(시간 X)부터 X + Duration까지 복합 충전 스케줄 간격을 계산하고, GetCompositeSchedule.conf PDU로 중앙 시스템에 전송해야 한다(SHALL).

요청의 ConnectorId가 '0'으로 설정된 경우, 충전기는 요청된 시간 동안 충전기가 그리드에서 소비할 것으로 예상되는 총 전력 또는 전류를 보고해야 한다(SHALL).

> **참고:** 충전기가 전송한 충전 스케줄은 해당 시점에 대해서만 표시되는 것이다. 이 스케줄은 외부 원인(예: 그리드 연결 용량을 기반으로 한 로컬 밸런싱이 활성화되어 있고 하나의 커넥터가 사용 가능해짐)으로 인해 시간이 지남에 따라 변경될 수 있다.

충전기가 요청된 스케줄을 보고할 수 없는 경우(예: connectorId를 알 수 없는 경우), 상태 Rejected로 응답해야 한다(SHALL).

## 5.8. Get Configuration (구성 가져오기)

구성 설정의 값을 검색하기 위해, 중앙 시스템은 충전기에 GetConfiguration.req PDU를 전송해야 한다(SHALL).

요청 PDU의 키 목록이 비어 있거나 누락된 경우(선택 사항임), 충전기는 GetConfiguration.conf에서 모든 구성 설정 목록을 반환해야 한다(SHALL). 그렇지 않으면 충전기는 인식된 키 및 해당 값과 읽기 전용 상태의 목록을 반환해야 한다(SHALL). 인식되지 않은 키는 GetConfiguration.conf의 선택적 unknown key list 요소의 일부로 응답 PDU에 배치되어야 한다(SHALL).

단일 PDU에서 요청되는 구성 키의 수는 충전기에 의해 제한될 수 있다(MAY). 이 최대값은 구성 키 GetConfigurationMaxKeys를 읽어 검색할 수 있다.

## 5.9. Get Diagnostics (진단 정보 가져오기)

중앙 시스템은 충전기에 진단 정보를 요청할 수 있다. 중앙 시스템은 충전기의 진단 정보를 가져오기 위해 GetDiagnostics.req PDU를 전송해야 하며(SHALL), 충전기가 진단 데이터를 업로드해야 하는 위치와 선택적으로 요청된 진단 정보의 시작 및 종료 시간을 포함한다.

GetDiagnostics.req PDU를 수신하고 진단 정보가 있는 경우, 충전기는 업로드될 진단 정보를 포함하는 파일의 이름을 명시하는 GetDiagnostics.conf PDU로 응답해야 한다(SHALL). 충전기는 단일 파일을 업로드해야 한다(SHALL). 진단 파일의 형식은 규정되지 않았다. 진단 파일이 없는 경우, GetDiagnostics.conf는 파일 이름을 포함해서는 안 된다(SHALL NOT).

진단 파일을 업로드하는 동안, 충전기는 중앙 시스템에 업로드 프로세스의 상태를 업데이트하기 위해 DiagnosticsStatusNotification.req PDU를 전송해야 한다(MUST).

## 5.10. Get Local List Version (로컬 목록 버전 가져오기)

Local Authorization List의 동기화를 지원하기 위해, 중앙 시스템은 충전기에 Local Authorization List의 버전 번호를 요청할 수 있다. 중앙 시스템은 이 값을 요청하기 위해 GetLocalListVersion.req PDU를 전송해야 한다(SHALL).

GetLocalListVersion.req PDU를 수신하면 충전기는 Local Authorization List의 버전 번호를 포함하는 GetLocalListVersion.conf PDU로 응답해야 한다(SHALL). 버전 번호 0(영)은 로컬 인증 목록이 비어 있음을 나타내는 데 사용되어야 하며(SHALL), 버전 번호 -1은 충전기가 Local Authorization List를 지원하지 않음을 나타내는 데 사용되어야 한다(SHALL).

## 5.11. Remote Start Transaction (원격 트랜잭션 시작)

중앙 시스템은 RemoteStartTransaction.req를 전송하여 충전기에 트랜잭션 시작을 요청할 수 있다. 수신 시, 충전기는 RemoteStartTransaction.conf로 응답해야 하며(SHALL) 요청을 수락하고 트랜잭션 시작을 시도할 것인지를 나타내는 상태를 포함해야 한다.

RemoteStartTransaction.req 메시지의 효과는 충전기의 AuthorizeRemoteTxRequests 구성 키 값에 따라 달라진다.

- AuthorizeRemoteTxRequests 값이 true인 경우, 충전기는 RemoteStartTransaction.req 메시지에 제공된 idTag로 트랜잭션을 시작하기 위한 충전기의 로컬 동작에 대한 응답인 것처럼 동작해야 한다(SHALL). 이는 충전기가 Local Authorization List, Authorization Cache 및/또는 Authorize.req 요청을 사용하여 먼저 idTag를 인증하려고 시도함을 의미한다. 트랜잭션은 인증을 받은 후에만 시작된다.

- AuthorizeRemoteTxRequests 값이 false인 경우, 충전기는 RemoteStartTransaction.req 메시지에 제공된 idTag에 대한 트랜잭션을 즉시 시작하려고 시도해야 한다(SHALL). 트랜잭션이 시작된 후, 충전기는 중앙 시스템에 StartTransaction 요청을 보내고, 중앙 시스템은 이 StartTransaction 요청을 처리할 때 idTag의 인증 상태를 확인한다는 점에 유의하라.

Remote Start Transaction의 일반적인 사용 사례는 다음과 같다:
- CPO 운영자가 트랜잭션 시작에 문제가 있는 EV 운전자를 지원할 수 있도록 함.
- 모바일 앱이 중앙 시스템을 통해 충전 트랜잭션을 제어할 수 있도록 함.
- SMS를 사용하여 중앙 시스템을 통해 충전 트랜잭션을 제어할 수 있도록 함.

RemoteStartTransaction.req는 식별자(idTag)를 포함해야 하며(SHALL), 충전기가 트랜잭션을 시작할 수 있는 경우 이를 사용하여 중앙 시스템에 StartTransaction.req를 보낸다. 트랜잭션은 StartTransaction에 설명된 것과 동일한 방식으로 시작된다. RemoteStartTransaction.req는 트랜잭션이 특정 커넥터에서 시작되어야 하는 경우 커넥터 id를 포함할 수 있다(MAY). 커넥터 id가 제공되지 않으면, 충전기가 커넥터 선택을 제어한다. 충전기는 커넥터 id가 없는 RemoteStartTransaction.req를 거부할 수 있다(MAY).

중앙 시스템은 RemoteStartTransaction 요청에 ChargingProfile을 포함할 수 있다(MAY). 이 ChargingProfile의 목적은 TxProfile로 설정되어야 한다(SHALL). 수락되면, 충전기는 트랜잭션에 대해 이 ChargingProfile을 사용해야 한다(SHALL).

> **참고:** Smart Charging을 지원하지 않는 충전기가 Charging Profile이 있는 RemoteStartTransaction.req를 수신하면, 이 매개변수는 무시되어야 한다(SHOULD).

## 5.12. Remote Stop Transaction (원격 트랜잭션 중지)

중앙 시스템은 트랜잭션의 식별자와 함께 RemoteStopTransaction.req를 충전기로 전송하여 충전기에 트랜잭션 중지를 요청할 수 있다. 충전기는 RemoteStopTransaction.conf로 응답해야 하며(SHALL) 요청을 수락했는지 여부와 주어진 transactionId를 가진 트랜잭션이 진행 중이며 중지될 것인지를 나타내는 상태를 포함해야 한다.

트랜잭션을 중지하는 이 원격 요청은 트랜잭션을 중지하는 로컬 동작과 동일하다. 따라서 트랜잭션이 중지되어야 하며(SHALL), 충전기는 StopTransaction.req를 전송해야 하고(SHALL), 해당하는 경우 커넥터를 잠금 해제해야 한다.

Remote Stop Transaction의 두 가지 주요 사용 사례는 다음과 같다:

- CPO 운영자가 트랜잭션 중지에 문제가 있는 EV 운전자를 지원할 수 있도록 함.
- 모바일 앱이 중앙 시스템을 통해 충전 트랜잭션을 제어할 수 있도록 함.

## 5.13. Reserve Now (지금 예약)

중앙 시스템은 충전기에 ReserveNow.req를 발행하여 특정 idTag가 사용할 커넥터를 예약할 수 있다.

예약을 요청하기 위해 중앙 시스템은 충전기에 ReserveNow.req PDU를 전송해야 한다(SHALL). 중앙 시스템은 예약할 커넥터를 지정할 수 있다(MAY). ReserveNow.req PDU를 수신하면, 충전기는 ReserveNow.conf PDU로 응답해야 한다(SHALL).

요청의 reservationId가 충전기의 예약과 일치하면, 충전기는 해당 예약을 요청의 새 예약으로 교체해야 한다(SHALL).

reservationId가 충전기의 어떤 예약과도 일치하지 않으면, 충전기는 커넥터 예약에 성공한 경우 상태 값 'Accepted'를 반환해야 한다(SHALL). 충전기 또는 지정된 커넥터가 점유 중인 경우 충전기는 'Occupied'를 반환해야 한다(SHALL). 충전기 또는 커넥터가 동일하거나 다른 idTag에 대해 예약된 경우에도 충전기는 'Occupied'를 반환해야 한다(SHALL). 충전기 또는 커넥터가 Faulted 상태인 경우 충전기는 'Faulted'를 반환해야 한다(SHALL). 충전기 또는 커넥터가 Unavailable 상태인 경우 충전기는 'Unavailable'을 반환해야 한다(SHALL). 충전기가 예약을 수락하지 않도록 구성된 경우 충전기는 'Rejected'를 반환해야 한다(SHALL).

충전기가 예약 요청을 수락하면, 들어오는 idTag 또는 parent idTag가 예약의 idTag 또는 parent idTag와 일치하는 경우를 제외하고 예약된 커넥터의 모든 들어오는 idTag에 대한 충전을 거부해야 한다(SHALL).

구성 키 ReserveConnectorZeroSupported가 true로 설정된 경우, 충전기는 커넥터 0에 대한 예약을 지원한다. 예약 요청의 connectorId가 0이면, 충전기는 특정 커넥터를 예약해서는 안 되며(SHALL NOT), 예약 유효 기간 동안 언제든지 하나의 커넥터가 예약된 idTag에 대해 사용 가능한 상태로 유지되도록 해야 한다(SHALL). 구성 키 ReserveConnectorZeroSupported가 설정되지 않았거나 false로 설정된 경우, 충전기는 'Rejected'를 반환해야 한다(SHALL).

예약의 parent idTag에 값이 있는 경우(선택 사항), 들어오는 idTag와 연결된 parent idTag를 결정하기 위해 충전기는 Local Authorization List 또는 Authorization Cache에서 조회할 수 있다(MAY). Local Authorization List 또는 Authorization Cache에서 찾을 수 없으면, 충전기는 들어오는 idTag에 대해 중앙 시스템에 Authorize.req를 전송해야 한다(SHALL). Authorize.conf 응답은 parent-id를 포함할 수 있다(MAY).

예약은 (1) 예약된 idTag 또는 parent idTag에 대한 트랜잭션이 예약된 커넥터 또는 예약된 connectorId가 0일 때 임의의 커넥터에서 시작되거나, (2) expiryDate에 지정된 시간에 도달하거나, (3) 충전기 또는 커넥터가 Faulted 또는 Unavailable로 설정될 때 충전기에서 종료되어야 한다(SHALL).

예약된 idTag에 대한 트랜잭션이 시작되면, 충전기는 예약이 종료되었음을 중앙 시스템에 알리기 위해 StartTransaction.req PDU에 reservationId를 포함하여 전송해야 한다(SHALL)(Start Transaction 참조).

예약이 만료되면, 충전기는 예약을 종료하고 커넥터를 사용 가능하게 만들어야 한다(SHALL). 충전기는 예약된 커넥터가 이제 사용 가능함을 중앙 시스템에 알리기 위해 상태 알림을 전송해야 한다(SHALL).

충전기가 Authorization Cache를 구현한 경우, ReserveNow.conf PDU를 수신하면 충전기는 idTag가 Local Authorization List에 없는 경우, Authorization Cache에 설명된 대로 응답의 IdTagInfo 값으로 캐시 항목을 업데이트해야 한다(SHALL).

> **참고:** ReserveNow.req를 수신한 후 트랜잭션 시작 전에 authorize.req로 식별자를 검증하는 것이 권장된다(RECOMMENDED).

## 5.14. Reset (재설정)

중앙 시스템은 충전기에 자체 재설정을 요청하기 위해 Reset.req PDU를 전송해야 한다(SHALL). 중앙 시스템은 하드 또는 소프트 재설정을 요청할 수 있다. Reset.req PDU를 수신하면, 충전기는 Reset.conf PDU로 응답해야 한다(SHALL). 응답 PDU는 충전기가 자체 재설정을 시도할 것인지 여부를 포함해야 한다(SHALL).

Reset.req를 수신한 후, 충전기는 재설정을 수행하기 전에 진행 중인 모든 트랜잭션에 대해 StopTransaction.req를 전송해야 한다(SHALL). 충전기가 중앙 시스템으로부터 StopTransaction.conf를 수신하지 못하면, StopTransaction.req를 대기열에 넣어야 한다.

소프트 재설정을 수신하면, 충전기는 진행 중인 트랜잭션을 정상적으로 중지하고 진행 중인 모든 트랜잭션에 대해 StopTransaction.req를 전송해야 한다(SHALL). 그런 다음 애플리케이션 소프트웨어를 재시작해야 한다(가능하지 않으면 프로세서/컨트롤러를 재시작).

하드 재설정을 수신하면 충전기는 (모든) 하드웨어를 재시작해야 하며(SHALL), 진행 중인 트랜잭션을 정상적으로 중지할 필요는 없다. 가능한 경우 충전기는 재시작한 후 BootNotification.conf를 통해 중앙 시스템에 수락된 후 이전에 진행 중이던 트랜잭션에 대해 StopTransaction.req를 전송한다. 이는 올바르게 작동하지 않는 충전기를 위한 최후의 수단 솔루션으로, "하드" 재설정을 전송하면 (대기열에 있는) 정보가 손실될 수 있다.

> **참고:** 지속 상태: 예를 들어 Unavailable로 설정된 커넥터는 유지되어야 한다.

## 5.15. Send Local List (로컬 목록 전송)

중앙 시스템은 충전기가 idTag 인증에 사용할 수 있는 Local Authorization List를 전송할 수 있다. 목록은 충전기의 현재 목록을 대체하는 전체 목록이거나 충전기의 현재 목록에 적용할 업데이트가 있는 차등 목록일 수 있다(MAY).

중앙 시스템은 충전기에 목록을 전송하기 위해 SendLocalList.req PDU를 전송해야 한다(SHALL). SendLocalList.req PDU는 업데이트 유형(전체 또는 차등)과 충전기가 업데이트된 후 로컬 인증 목록과 연결해야 하는 버전 번호를 포함해야 한다(SHALL).

SendLocalList.req PDU를 수신하면, 충전기는 SendLocalList.conf PDU로 응답해야 한다(SHALL). 응답 PDU는 충전기가 로컬 인증 목록의 업데이트를 수락했는지 여부를 나타내야 한다(SHALL). 상태가 Failed 또는 VersionMismatch이고 updateType이 Differential인 경우, 중앙 시스템은 updateType Full로 전체 로컬 인증 목록을 다시 전송해야 한다(SHOULD).

## 5.16. Set Charging Profile (충전 프로파일 설정)

중앙 시스템은 다음 상황에서 충전 프로파일을 설정하기 위해 충전기에 SetChargingProfile.req를 전송할 수 있다:

- 트랜잭션 시작 시 트랜잭션에 대한 충전 프로파일을 설정하기 위해;
- 충전기로 전송된 RemoteStartTransaction.req에서;
- 트랜잭션 중에 트랜잭션의 활성 프로파일을 변경하기 위해;
- 트랜잭션 컨텍스트 외부에서 별도의 메시지로 로컬 컨트롤러, 충전기 또는 커넥터에 기본 충전 프로파일을 설정하기 위해.

> **참고:** 트랜잭션과 TxProfile 간의 불일치를 방지하기 위해, 프로파일이 특정 트랜잭션에 적용되는 경우 중앙 시스템은 SetChargingProfile.req에 transactionId를 포함해야 한다(SHALL).

이러한 상황은 아래에 설명되어 있다.

### 5.16.1. Setting a charging profile at start of transaction (트랜잭션 시작 시 충전 프로파일 설정)

중앙 시스템이 StartTransaction.req를 수신하면 중앙 시스템은 StartTransaction.conf로 응답해야 한다(SHALL). 충전 프로파일이 필요한 경우, 중앙 시스템은 충전기에 SetChargingProfile.req를 전송하도록 선택할 수 있다(MAY).

트랜잭션이 여전히 진행 중일 가능성이 있는지 확인하기 위해 충전 프로파일을 전송하기 전에 StartTransaction.req PDU의 타임스탬프를 확인하는 것이 권장된다(RECOMMENDED). StartTransaction.req는 오프라인 기간 동안 캐시되었을 수 있다.

### 5.16.2. Setting a charge profile in a RemoteStartTransaction request (RemoteStartTransaction 요청에서 충전 프로파일 설정)

중앙 시스템은 RemoteStartTransaction 요청에 충전 프로파일을 포함할 수 있다(MAY).

중앙 시스템이 ChargingProfile을 포함하는 경우, ChargingProfilePurpose는 TxProfile로 설정되어야 하며(MUST) transactionId는 설정되어서는 안 된다(SHALL NOT).

> **참고:** 충전기는 주어진 프로파일을 새로 시작된 트랜잭션에 적용해야 한다(SHALL). 이 트랜잭션은 StartTransaction.conf를 통해 중앙 시스템이 할당한 transactionId를 받는다. 충전기가 이 트랜잭션의 transactionId와 함께 RemoteStartTransaction.req에 주어진 프로파일과 동일한 StackLevel을 가진 SetChargingProfile.req를 수신하면, 충전기는 기존 충전 프로파일을 교체해야 하며(SHALL), 그렇지 않으면 이미 존재하는 프로파일 옆에 프로파일을 설치/스택해야 한다(SHALL).

### 5.16.3. Setting a charging profile during a transaction (트랜잭션 중 충전 프로파일 설정)

중앙 시스템은 해당 트랜잭션의 충전 프로파일을 업데이트하기 위해 충전기에 충전 프로파일을 전송할 수 있다(MAY). 중앙 시스템은 이를 위해 SetChargingProfile.req PDU를 사용해야 한다(SHALL). 동일한 chargingProfileId 또는 동일한 stackLevel / ChargingProfilePurpose 조합을 가진 충전 프로파일이 충전기에 존재하는 경우, 새 충전 프로파일은 기존 충전 프로파일을 교체해야 하며(SHALL), 그렇지 않으면 추가되어야 한다(SHALL).

그런 다음 충전기는 어떤 충전 프로파일이 활성화될지 결정하기 위해 충전 프로파일 모음을 재평가해야 한다(SHALL). 업데이트된 충전 프로파일이 현재 트랜잭션에만 적용되도록 하려면, ChargingProfile의 chargingProfilePurpose를 TxProfile로 설정해야 한다(MUST). (Charging Profile Purposes 섹션 참조)

### 5.16.4. Setting a charging profile outside of a transaction (트랜잭션 외부에서 충전 프로파일 설정)

중앙 시스템은 기본 충전 프로파일로 사용될 충전 프로파일을 충전기로 전송할 수 있다(MAY). 중앙 시스템은 이를 위해 SetChargingProfile.req PDU를 사용해야 한다(SHALL). 이러한 충전 프로파일은 언제든지 전송될 수 있다(MAY). 동일한 chargingProfileId 또는 동일한 stackLevel / ChargingProfilePurpose 조합을 가진 충전 프로파일이 충전기에 존재하는 경우, 새 충전 프로파일은 기존 충전 프로파일을 교체해야 하며(SHALL), 그렇지 않으면 추가되어야 한다(SHALL). 그런 다음 충전기는 어떤 충전 프로파일이 활성화될지 결정하기 위해 충전 프로파일 모음을 재평가해야 한다(SHALL).

> **참고:** 활성 트랜잭션이 없거나 트랜잭션보다 먼저 목적이 TxProfile로 설정된 ChargingProfile을 설정하는 것은 불가능하다.

> **참고:** 실행 중에 ChargingProfile이 갱신되는 경우, ChargingProfile 사이에 기본 충전 동작이 없도록 새 ChargingProfile의 startSchedule을 과거로 설정하는 것이 권장된다. 충전기는 새 ChargingProfile이 설치될 때까지 기존 ChargingProfile을 계속 실행해야 한다(SHALL).

> **참고:** chargingSchedulePeriod가 duration보다 길면, 나머지 기간은 실행되어서는 안 된다(SHALL NOT). duration이 chargingSchedulePeriod보다 길면, 충전기는 duration이 종료될 때까지 마지막 chargingSchedulePeriod의 값을 유지해야 한다(SHALL).

> **참고:** recurrencyKind가 반복 기간 duration보다 긴 chargingSchedulePeriod 및/또는 duration과 함께 사용되는 경우, 나머지 기간은 실행되어서는 안 된다(SHALL NOT).

> **참고:** chargingSchedule의 첫 번째 chargingSchedulePeriod의 StartSchedule는 항상 0이어야 한다(SHALL).

> **참고:** recurrencyKind가 recurrencyKind 기간보다 짧은 chargingSchedule duration과 함께 사용되는 경우, 충전기는 chargingSchedule duration이 종료된 후 기본 동작으로 되돌아가야 한다(SHALL). 이 되돌아가기는 충전기가 사용 가능한 경우 더 낮은 stackLevel을 가진 ChargingProfile을 사용해야 함을 의미한다(SHALL). 다른 ChargingProfile을 사용할 수 없는 경우, 충전기는 ChargingProfile이 설치되지 않은 것처럼 충전을 허용해야 한다(SHALL). chargingSchedulePeriod 및/또는 duration이 반복 기간 duration보다 길면, 나머지 기간은 실행되어서는 안 된다(SHALL NOT).

## 5.17. Trigger Message (메시지 트리거)

정상 작동 중에 충전기는 중앙 시스템에 상태 및 관련 발생 사항을 알린다. 보고할 내용이 없으면 충전기는 최소한 미리 정의된 간격으로 heartBeat를 전송한다. 정상적인 상황에서 이것은 문제없지만, 중앙 시스템이 (어떤 이유로든) 마지막으로 알려진 상태를 의심할 이유가 있는 경우 어떻게 해야 할까? 펌웨어 업데이트가 진행 중이고 이에 대해 수신한 마지막 상태 알림이 합리적으로 예상할 수 있는 것보다 훨씬 오래 전이었다면 중앙 시스템은 무엇을 할 수 있을까? 진단 요청의 진행 상황에 대해서도 같은 질문을 할 수 있다. 이러한 상황의 문제는 필요한 정보가 기존 메시지에 포함되어 있지 않다는 것이 아니라, 문제가 엄격히 타이밍 문제라는 것이다. 충전기는 정보를 가지고 있지만 중앙 시스템이 업데이트를 원한다는 것을 알 방법이 없다.

TriggerMessage.req는 중앙 시스템이 충전기에 충전기 시작 메시지를 전송하도록 요청할 수 있게 한다. 요청에서 중앙 시스템은 수신하려는 메시지를 나타낸다. 이러한 각 요청 메시지에 대해 중앙 시스템은 이 요청이 적용되는 커넥터를 선택적으로 나타낼 수 있다(MAY). 요청된 메시지가 우선한다: 지정된 connectorId가 메시지와 관련이 없는 경우 무시해야 한다. 이러한 경우 요청된 메시지는 여전히 전송되어야 한다.

반대로, connectorId가 관련이 있지만 없는 경우, 이는 "모든 허용된 connectorId 값에 대해"로 해석되어야 한다. 예를 들어, connectorId 0에 대한 statusNotification 요청은 충전기 상태에 대한 요청이다. connectorId가 없는 statusNotification 요청은 여러 statusNotification에 대한 요청이다: 충전기 자체에 대한 알림과 각 커넥터에 대한 알림.

충전기는 요청된 메시지를 전송하기 전에 먼저 TriggerMessage 응답을 전송해야 한다(SHALL). TriggerMessage.conf에서 충전기는 ACCEPTED 또는 REJECTED를 반환하여 전송 여부를 나타내야 한다(SHALL). 전송 요청을 수락할지 거부할지는 충전기에 달려 있다. 요청된 메시지를 알 수 없거나 구현되지 않은 경우 충전기는 NOT_IMPLEMENTED를 반환해야 한다(SHALL).

충전기가 수락된 것으로 표시한 메시지는 전송되어야 한다(SHOULD). 요청을 수락하고 실제로 요청된 메시지를 전송하는 사이에 정상 작동으로 인해 동일한 메시지가 전송되는 상황이 발생할 수 있다. 이러한 경우 방금 전송된 메시지가 요청을 준수하는 것으로 간주될 수 있다(MAY).

TriggerMessage 메커니즘은 과거 데이터를 검색하기 위한 것이 아니다. 트리거하는 메시지는 현재 정보만 제공해야 한다. 예를 들어 이러한 방식으로 트리거된 MeterValues.req 메시지는 구성 키 MeterValuesSampledData에 구성된 모든 측정값에 대한 가장 최근 측정값을 반환해야 한다(SHALL).

StartTransaction 및 StopTransaction은 상태와 관련이 없고 본질적으로 전환을 설명하기 때문에 이 메커니즘에서 제외되었다.

## 5.18. Unlock Connector (커넥터 잠금 해제)

중앙 시스템은 충전기에 커넥터 잠금 해제를 요청할 수 있다. 이를 위해 중앙 시스템은 UnlockConnector.req PDU를 전송해야 한다(SHALL).

이 메시지의 목적: 커넥터 케이블 고정 장치의 오작동으로 충전기에서 케이블을 뽑는 데 문제가 있는 EV 운전자를 돕는 것이다. EV 운전자가 CPO 헬프 데스크에 전화할 때, 운영자는 충전기에 UnlockConnector.req 전송을 수동으로 트리거하여 커넥터를 잠금 해제하는 새로운 시도를 강제할 수 있다. 이번에는 커넥터가 잠금 해제되고 EV 운전자가 케이블을 뽑고 운전할 수 있기를 바란다.

UnlockConnector.req는 실행 중인 트랜잭션을 원격으로 중지하는 데 사용되어서는 안 된다(SHOULD NOT). 대신 Remote Stop Transaction을 사용하라.

UnlockConnector.req PDU를 수신하면, 충전기는 UnlockConnector.conf PDU로 응답해야 한다(SHALL). 응답 PDU는 충전기가 커넥터를 잠금 해제할 수 있었는지 여부를 나타내야 한다(SHALL).

특정 커넥터에서 트랜잭션이 진행 중이었다면, 충전기는 Stop Transaction에 설명된 대로 먼저 트랜잭션을 완료해야 한다(SHALL).

> **참고:** UnlockConnector.req는 커넥터의 케이블 고정 잠금장치를 잠금 해제하기 위한 것이며, 커넥터 접근 도어를 잠금 해제하기 위한 것이 아니다.

## 5.19. Update Firmware (펌웨어 업데이트)

중앙 시스템은 충전기에 펌웨어를 업데이트해야 함을 알릴 수 있다. 중앙 시스템은 충전기에 새 펌웨어를 설치하도록 지시하기 위해 UpdateFirmware.req PDU를 전송해야 한다(SHALL). PDU는 충전기가 새 펌웨어를 검색할 수 있는 날짜 및 시간과 펌웨어를 다운로드할 수 있는 위치를 포함해야 한다(SHALL).

UpdateFirmware.req PDU를 수신하면, 충전기는 UpdateFirmware.conf PDU로 응답해야 한다(SHALL). 충전기는 retrieve-date 이후 가능한 한 빨리 펌웨어 검색을 시작해야 한다(SHOULD).

펌웨어 다운로드 및 설치 중에, 충전기는 업데이트 프로세스의 상태로 중앙 시스템을 업데이트하기 위해 FirmwareStatusNotification.req PDU를 전송해야 한다(MUST).

충전기는 새 펌웨어 이미지가 "유효한" 경우, 가능한 한 빨리 새 펌웨어를 설치해야 한다(SHALL).

펌웨어 설치 중에 충전을 계속할 수 없는 경우, 설치를 시작하기 전에 충전 세션이 종료될 때까지(충전기 유휴) 기다리는 것이 권장된다(RECOMMENDED). 세션이 종료되기를 기다리는 동안 사용 중이 아닌 커넥터를 UNAVAILABLE로 설정하는 것이 권장된다(RECOMMENDED).

> **참고:** 위 시퀀스 다이어그램은 예시이다. 새 펌웨어가 부팅되고 중앙 시스템에 연결할 수 있는지 확인하기 위해 상태: Installed를 전송하기 전에 먼저 충전기를 재부팅하는 것이 좋은 관행이다. 이것은 요구 사항이 아니다.

