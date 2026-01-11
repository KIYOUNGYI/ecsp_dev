# 6. Messages (메시지)

> 출처: ocpp-1.6 edition 2.pdf, Chapter 6
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

## 6.1. Authorize.req

이것은 충전기가 중앙 시스템으로 전송하는 Authorize.req PDU의 필드 정의를 포함한다. Authorize도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| idTag | IdToken | 1..1 | Required. 인증이 필요한 식별자를 포함한다. |

## 6.2. Authorize.conf

이것은 Authorize.req PDU에 대한 응답으로 중앙 시스템이 충전기로 전송하는 Authorize.conf PDU의 필드 정의를 포함한다. Authorize도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| idTagInfo | IdTagInfo | 1..1 | Required. 인증 상태, 만료 및 parent id에 대한 정보를 포함한다. |

## 6.3. BootNotification.req

이것은 충전기가 중앙 시스템으로 전송하는 BootNotification.req PDU의 필드 정의를 포함한다. Boot Notification도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| chargeBoxSerialNumber | CiString25Type | 0..1 | Optional. 충전기 내부의 Charge Box의 일련번호를 식별하는 값을 포함한다. 사용 중단됨, 향후 버전에서 제거될 예정 |
| chargePointModel | CiString20Type | 1..1 | Required. 충전기의 모델을 식별하는 값을 포함한다. |
| chargePointSerialNumber | CiString25Type | 0..1 | Optional. 충전기의 일련번호를 식별하는 값을 포함한다. |
| chargePointVendor | CiString20Type | 1..1 | Required. 충전기의 벤더를 식별하는 값을 포함한다. |
| firmwareVersion | CiString50Type | 0..1 | Optional. 충전기의 펌웨어 버전을 포함한다. |
| iccid | CiString20Type | 0..1 | Optional. 모뎀 SIM 카드의 ICCID를 포함한다. |
| imsi | CiString20Type | 0..1 | Optional. 모뎀 SIM 카드의 IMSI를 포함한다. |
| meterSerialNumber | CiString25Type | 0..1 | Optional. 충전기의 주요 전기 미터 일련번호를 포함한다. |
| meterType | CiString25Type | 0..1 | Optional. 충전기의 주요 전기 미터 유형을 포함한다. |

## 6.4. BootNotification.conf

이것은 BootNotification.req PDU에 대한 응답으로 중앙 시스템이 충전기로 전송하는 BootNotification.conf PDU의 필드 정의를 포함한다. Boot Notification도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| currentTime | dateTime | 1..1 | Required. 중앙 시스템의 현재 시간을 포함한다. |
| interval | integer | 1..1 | Required. RegistrationStatus가 Accepted인 경우, heartbeat 간격을 초 단위로 포함한다. 중앙 시스템이 Accepted 이외의 값을 반환하는 경우, interval 필드의 값은 다음 BootNotification 요청을 보내기 전의 최소 대기 시간을 나타낸다. |
| status | RegistrationStatus | 1..1 | Required. 충전기가 중앙 시스템에 등록되었는지 여부를 포함한다. |

## 6.5. CancelReservation.req

이것은 중앙 시스템이 충전기로 전송하는 CancelReservation.req PDU의 필드 정의를 포함한다. Cancel Reservation도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| reservationId | integer | 1..1 | Required. 취소할 예약의 Id. |

## 6.6. CancelReservation.conf

이것은 CancelReservation.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 CancelReservation.conf PDU의 필드 정의를 포함한다. Cancel Reservation도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | CancelReservationStatus | 1..1 | Required. 중앙 시스템의 예약 취소 성공 또는 실패를 나타낸다. |

## 6.7. ChangeAvailability.req

이것은 중앙 시스템이 충전기로 전송하는 ChangeAvailability.req PDU의 필드 정의를 포함한다. Change Availability도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| connectorId | integer<br>connectorId >= 0 | 1..1 | Required. 가용성 변경이 필요한 커넥터의 id. Id '0'(영)은 충전기와 모든 커넥터의 가용성이 변경되어야 하는 경우 사용된다. |
| type | AvailabilityType | 1..1 | Required. 충전기가 수행해야 하는 가용성 변경 유형을 포함한다. |

