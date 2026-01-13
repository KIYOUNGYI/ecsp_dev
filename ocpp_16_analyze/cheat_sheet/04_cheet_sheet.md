# OCPP 1.6 - Chapter 4 Cheat Sheet
## Operations Initiated by Charge Point

> **목적**: 충전기에서 시작하는 주요 OCPP 메시지들의 핵심 동작 이해  
> **작성일**: 2026-01-13

---

## 📋 목차

1. [4.1 Authorize (인증)](#41-authorize-인증)
2. [4.2 Boot Notification (부팅 알림)](#42-boot-notification-부팅-알림)
3. [4.3 Data Transfer (데이터 전송)](#43-data-transfer-데이터-전송)
4. [4.4 Diagnostics Status Notification (진단 상태 알림)](#44-diagnostics-status-notification-진단-상태-알림)
5. [4.5 Firmware Status Notification (펌웨어 상태 알림)](#45-firmware-status-notification-펌웨어-상태-알림)
6. [4.6 Heartbeat (심박)](#46-heartbeat-심박)
7. [4.7 Meter Values (미터 값)](#47-meter-values-미터-값)
8. [4.8 Start Transaction (트랜잭션 시작)](#48-start-transaction-트랜잭션-시작)
9. [4.9 Status Notification (상태 알림)](#49-status-notification-상태-알림)
10. [4.10 Stop Transaction (트랜잭션 종료)](#410-stop-transaction-트랜잭션-종료)

---

## 🔑 4.1 Authorize (인증)

### 개요
사용자(RFID 카드 등)가 충전을 시작하기 전 권한 확인

### 메시지 흐름
```
고객 RFID 카드 태그
    ↓
CP → CS: Authorize.req { idTag: "RFID123" }
    ↓
CS → CP: Authorize.conf { 
  idTagInfo: {
    status: "Accepted",  // or "Blocked", "Expired", "Invalid"
    expiryDate: "2026-12-31T23:59:59Z"
  }
}
```

### 응답 Status
- **Accepted**: ✅ 충전 가능
- **Blocked**: 🚫 차단된 사용자
- **Expired**: ⏰ 만료된 카드
- **Invalid**: ❌ 유효하지 않은 카드
- **ConcurrentTx**: ⚠️ 동시 충전 중

### 실전 예시
```json
// Request
{
  "idTag": "USER_CARD_001"
}

// Response - 정상
{
  "idTagInfo": {
    "status": "Accepted",
    "expiryDate": "2027-01-13T00:00:00Z",
    "parentIdTag": "PARENT_001"
  }
}

// Response - 차단
{
  "idTagInfo": {
    "status": "Blocked"
  }
}
```

---

## 🔌 4.2 Boot Notification (부팅 알림)

### 개요
충전기가 켜지면 가장 먼저 보내는 "안녕하세요!" 메시지

### 어린이용 설명 🧒
```
🔌 충전기 켜짐
    ↓
📢 "안녕하세요!" (BootNotification.req)
    ↓
    ├─→ ✅ "환영해! 5분마다 연락해" (Accepted, interval: 300)
    │       ↓
    │   🎉 일 시작! (충전 가능)
    │   ⏰ 5분마다 Heartbeat 전송
    │
    ├─→ ⏸️ "잠깐만, 60초 뒤에 다시 말해" (Pending, interval: 60)
    │       ↓
    │   🤔 선생님 질문에 대답만 하기
    │   ⏰ 60초 뒤에 다시 "안녕하세요!" (BootNotification.req)
    │       ↓
    │   (나중에 Accepted 받으면 일 시작)
    │
    └─→ ❌ "안 돼, 10분 뒤에 다시 와" (Rejected, interval: 600)
            ↓
        😢 기다리기
        ⏰ 10분(600초) 뒤에 다시 "안녕하세요!" (BootNotification.req)
            ↓
        🔁 다시 시도
```

### 메시지 흐름
```
충전기 부팅
    ↓
CP → CS: BootNotification.req
    ↓
CS → CP: BootNotification.conf {
  status: "Accepted" | "Pending" | "Rejected",
  interval: 300,  // 초 단위
  currentTime: "2026-01-13T10:00:00Z"
}
```

### Status별 의미

#### ✅ Accepted (수락됨)
- **의미**: 충전기 등록 완료, 정상 작동 시작
- **interval**: Heartbeat 전송 주기 (초)
- **동작**: 
  - 충전 가능 상태
  - interval마다 Heartbeat 전송
  - 모든 OCPP 메시지 전송 가능

#### ⏸️ Pending (대기 중)
- **의미**: 등록 진행 중, 설정 확인 필요
- **interval**: BootNotification 재전송 주기 (초)
- **동작**:
  - ❌ 트랜잭션 불가 (Authorize, StartTransaction, StopTransaction)
  - ❌ 자발적 메시지 전송 불가
  - ✅ Central System의 요청에는 응답 가능 (GetConfiguration 등)
  - ⏰ interval 후 BootNotification 재전송

#### ❌ Rejected (거부됨)
- **의미**: 미등록 또는 차단된 충전기
- **interval**: BootNotification 재시도 주기 (초)
- **동작**:
  - ❌ 모든 업무 불가
  - ⏰ interval 후 BootNotification 재시도

### interval 값의 의미

| Status | interval의 의미 | 전송 메시지 | 예시 |
|--------|----------------|------------|------|
| **Accepted** | Heartbeat 주기 | `Heartbeat.req` | 300초(5분)마다 |
| **Pending** | BootNotification 재전송 주기 | `BootNotification.req` | 60초마다 |
| **Rejected** | BootNotification 재시도 주기 | `BootNotification.req` | 600초(10분)마다 |

### 실전 시나리오

#### 시나리오 1: 정상 온보딩 (Pending → Accepted)
```
10:00:00 → CP: BootNotification.req (첫 시도)
10:00:01 → CS: BootNotification.conf (status: Pending, interval: 30)
           
           // Pending 상태 동안
10:00:05 → CS: GetConfiguration.req ✅
10:00:06 → CP: GetConfiguration.conf ✅

10:00:10 → CS: ChangeConfiguration.req (설정 변경) ✅
10:00:11 → CP: ChangeConfiguration.conf ✅

           // 30초 경과
10:00:31 → CP: BootNotification.req (재전송)
10:00:32 → CS: BootNotification.conf (status: Accepted, interval: 300)

           // Accepted 상태
10:05:32 → CP: Heartbeat.req (5분마다)
10:10:32 → CP: Heartbeat.req
```

#### 시나리오 2: 미등록 충전기 (Rejected)
```
10:00:00 → CP: BootNotification.req
10:00:01 → CS: BootNotification.conf (status: Rejected, interval: 3600)

           // 1시간 대기
11:00:01 → CP: BootNotification.req (재시도)
11:00:02 → CS: BootNotification.conf (status: Rejected, interval: 3600)
           
           // 계속 거부...
```

#### 시나리오 3: Central System의 전략적 interval 설정
```typescript
// Central System의 전략
async function handleBootNotification(chargePointId, request) {
  const chargePoint = await db.getChargePoint(chargePointId);
  
  // 1. 미등록 충전기 → 긴 재시도 주기
  if (!chargePoint) {
    return {
      status: "Rejected",
      interval: 3600,  // 1시간 후 재시도
      currentTime: new Date().toISOString()
    };
  }
  
  // 2. 설정 변경 필요 → 짧은 재시도 주기
  if (chargePoint.needsConfiguration) {
    return {
      status: "Pending",
      interval: 60,  // 1분 후 재시도 (빨리 설정하기 위해)
      currentTime: new Date().toISOString()
    };
  }
  
  // 3. 서버 부하에 따른 Heartbeat 주기 조절
  const serverLoad = await getServerLoad();
  
  if (serverLoad > 80) {
    return {
      status: "Accepted",
      interval: 600,  // 10분마다 (부하 분산)
      currentTime: new Date().toISOString()
    };
  } else {
    return {
      status: "Accepted",
      interval: 300,  // 5분마다 (정상)
      currentTime: new Date().toISOString()
    };
  }
}
```

### Pending 상태의 핵심 규칙

#### ❌ 충전기가 할 수 없는 것
```javascript
// 트랜잭션 관련 - 모두 불가
cp.send(new AuthorizeRequest());        // ❌
cp.send(new StartTransactionRequest()); // ❌
cp.send(new StopTransactionRequest());  // ❌

// 자발적 메시지 - 모두 불가
cp.send(new MeterValuesRequest());      // ❌
cp.send(new StatusNotificationRequest()); // ❌ (자발적)
```

#### ✅ Central System이 할 수 있는 것
```javascript
// interval 값과 무관하게 언제든 가능!
cs.send(new GetConfigurationRequest());    // ✅
cs.send(new ChangeConfigurationRequest()); // ✅
cs.send(new GetDiagnosticsRequest());      // ✅
cs.send(new UpdateFirmwareRequest());      // ✅
cs.send(new TriggerMessageRequest());      // ✅
cs.send(new ResetRequest());               // ✅
```

#### ✅ 충전기가 할 수 있는 것
```javascript
// Central System의 요청에 응답만 가능
cp.on('GetConfiguration', (req) => {
  return new GetConfigurationResponse(); // ✅ 응답 OK
});

// interval 후 BootNotification 재전송
setTimeout(() => {
  cp.send(new BootNotificationRequest()); // ✅ 재전송 OK
}, interval * 1000);
```

### 실전 예시
```json
// Request
{
  "chargePointVendor": "VendorX",
  "chargePointModel": "Model-ABC",
  "chargePointSerialNumber": "SN-12345",
  "chargeBoxSerialNumber": "BOX-67890",
  "firmwareVersion": "1.0.5",
  "iccid": "89012345678901234567",
  "imsi": "123456789012345",
  "meterType": "EnergyMeter-v2",
  "meterSerialNumber": "METER-111"
}

// Response - Accepted (정상 등록)
{
  "status": "Accepted",
  "currentTime": "2026-01-13T10:00:00Z",
  "interval": 300  // 5분마다 Heartbeat
}

// Response - Pending (설정 확인 중)
{
  "status": "Pending",
  "currentTime": "2026-01-13T10:00:00Z",
  "interval": 60  // 1분 후 다시 BootNotification
}

// Response - Rejected (미등록)
{
  "status": "Rejected",
  "currentTime": "2026-01-13T10:00:00Z",
  "interval": 3600  // 1시간 후 재시도
}
```

### 4.2.1 Transactions before being accepted
**중요**: Pending/Rejected 상태에서는 트랜잭션 시작 불가!

```
Pending 상태에서 고객이 RFID 태그
    ↓
❌ Authorize.req 전송 불가
❌ StartTransaction.req 전송 불가
    ↓
고객에게 "충전기 준비 중" 메시지 표시
    ↓
Accepted 받은 후 정상 작동
```

---

## 📡 4.3 Data Transfer (데이터 전송)

### 개요
OCPP 표준 외 커스텀 데이터 교환용

### 메시지 흐름
```
CP → CS: DataTransfer.req {
  vendorId: "VendorX",
  messageId: "CustomMetric",
  data: "{ temperature: 45 }"
}
    ↓
CS → CP: DataTransfer.conf {
  status: "Accepted" | "Rejected" | "UnknownMessageId" | "UnknownVendorId"
}
```

### 사용 예시
```json
// 충전기 온도 전송
{
  "vendorId": "com.vendorx",
  "messageId": "TemperatureAlert",
  "data": "{\"temperature\": 65, \"threshold\": 60}"
}

// 커스텀 설정 조회
{
  "vendorId": "com.vendorx",
  "messageId": "GetCustomSettings",
  "data": ""
}
```

---

## 🔧 4.4 Diagnostics Status Notification (진단 상태 알림)

### 개요
진단 파일 업로드 상태 알림

### Status 값
- **Idle**: 대기 중
- **Uploading**: 업로드 중
- **Uploaded**: 업로드 완료
- **UploadFailed**: 업로드 실패

### 메시지 흐름
```
CS → CP: GetDiagnostics.req
    ↓
CP → CS: DiagnosticsStatusNotification.req { status: "Uploading" }
    ↓
    (파일 업로드 중)
    ↓
CP → CS: DiagnosticsStatusNotification.req { status: "Uploaded" }
```

---

## 📦 4.5 Firmware Status Notification (펌웨어 상태 알림)

### 개요
펌웨어 업데이트 진행 상태 알림

### Status 값
- **Downloaded**: 다운로드 완료
- **DownloadFailed**: 다운로드 실패
- **Downloading**: 다운로드 중
- **Idle**: 대기 중
- **InstallationFailed**: 설치 실패
- **Installing**: 설치 중
- **Installed**: 설치 완료

### 메시지 흐름
```
CS → CP: UpdateFirmware.req
    ↓
CP → CS: FirmwareStatusNotification.req { status: "Downloading" }
    ↓
CP → CS: FirmwareStatusNotification.req { status: "Downloaded" }
    ↓
CP → CS: FirmwareStatusNotification.req { status: "Installing" }
    ↓
CP → CS: FirmwareStatusNotification.req { status: "Installed" }
    ↓
    (재부팅)
    ↓
CP → CS: BootNotification.req (새 펌웨어 버전)
```

---

## 💓 4.6 Heartbeat (심박)

### 개요
충전기가 살아있음을 주기적으로 알림 ("저 살아있어요!")

### 메시지 흐름
```
CP → CS: Heartbeat.req {}
    ↓
CS → CP: Heartbeat.conf { 
  currentTime: "2026-01-13T10:05:00Z" 
}
```

### 주기 설정
- BootNotification.conf의 **interval** 값으로 결정 (Accepted 상태일 때)
- 또는 Configuration Key: `HeartbeatInterval` (초 단위)

### 실전 시나리오
```
10:00:00 → BootNotification.conf (status: Accepted, interval: 300)
10:05:00 → Heartbeat.req
10:10:00 → Heartbeat.req
10:15:00 → Heartbeat.req
...
```

### 모니터링 활용
```typescript
// Central System에서 충전기 연결 상태 모니터링
class ChargePointMonitor {
  async checkHeartbeat(chargePointId: string) {
    const lastHeartbeat = await db.getLastHeartbeat(chargePointId);
    const now = new Date();
    const diff = now.getTime() - lastHeartbeat.getTime();
    
    // 예상 interval의 2배 초과 시 오프라인 판단
    const expectedInterval = 300 * 1000; // 5분
    
    if (diff > expectedInterval * 2) {
      console.warn(`ChargePoint ${chargePointId} is offline!`);
      await this.sendAlert(chargePointId);
    }
  }
}
```

---

## ⚡ 4.7 Meter Values (미터 값)

### 개요
전력 사용량, 전압, 전류 등 측정값 전송

### 메시지 흐름
```
CP → CS: MeterValues.req {
  connectorId: 1,
  transactionId: 12345,
  meterValue: [{
    timestamp: "2026-01-13T10:05:00Z",
    sampledValue: [{
      value: "15.5",
      context: "Sample.Periodic",
      measurand: "Energy.Active.Import.Register",
      unit: "kWh"
    }]
  }]
}
```

### 주요 Measurand (측정 항목)
- **Energy.Active.Import.Register**: 누적 전력량 (kWh)
- **Power.Active.Import**: 현재 전력 (kW)
- **Current.Import**: 전류 (A)
- **Voltage**: 전압 (V)
- **SoC**: 배터리 충전율 (%)
- **Temperature**: 온도 (Celsius)

### Context
- **Sample.Periodic**: 주기적 샘플링
- **Sample.Clock**: 특정 시각 샘플링
- **Transaction.Begin**: 트랜잭션 시작
- **Transaction.End**: 트랜잭션 종료

### 실전 예시
```json
{
  "connectorId": 1,
  "transactionId": 789,
  "meterValue": [
    {
      "timestamp": "2026-01-13T10:05:00Z",
      "sampledValue": [
        {
          "value": "12.5",
          "context": "Sample.Periodic",
          "measurand": "Energy.Active.Import.Register",
          "unit": "kWh"
        },
        {
          "value": "7.2",
          "context": "Sample.Periodic",
          "measurand": "Power.Active.Import",
          "unit": "kW"
        },
        {
          "value": "32",
          "context": "Sample.Periodic",
          "measurand": "Current.Import",
          "unit": "A",
          "phase": "L1"
        },
        {
          "value": "230",
          "context": "Sample.Periodic",
          "measurand": "Voltage",
          "unit": "V",
          "phase": "L1"
        }
      ]
    }
  ]
}
```

---

## 🚀 4.8 Start Transaction (트랜잭션 시작)

### 개요
충전 세션 시작 (RFID 태그 후)

### 메시지 흐름
```
고객 RFID 태그
    ↓
CP → CS: Authorize.req { idTag: "USER001" }
    ↓
CS → CP: Authorize.conf { idTagInfo: { status: "Accepted" } }
    ↓
커넥터에 차량 연결
    ↓
CP → CS: StartTransaction.req {
  connectorId: 1,
  idTag: "USER001",
  meterStart: 1000,
  timestamp: "2026-01-13T10:00:00Z"
}
    ↓
CS → CP: StartTransaction.conf {
  transactionId: 12345,
  idTagInfo: { status: "Accepted" }
}
```

### 실전 시나리오
```
10:00:00 → 고객 RFID 태그
10:00:01 → Authorize.req (idTag: "CARD_123")
10:00:02 → Authorize.conf (status: Accepted)
10:00:05 → 차량 연결 감지
10:00:06 → StartTransaction.req (connectorId: 1, meterStart: 1000)
10:00:07 → StartTransaction.conf (transactionId: 789)
10:00:07 → 충전 시작! ⚡
```

### 중요 필드
- **connectorId**: 커넥터 번호 (1, 2, ...)
- **idTag**: 사용자 식별자 (RFID)
- **meterStart**: 시작 시 미터 값 (Wh)
- **timestamp**: 시작 시각
- **reservationId**: 예약 ID (옵션)

---

## 📢 4.9 Status Notification (상태 알림)

### 개요
커넥터 상태 변경 알림

### Status 값
- **Available**: 사용 가능
- **Preparing**: 준비 중
- **Charging**: 충전 중
- **SuspendedEVSE**: 충전기 측 일시정지
- **SuspendedEV**: 차량 측 일시정지
- **Finishing**: 종료 중
- **Reserved**: 예약됨
- **Unavailable**: 사용 불가
- **Faulted**: 고장

### ErrorCode 값
- **NoError**: 정상
- **ConnectorLockFailure**: 커넥터 잠금 실패
- **EVCommunicationError**: 차량 통신 오류
- **GroundFailure**: 접지 실패
- **HighTemperature**: 고온
- **OverCurrentFailure**: 과전류
- **PowerMeterFailure**: 전력계 고장
- **UnderVoltage**: 저전압
- **OverVoltage**: 과전압

### 메시지 흐름
```
커넥터 상태 변경
    ↓
CP → CS: StatusNotification.req {
  connectorId: 1,
  errorCode: "NoError",
  status: "Charging",
  timestamp: "2026-01-13T10:05:00Z"
}
```

### 충전 세션 상태 흐름
```
Available (사용 가능)
    ↓ (RFID 태그)
Preparing (준비 중)
    ↓ (차량 연결)
Charging (충전 중)
    ↓ (충전 완료)
Finishing (종료 중)
    ↓ (커넥터 분리)
Available (사용 가능)
```

### 실전 예시
```json
// 충전 시작
{
  "connectorId": 1,
  "errorCode": "NoError",
  "status": "Charging",
  "timestamp": "2026-01-13T10:00:00Z",
  "info": "Started charging",
  "vendorId": "VendorX",
  "vendorErrorCode": ""
}

// 고장 발생
{
  "connectorId": 1,
  "errorCode": "HighTemperature",
  "status": "Faulted",
  "timestamp": "2026-01-13T12:30:00Z",
  "info": "Temperature: 75°C",
  "vendorId": "VendorX",
  "vendorErrorCode": "ERR_TEMP_001"
}
```

---

## 🛑 4.10 Stop Transaction (트랜잭션 종료)

### 개요
충전 세션 종료

### 메시지 흐름
```
충전 종료 (차량 만충 or 사용자 중지)
    ↓
CP → CS: StopTransaction.req {
  transactionId: 12345,
  idTag: "USER001",
  meterStop: 1250,
  timestamp: "2026-01-13T11:00:00Z",
  reason: "EVDisconnected"
}
    ↓
CS → CP: StopTransaction.conf {
  idTagInfo: { status: "Accepted" }
}
```

### Reason 값
- **EmergencyStop**: 비상 정지
- **EVDisconnected**: 차량 연결 해제
- **HardReset**: 하드 리셋
- **Local**: 로컬 중지
- **Other**: 기타
- **PowerLoss**: 전력 손실
- **Reboot**: 재부팅
- **Remote**: 원격 중지
- **SoftReset**: 소프트 리셋
- **UnlockCommand**: 잠금 해제 명령
- **DeAuthorized**: 인증 해제

### 실전 시나리오
```
10:00:07 → StartTransaction (transactionId: 789, meterStart: 1000)
10:05:00 → MeterValues (currentMeter: 1050)
10:10:00 → MeterValues (currentMeter: 1100)
10:15:00 → MeterValues (currentMeter: 1150)
10:20:00 → 차량 만충, 연결 해제
10:20:01 → StopTransaction (transactionId: 789, meterStop: 1200, reason: EVDisconnected)
10:20:02 → 충전 종료, 사용량: 200Wh (1200 - 1000)
```

### transactionData 포함 예시
```json
{
  "transactionId": 789,
  "idTag": "USER001",
  "meterStop": 1250,
  "timestamp": "2026-01-13T11:00:00Z",
  "reason": "EVDisconnected",
  "transactionData": [
    {
      "timestamp": "2026-01-13T10:30:00Z",
      "sampledValue": [
        {
          "value": "1100",
          "context": "Sample.Periodic",
          "measurand": "Energy.Active.Import.Register",
          "unit": "Wh"
        }
      ]
    },
    {
      "timestamp": "2026-01-13T10:45:00Z",
      "sampledValue": [
        {
          "value": "1200",
          "context": "Sample.Periodic",
          "measurand": "Energy.Active.Import.Register",
          "unit": "Wh"
        }
      ]
    }
  ]
}
```

---

## 🎯 전체 충전 세션 흐름 예시

### 정상 충전 시나리오 (처음부터 끝까지)
```
=== 1. 충전기 부팅 ===
09:00:00 → CP: BootNotification.req
09:00:01 → CS: BootNotification.conf (status: Accepted, interval: 300)
09:00:01 → 충전기 준비 완료

=== 2. 주기적 Heartbeat ===
09:05:01 → CP: Heartbeat.req
09:10:01 → CP: Heartbeat.req
...

=== 3. 고객 도착 및 인증 ===
10:00:00 → 고객 RFID 태그 "USER_CARD_001"
10:00:01 → CP: Authorize.req (idTag: "USER_CARD_001")
10:00:02 → CS: Authorize.conf (status: Accepted)
10:00:02 → 충전기 LED 녹색 점등 ✅

=== 4. 차량 연결 ===
10:00:05 → 차량 커넥터 연결 감지
10:00:05 → CP: StatusNotification.req (status: Preparing)

=== 5. 트랜잭션 시작 ===
10:00:06 → CP: StartTransaction.req (
              connectorId: 1,
              idTag: "USER_CARD_001",
              meterStart: 5000,  // 5kWh
              timestamp: "2026-01-13T10:00:06Z"
            )
10:00:07 → CS: StartTransaction.conf (transactionId: 12345)
10:00:07 → 충전 시작! ⚡
10:00:07 → CP: StatusNotification.req (status: Charging)

=== 6. 주기적 미터 값 전송 (5분마다) ===
10:05:00 → CP: MeterValues.req (
              transactionId: 12345,
              currentMeter: 5600,  // 5.6kWh
              power: 7.2kW
            )
10:10:00 → CP: MeterValues.req (currentMeter: 6200)  // 6.2kWh
10:15:00 → CP: MeterValues.req (currentMeter: 6800)  // 6.8kWh
10:20:00 → CP: MeterValues.req (currentMeter: 7400)  // 7.4kWh

=== 7. 충전 완료 ===
10:23:00 → 차량 배터리 만충 (SoC 100%)
10:23:01 → CP: StatusNotification.req (status: SuspendedEV)
10:23:30 → 고객이 커넥터 분리
10:23:31 → CP: StatusNotification.req (status: Finishing)

=== 8. 트랜잭션 종료 ===
10:23:32 → CP: StopTransaction.req (
              transactionId: 12345,
              idTag: "USER_CARD_001",
              meterStop: 7500,  // 7.5kWh
              timestamp: "2026-01-13T10:23:32Z",
              reason: "EVDisconnected"
            )
10:23:33 → CS: StopTransaction.conf (status: Accepted)
10:23:33 → 충전 완료! 사용량: 2.5kWh (7.5 - 5.0)
10:23:33 → CP: StatusNotification.req (status: Available)

=== 9. 다시 대기 상태 ===
10:23:34 → 다음 고객 대기 중...
```

### 비정상 시나리오: 충전 중 고장 발생
```
10:00:00 → 충전 시작 (StartTransaction)
10:15:00 → MeterValues 전송 중...
10:15:30 → 과전류 감지!
10:15:31 → CP: StatusNotification.req (
              status: Faulted,
              errorCode: "OverCurrentFailure"
            )
10:15:32 → 충전 즉시 중단
10:15:33 → CP: StopTransaction.req (
              transactionId: 12345,
              reason: "EmergencyStop",
              meterStop: 6500
            )
10:15:34 → 관리자에게 알림 발송 📧
```

---

## 💡 핵심 포인트 정리

### 1. BootNotification의 interval 이해
- **Accepted**: Heartbeat 주기
- **Pending**: BootNotification 재전송 주기
- **Rejected**: BootNotification 재시도 주기

### 2. Pending 상태의 제약
- ❌ 트랜잭션 불가 (Authorize, Start/Stop)
- ❌ 자발적 메시지 전송 불가
- ✅ Central System 요청에는 응답 가능
- ✅ interval 후 BootNotification 재전송

### 3. 메시지 전송 순서 (충전 세션)
```
Authorize → StartTransaction → MeterValues (주기적) → StopTransaction
```

### 4. 상태 알림 순서 (StatusNotification)
```
Available → Preparing → Charging → Finishing → Available
```

### 5. Heartbeat 활용
- 충전기 온라인 상태 모니터링
- 시간 동기화 (currentTime)
- 예상 interval의 2배 초과 시 오프라인 판단

---

## 🔍 디버깅 Tips

### 충전기가 Pending에서 벗어나지 못할 때
```bash
# 1. GetConfiguration으로 설정 확인
CS → CP: GetConfiguration.req { key: ["AuthorizationKey"] }

# 2. 필요한 설정 변경
CS → CP: ChangeConfiguration.req { key: "AuthorizationKey", value: "..." }

# 3. TriggerMessage로 즉시 BootNotification 재전송 유도
CS → CP: TriggerMessage.req { requestedMessage: "BootNotification" }
```

### 트랜잭션이 시작되지 않을 때
```bash
# 1. 충전기 상태 확인
CP → CS: StatusNotification.req
# Status가 Available인지 확인

# 2. Authorize 성공 확인
CP → CS: Authorize.req
# idTagInfo.status가 Accepted인지 확인

# 3. 충전기 재부팅
CS → CP: Reset.req { type: "Soft" }
```

### Heartbeat가 오지 않을 때
```bash
# 1. 마지막 Heartbeat 시간 확인
SELECT lastHeartbeat FROM charge_points WHERE id = 'CP001';

# 2. WebSocket 연결 상태 확인
# 연결 끊김 → Reconnect 로직 확인

# 3. interval 설정 확인
# BootNotification.conf에서 설정한 interval 값 검증
```

---

## 📚 참고 문서
- OCPP 1.6 Edition 2 Specification
- `/ocpp_16_analyze/04_Operations_Initiated_by_Charge_Point.md`
- `/ocpp-j16_analyze/04_RPC_Framework.md`

---

**작성일**: 2026-01-13  
**버전**: 1.0  
**상태**: ✅ Complete