## 6.8. ChangeAvailability.conf

이것은 충전기가 중앙 시스템으로 반환하는 ChangeAvailability.conf PDU의 필드 정의를 포함한다. Change Availability도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | AvailabilityStatus | 1..1 | Required. 충전기가 가용성 변경을 수행할 수 있는지 여부를 나타낸다. |

## 6.9. ChangeConfiguration.req

이것은 중앙 시스템이 충전기로 전송하는 ChangeConfiguration.req PDU의 필드 정의를 포함한다. 'key'와 'value' 필드의 내용과 의미는 충전기와 중앙 시스템 간에 합의되는 것이 권장된다(RECOMMENDED). Change Configuration도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| key | CiString50Type | 1..1 | Required. 변경할 구성 설정의 이름.<br><br>표준 구성 키 이름 및 관련 값은 참조 |
| value | CiString500Type | 1..1 | Required. 설정의 새 값을 문자열로 표현.<br><br>표준 구성 키 이름 및 관련 값은 참조 |

## 6.10. ChangeConfiguration.conf

이것은 충전기가 중앙 시스템으로 반환하는 ChangeConfiguration.conf PDU의 필드 정의를 포함한다. Change Configuration도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | ConfigurationStatus | 1..1 | Required. 구성 변경이 수락되었는지 여부를 반환한다. |

## 6.11. ClearCache.req

이것은 중앙 시스템이 충전기로 전송하는 ClearCache.req PDU의 필드 정의를 포함한다. Clear Cache도 참조하라.

필드가 정의되지 않음.

## 6.12. ClearCache.conf

이것은 ClearCache.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 ClearCache.conf PDU의 필드 정의를 포함한다. Clear Cache도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | ClearCacheStatus | 1..1 | Required. 충전기가 요청을 실행한 경우 Accepted, 그렇지 않으면 Rejected. |

## 6.13. ClearChargingProfile.req

이것은 중앙 시스템이 충전기로 전송하는 ClearChargingProfile.req PDU의 필드 정의를 포함한다.

중앙 시스템은 이 메시지를 사용하여 특정 충전 프로필(id로 표시됨) 또는 선택적 connectorId, stackLevel 및 chargingProfilePurpose 필드의 값과 일치하는 충전 프로필 선택을 지우거나(제거) 할 수 있다. Clear Charging Profile도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| id | integer | 0..1 | Optional. 지울 충전 프로필의 ID. |
| connectorId | integer | 0..1 | Optional. 충전 프로필을 지울 커넥터의 ID를 지정한다. connectorId가 0(영)이면 전체 충전기에 대한 충전 프로필을 지정한다. 이 매개변수가 없으면 요청의 다른 기준과 일치하는 모든 충전 프로필에 지우기가 적용된다는 의미이다. |
| chargingProfilePurpose | ChargingProfilePurposeType | 0..1 | Optional. 다른 기준을 충족하는 경우 지워질 충전 프로필의 목적을 지정한다. |
| stackLevel | integer | 0..1 | Optional. 다른 기준을 충족하는 경우 지워질 충전 프로필의 stackLevel을 지정한다. |

## 6.14. ClearChargingProfile.conf

이것은 ClearChargingProfile.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 ClearChargingProfile.conf PDU의 필드 정의를 포함한다. Clear Charging Profile도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | ClearChargingProfileStatus | 1..1 | Required. 충전기가 요청을 실행할 수 있었는지 여부를 나타낸다. |

## 6.15. DataTransfer.req

이것은 중앙 시스템이 충전기로 전송하거나 그 반대로 전송하는 DataTransfer.req PDU의 필드 정의를 포함한다. Data Transfer도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| vendorId | CiString255Type | 1..1 | Required. 벤더별 구현을 식별한다. |
| messageId | CiString50Type | 0..1 | Optional. 추가 식별 필드 |
| data | Text<br>Length undefined | 0..1 | Optional. 지정된 길이나 형식이 없는 데이터. |

## 6.16. DataTransfer.conf

이것은 DataTransfer.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 또는 그 반대로 전송하는 DataTransfer.conf PDU의 필드 정의를 포함한다. Data Transfer도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | DataTransferStatus | 1..1 | Required. 데이터 전송의 성공 또는 실패를 나타낸다. |
| data | Text<br>Length undefined | 0..1 | Optional. 요청에 대한 응답 데이터. |

## 6.17. DiagnosticsStatusNotification.req

이것은 충전기가 중앙 시스템으로 전송하는 DiagnosticsStatusNotification.req PDU의 필드 정의를 포함한다. Diagnostics Status Notification도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | DiagnosticsStatus | 1..1 | Required. 진단 업로드의 상태를 포함한다. |

## 6.18. DiagnosticsStatusNotification.conf

이것은 DiagnosticsStatusNotification.req PDU에 대한 응답으로 중앙 시스템이 충전기로 전송하는 DiagnosticsStatusNotification.conf PDU의 필드 정의를 포함한다. Diagnostics Status Notification도 참조하라.

필드가 정의되지 않음.

## 6.19. FirmwareStatusNotification.req

이것은 충전기가 중앙 시스템으로 전송하는 FirmwareStatusNotification.req PDU의 필드 정의를 포함한다. Firmware Status Notification도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | FirmwareStatus | 1..1 | Required. 펌웨어 설치의 진행 상태를 포함한다. |

## 6.20. FirmwareStatusNotification.conf

이것은 FirmwareStatusNotification.req PDU에 대한 응답으로 중앙 시스템이 충전기로 전송하는 FirmwareStatusNotification.conf PDU의 필드 정의를 포함한다. Firmware Status Notification도 참조하라.

필드가 정의되지 않음.

## 6.21. GetCompositeSchedule.req

이것은 중앙 시스템이 충전기로 전송하는 GetCompositeSchedule.req PDU의 필드 정의를 포함한다. Get Composite Schedule도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| connectorId | integer | 1..1 | Required. 스케줄이 요청되는 커넥터의 ID. ConnectorId=0이면 충전기가 그리드 연결에 대한 예상 소비량을 계산한다. |
| duration | integer | 1..1 | Required. 초 단위 시간. 요청된 스케줄의 길이 |
| chargingRateUnit | ChargingRateUnitType | 0..1 | Optional. 전력 또는 전류 프로필을 강제하는 데 사용할 수 있다. |

## 6.22. GetCompositeSchedule.conf

이것은 GetCompositeSchedule.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 GetCompositeSchedule.conf PDU의 필드 정의를 포함한다. Get Composite Schedule도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | GetCompositeScheduleStatus | 1..1 | Required. 요청의 상태. 충전기가 요청을 처리할 수 있었는지 여부를 나타낸다. |
| connectorId | integer | 0..1 | Optional. 이 알림에 포함된 충전 스케줄이 적용되는 커넥터. |
| scheduleStart | dateTime | 0..1 | Optional. 시간. 충전 프로필에 포함된 기간은 시간상 이 시점과 관련이 있다.<br>상태가 "Rejected"인 경우, 이 필드는 없을 수 있다. |
| chargingSchedule | ChargingSchedule | 0..1 | Optional. 계획된 복합 충전 스케줄, 시간 경과에 따른 에너지 소비. 항상 ScheduleStart와 관련이 있다.<br>상태가 "Rejected"인 경우, 이 필드는 없을 수 있다. |

## 6.23. GetConfiguration.req

이것은 중앙 시스템이 충전기로 전송하는 GetConfiguration.req PDU의 필드 정의를 포함한다. Get Configuration도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| key | CiString50Type | 0..* | Optional. 구성 값이 요청되는 키 목록. |

## 6.24. GetConfiguration.conf

이것은 GetConfiguration.req에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 GetConfiguration.conf PDU의 필드 정의를 포함한다. Get Configuration도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| configurationKey | KeyValue | 0..* | Optional. 요청되거나 알려진 키 목록 |
| unknownKey | CiString50Type | 0..* | Optional. 알 수 없는 요청된 키 |

## 6.25. GetDiagnostics.req

이것은 중앙 시스템이 충전기로 전송하는 GetDiagnostics.req PDU의 필드 정의를 포함한다. Get Diagnostics도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| location | anyURI | 1..1 | Required. 진단 파일을 업로드할 위치(디렉토리)를 포함한다. |
| retries | integer | 0..1 | Optional. 충전기가 진단을 업로드하기 전에 몇 번이나 시도해야 하는지 지정한다. 이 필드가 없으면 충전기가 재시도 횟수를 결정한다. |
| retryInterval | integer | 0..1 | Optional. 재시도를 시도할 수 있는 간격(초). 이 필드가 없으면 충전기가 시도 간 대기 시간을 결정한다. |
| startTime | dateTime | 0..1 | Optional. 진단에 포함할 가장 오래된 로깅 정보의 날짜 및 시간을 포함한다. |
| stopTime | dateTime | 0..1 | Optional. 진단에 포함할 최신 로깅 정보의 날짜 및 시간을 포함한다. |

## 6.26. GetDiagnostics.conf

이것은 GetDiagnostics.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 GetDiagnostics.conf PDU의 필드 정의를 포함한다. Get Diagnostics도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| fileName | CiString255Type | 0..1 | Optional. 진단 파일이 업로드될 파일명(경로 포함)을 지정한다. |

## 6.27. GetLocalListVersion.req

이것은 중앙 시스템이 충전기로 전송하는 GetLocalListVersion.req PDU의 필드 정의를 포함한다. Get Local List Version도 참조하라.

필드가 정의되지 않음.

## 6.28. GetLocalListVersion.conf

이것은 GetLocalListVersion.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 GetLocalListVersion.conf PDU의 필드 정의를 포함한다. Get Local List Version도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| listVersion | integer | 1..1 | Required. 충전기의 로컬 인증 목록의 현재 버전 번호를 포함한다. |

## 6.29. Heartbeat.req

이것은 충전기가 중앙 시스템으로 전송하는 Heartbeat.req PDU의 필드 정의를 포함한다. Heartbeat도 참조하라.

필드가 정의되지 않음.

## 6.30. Heartbeat.conf

이것은 Heartbeat.req PDU에 대한 응답으로 중앙 시스템이 충전기로 전송하는 Heartbeat.conf PDU의 필드 정의를 포함한다. Heartbeat도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| currentTime | dateTime | 1..1 | Required. 중앙 시스템의 현재 시간을 포함한다. |

## 6.31. MeterValues.req

이것은 충전기가 중앙 시스템으로 전송하는 MeterValues.req PDU의 필드 정의를 포함한다. Meter Values도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| connectorId | integer<br>connectorId >= 0 | 1..1 | Required. 충전기의 커넥터를 지정하는 번호(>0)를 포함한다. '0'(영)은 메인 전력계를 지정하는 데 사용된다. |
| transactionId | integer | 0..1 | Optional. 이러한 미터 샘플이 관련된 트랜잭션. |
| meterValue | MeterValue | 1..* | Required. 타임스탬프가 있는 샘플링된 미터 값. |

## 6.32. MeterValues.conf

이것은 MeterValues.req PDU에 대한 응답으로 중앙 시스템이 충전기로 전송하는 MeterValues.conf PDU의 필드 정의를 포함한다. Meter Values도 참조하라.

필드가 정의되지 않음.

## 6.33. RemoteStartTransaction.req

이것은 중앙 시스템이 충전기로 전송하는 RemoteStartTransaction.req PDU의 필드 정의를 포함한다. Remote Start Transaction도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| connectorId | integer | 0..1 | Optional. 트랜잭션을 시작할 커넥터의 번호. connectorId는 > 0이어야 한다(SHALL). |
| idTag | IdToken | 1..1 | Required. 충전기가 트랜잭션을 시작하는 데 사용해야 하는 식별자. |
| chargingProfile | ChargingProfile | 0..1 | Optional. 요청된 트랜잭션에 대해 충전기가 사용할 충전 프로필. ChargingProfilePurpose는 TxProfile로 설정되어야 한다(MUST). |

## 6.34. RemoteStartTransaction.conf

이것은 충전기가 중앙 시스템으로 전송하는 RemoteStartTransaction.conf PDU의 필드 정의를 포함한다. Remote Start Transaction도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | RemoteStartStopStatus | 1..1 | Required. 충전기가 트랜잭션 시작 요청을 수락하는지 여부를 나타내는 상태. |

## 6.35. RemoteStopTransaction.req

이것은 중앙 시스템이 충전기로 전송하는 RemoteStopTransaction.req PDU의 필드 정의를 포함한다. Remote Stop Transaction도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| transactionId | integer | 1..1 | Required. 충전기가 중지하도록 요청받은 트랜잭션의 식별자. |

## 6.36. RemoteStopTransaction.conf

이것은 충전기가 중앙 시스템으로 전송하는 RemoteStopTransaction.conf PDU의 필드 정의를 포함한다. Remote Stop Transaction도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | RemoteStartStopStatus | 1..1 | Required. 충전기가 트랜잭션 중지 요청을 수락하는지 여부를 나타내는 상태. |

## 6.37. ReserveNow.req

이것은 중앙 시스템이 충전기로 전송하는 ReserveNow.req PDU의 필드 정의를 포함한다. Reserve Now도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| connectorId | integer<br>connectorId >= 0 | 1..1 | Required. 예약할 커넥터의 id를 포함한다. 값이 0이면 예약이 특정 커넥터에 대한 것이 아님을 의미한다. |
| expiryDate | dateTime | 1..1 | Required. 예약이 종료되는 날짜 및 시간을 포함한다. |
| idTag | IdToken | 1..1 | Required. 충전기가 커넥터를 예약해야 하는 식별자. |
| parentIdTag | IdToken | 0..1 | Optional. parent idTag. |
| reservationId | integer | 1..1 | Required. 이 예약의 고유 id. |

## 6.38. ReserveNow.conf

이것은 ReserveNow.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 ReserveNow.conf PDU의 필드 정의를 포함한다. Reserve Now도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | ReservationStatus | 1..1 | Required. 예약의 성공 또는 실패를 나타낸다. |

## 6.39. Reset.req

이것은 중앙 시스템이 충전기로 전송하는 Reset.req PDU의 필드 정의를 포함한다. Reset도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| type | ResetType | 1..1 | Required. 충전기가 수행해야 하는 재설정 유형을 포함한다. |

## 6.40. Reset.conf

이것은 Reset.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 Reset.conf PDU의 필드 정의를 포함한다. Reset도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | ResetStatus | 1..1 | Required. 충전기가 재설정을 수행할 수 있는지 여부를 나타낸다. |

## 6.41. SendLocalList.req

이것은 중앙 시스템이 충전기로 전송하는 SendLocalList.req PDU의 필드 정의를 포함한다.

(비어있는) localAuthorizationList가 제공되지 않고 updateType이 Full인 경우, 모든 식별자가 목록에서 제거된다. (비어있는) localAuthorizationList 없이 Differential 업데이트를 요청하면 목록에 영향을 주지 않는다. localAuthorizationList의 모든 idTag는 고유해야 하며(MUST), 중복 값은 허용되지 않는다. Send Local List도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| listVersion | integer | 1..1 | Required. 전체 업데이트의 경우 전체 목록의 버전 번호이다. 차등 업데이트의 경우 업데이트가 적용된 후 목록의 버전 번호이다. |
| localAuthorizationList | AuthorizationData | 0..* | Optional. 전체 업데이트의 경우 새로운 로컬 인증 목록을 구성하는 값 목록을 포함한다. 차등 업데이트의 경우 충전기의 로컬 인증 목록에 적용할 변경 사항을 포함한다. AuthorizationData 요소의 최대 개수는 구성 키 SendLocalListMaxLength에서 사용 가능하다. |
| updateType | UpdateType | 1..1 | Required. 이 요청의 업데이트 유형(전체 또는 차등)을 포함한다. |

## 6.42. SendLocalList.conf

이것은 SendLocalList.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 SendLocalList.conf PDU의 필드 정의를 포함한다. Send Local List도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | UpdateStatus | 1..1 | Required. 충전기가 로컬 인증 목록의 업데이트를 성공적으로 수신하고 적용했는지 여부를 나타낸다. |

## 6.43. SetChargingProfile.req

이것은 중앙 시스템이 충전기로 전송하는 SetChargingProfile.req PDU의 필드 정의를 포함한다. Set Charging Profile도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| connectorId | integer | 1..1 | Required. 충전 프로필이 적용되는 커넥터. connectorId = 0이면 메시지에 충전기에 대한 전체 제한이 포함된다. |
| csChargingProfiles | ChargingProfile | 1..1 | Required. 충전기에 설정할 충전 프로필. |

## 6.44. SetChargingProfile.conf

이것은 SetChargingProfile.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 SetChargingProfile.conf PDU의 필드 정의를 포함한다. Set Charging Profile도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | ChargingProfileStatus | 1..1 | Required. 충전기가 메시지를 성공적으로 처리할 수 있었는지 여부를 반환한다. 이것은 스케줄이 문자 그대로 따라갈 것임을 보장하지 않는다. 충전기가 고려해야 할 다른 제약 조건이 있을 수 있다. |

## 6.45. StartTransaction.req

이것은 충전기가 중앙 시스템으로 전송하는 StartTransaction.req PDU의 필드 정의를 포함한다. Start Transaction도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| connectorId | integer<br>connectorId > 0 | 1..1 | Required. 사용되는 충전기의 커넥터를 식별한다. |
| idTag | IdToken | 1..1 | Required. 트랜잭션을 시작해야 하는 식별자를 포함한다. |
| meterStart | integer | 1..1 | Required. 트랜잭션 시작 시 커넥터의 미터 값(Wh)을 포함한다. |
| reservationId | integer | 0..1 | Optional. 이 트랜잭션의 결과로 종료되는 예약의 id를 포함한다. |
| timestamp | dateTime | 1..1 | Required. 트랜잭션이 시작되는 날짜 및 시간을 포함한다. |

## 6.46. StartTransaction.conf

이것은 StartTransaction.req PDU에 대한 응답으로 중앙 시스템이 충전기로 전송하는 StartTransaction.conf PDU의 필드 정의를 포함한다. Start Transaction도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| idTagInfo | IdTagInfo | 1..1 | Required. 인증 상태, 만료 및 parent id에 대한 정보를 포함한다. |
| transactionId | integer | 1..1 | Required. 중앙 시스템이 제공한 트랜잭션 id를 포함한다. |

## 6.47. StatusNotification.req

이것은 충전기가 중앙 시스템으로 전송하는 StatusNotification.req PDU의 필드 정의를 포함한다. Status Notification도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| connectorId | integer<br>connectorId >= 0 | 1..1 | Required. 상태가 보고되는 커넥터의 id. Id '0'(영)은 충전기 메인 컨트롤러에 대한 상태인 경우 사용된다. |
| errorCode | ChargePointErrorCode | 1..1 | Required. 충전기가 보고한 오류 코드를 포함한다. |
| info | CiString50Type | 0..1 | Optional. 오류와 관련된 추가 자유 형식 정보. |
| status | ChargePointStatus | 1..1 | Required. 충전기의 현재 상태를 포함한다. |
| timestamp | dateTime | 0..1 | Optional. 상태가 보고되는 시간. 없는 경우 메시지 수신 시간으로 간주된다. |
| vendorId | CiString255Type | 0..1 | Optional. 벤더별 구현을 식별한다. |
| vendorErrorCode | CiString50Type | 0..1 | Optional. 벤더별 오류 코드를 포함한다. |

## 6.48. StatusNotification.conf

이것은 StatusNotification.req PDU에 대한 응답으로 중앙 시스템이 충전기로 전송하는 StatusNotification.conf PDU의 필드 정의를 포함한다. Status Notification도 참조하라.

필드가 정의되지 않음.

## 6.49. StopTransaction.req

이것은 충전기가 중앙 시스템으로 전송하는 StopTransaction.req PDU의 필드 정의를 포함한다. Stop Transaction도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| idTag | IdToken | 0..1 | Optional. 충전 중지를 요청한 식별자를 포함한다. 충전기가 idTag 없이 충전을 종료할 수 있기 때문에 선택 사항이다(예: 재설정의 경우). 충전기는 알려진 경우 idTag를 전송해야 한다(SHALL). |
| meterStop | integer | 1..1 | Required. 트랜잭션 종료 시 커넥터의 미터 값(Wh)을 포함한다. |
| timestamp | dateTime | 1..1 | Required. 트랜잭션이 중지된 날짜 및 시간을 포함한다. |
| transactionId | integer | 1..1 | Required. StartTransaction.conf에서 받은 트랜잭션 id를 포함한다. |
| reason | Reason | 0..1 | Optional. 트랜잭션이 중지된 이유를 포함한다. Reason이 "Local"인 경우에만 생략될 수 있다(MAY). |
| transactionData | MeterValue | 0..* | Optional. 청구 목적과 관련된 트랜잭션 사용 세부정보를 포함한다. |

## 6.50. StopTransaction.conf

이것은 StopTransaction.req PDU에 대한 응답으로 중앙 시스템이 충전기로 전송하는 StopTransaction.conf PDU의 필드 정의를 포함한다. Stop Transaction도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| idTagInfo | IdTagInfo | 0..1 | Optional. 인증 상태, 만료 및 parent id에 대한 정보를 포함한다. 식별자 없이 트랜잭션이 중지되었을 수 있기 때문에 선택 사항이다. |

## 6.51. TriggerMessage.req

이것은 중앙 시스템이 충전기로 전송하는 TriggerMessage.req PDU의 필드 정의를 포함한다. Trigger Message도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| requestedMessage | MessageTrigger | 1..1 | Required. |
| connectorId | integer<br>connectorId > 0 | 0..1 | Optional. 요청이 특정 커넥터에 적용되는 경우에만 입력된다. |

## 6.52. TriggerMessage.conf

이것은 TriggerMessage.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 TriggerMessage.conf PDU의 필드 정의를 포함한다. Trigger Message도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | TriggerMessageStatus | 1..1 | Required. 충전기가 요청된 알림을 전송할지 여부를 나타낸다. |

## 6.53. UnlockConnector.req

이것은 중앙 시스템이 충전기로 전송하는 UnlockConnector.req PDU의 필드 정의를 포함한다. Unlock Connector도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| connectorId | integer<br>connectorId > 0 | 1..1 | Required. 잠금 해제할 커넥터의 식별자를 포함한다. |

## 6.54. UnlockConnector.conf

이것은 UnlockConnector.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 UnlockConnector.conf PDU의 필드 정의를 포함한다. Unlock Connector도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| status | UnlockStatus | 1..1 | Required. 충전기가 커넥터를 잠금 해제했는지 여부를 나타낸다. |

## 6.55. UpdateFirmware.req

이것은 중앙 시스템이 충전기로 전송하는 UpdateFirmware.req PDU의 필드 정의를 포함한다. Update Firmware도 참조하라.

| FIELD NAME | FIELD TYPE | CARD. | DESCRIPTION |
|------------|------------|-------|-------------|
| location | anyURI | 1..1 | Required. 펌웨어를 검색할 위치를 가리키는 URI를 포함하는 문자열을 포함한다. |
| retries | integer | 0..1 | Optional. 포기하기 전에 충전기가 펌웨어 다운로드를 시도해야 하는 횟수를 지정한다. 이 필드가 없으면 재시도 횟수는 충전기가 결정한다. |
| retrieveDate | dateTime | 1..1 | Required. 충전기가 (새) 펌웨어를 검색할 수 있는 날짜 및 시간을 포함한다. |
| retryInterval | integer | 0..1 | Optional. 재시도를 시도할 수 있는 간격(초). 이 필드가 없으면 시도 간 대기 시간은 충전기가 결정한다. |

## 6.56. UpdateFirmware.conf

이것은 UpdateFirmware.req PDU에 대한 응답으로 충전기가 중앙 시스템으로 전송하는 UpdateFirmware.conf PDU의 필드 정의를 포함한다. Update Firmware도 참조하라.

필드가 정의되지 않음.
