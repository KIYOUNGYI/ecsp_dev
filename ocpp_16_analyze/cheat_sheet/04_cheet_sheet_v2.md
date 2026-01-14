# OCPP 1.6 - Chapter 4 Cheat Sheet v2

> **목적**: 번역 검수하면서 프로토콜 이해를 돕는 예시 작성  
> **작성일**: 2026-01-13  
> **업데이트**: 2026-01-14 (4.9 Status Notification 추가)  
> **방식**: 단계별로 검수하면서 채워나가기

---

## 📋 목차
- [4.2 Boot Notification](#-42-boot-notification-부팅-알림)
- [4.7 Meter Values](#-47-meter-values-미터-값)
- [4.9 Status Notification](#-49-status-notification-상태-알림) ⭐ NEW

---

## 🔌 4.2 Boot Notification (부팅 알림)

### 📌 핵심 개념

**Boot Notification**은 충전기가 켜지면 가장 먼저 보내는 "안녕하세요!" 메시지입니다.

### 🧒 어린이용 설명

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

---

### 📊 Status별 의미

#### ✅ Accepted (수락됨)

**의미**: 충전기 등록 완료, 정상 작동 시작

**interval**: Heartbeat 전송 주기 (초)

**동작**:
- ✅ 충전 가능 상태
- ✅ interval마다 Heartbeat 전송
- ✅ 모든 OCPP 메시지 전송 가능

**예시**:
```json
{
  "status": "Accepted",
  "currentTime": "2026-01-13T10:00:00Z",
  "interval": 300  // 5분마다 Heartbeat
}
```

#### ⏸️ Pending (대기 중)

**의미**: 등록 진행 중, 설정 확인 필요

**interval**: BootNotification 재전송 주기 (초)

**동작**:
- ❌ 트랜잭션 불가 (Authorize, StartTransaction, StopTransaction)
- ❌ 자발적 메시지 전송 불가
- ✅ Central System의 요청에는 응답 가능 (GetConfiguration 등)
- ⏰ interval 후 BootNotification 재전송

**예시**:
```json
{
  "status": "Pending",
  "currentTime": "2026-01-13T10:00:00Z",
  "interval": 60  // 1분 후 다시 BootNotification
}
```

#### ❌ Rejected (거부됨)

**의미**: 미등록 또는 차단된 충전기

**interval**: BootNotification 재시도 주기 (초)

**동작**:
- ❌ 모든 업무 불가
- ⏰ interval 후 BootNotification 재시도

**예시**:
```json
{
  "status": "Rejected",
  "currentTime": "2026-01-13T10:00:00Z",
  "interval": 3600  // 1시간 후 재시도
}
```

---

### 🔑 interval 값의 의미

| Status | interval의 의미 | 전송 메시지 | 예시 |
|--------|----------------|------------|------|
| **Accepted** | Heartbeat 주기 | `Heartbeat.req` | 300초(5분)마다 |
| **Pending** | BootNotification 재전송 주기 | `BootNotification.req` | 60초마다 |
| **Rejected** | BootNotification 재시도 주기 | `BootNotification.req` | 600초(10분)마다 |

**핵심 포인트**:
- interval 값은 **초(seconds) 단위**입니다
- Central System이 충전기에게 **"언제 다시 연락할지"** 알려주는 값입니다
- **Status에 따라 의미가 다릅니다**!

---

### 💡 Pending 상태의 핵심 규칙

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

**중요**: Pending 상태에서도 Central System은 **interval과 무관하게** 언제든 충전기에게 요청할 수 있습니다!

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

---

### 🎬 실전 시나리오

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

**핵심**: Central System은 **상황에 따라 다른 interval 값**을 줄 수 있습니다!

---

### 📝 4.2.1 Transactions before being accepted

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

## 📚 참고

- 원문: `/ocpp_16_analyze/04_Operations_Initiated_by_Charge_Point.md` - 4.2 Boot Notification
- 검수 완료: ✅ 2026-01-13

---

## 📊 4.7 Meter Values (미터 값)

### 📌 핵심 개념

**MeterValues**는 충전기가 Central System에게 "지금 이만큼 충전되고 있어요!"라고 보고하는 메시지입니다.

### 🧒 어린이용 설명

```
⚡ 충전 중...
    ↓
📊 60초마다 현재 상태 보고
    ├─ 💡 "25.5kWh 충전했어요!" (Energy)
    ├─ ⚡ "지금 7.2kW로 충전 중이에요!" (Power)
    ├─ 🔌 "32A 전류가 흐르고 있어요!" (Current)
    ├─ 📈 "배터리 45% 찼어요!" (SoC)
    └─ 🌡️ "온도는 35도예요!" (Temperature)
    ↓
💰 실시간 요금: ₩7,650
```

**비유**: 
- 스마트워치처럼 매 순간 건강 상태를 체크하는 것
- 자동차 계기판처럼 속도, 연료, 온도를 보여주는 것

---

### 📅 언제 보내나?

| 타이밍 | Context | 설명 |
|--------|---------|------|
| **주기적** | `Sample.Periodic` | 60초마다 자동 전송 (설정 가능) |
| **충전 시작** | `Transaction.Begin` | StartTransaction과 함께 전송 |
| **충전 종료** | `Transaction.End` | StopTransaction과 함께 전송 |

**설정 키**:
```
MeterValueSampleInterval = 60  (60초마다 전송)
```

---

### 🏗️ 구조

```
MeterValues
├── connectorId        (어떤 커넥터?)
├── transactionId      (어떤 충전 세션?)
└── meterValue[]       (측정값 리스트)
    ├── timestamp      (측정 시각)
    └── sampledValue[] (실제 측정 데이터들)
        ├── value      (측정값)
        ├── measurand  (무엇을 측정?)
        ├── phase      (어느 선로?)
        ├── unit       (단위)
        ├── context    (왜 측정?)
        ├── location   (어디서 측정?)
        └── format     (형식)
```

---

### 🔍 핵심 필드 설명

#### 1️⃣ **measurand** (무엇을 측정했나?)

| Measurand | 의미 | 단위 | 비유 | 예시 |
|-----------|------|------|------|------|
| `Energy.Active.Import.Register` | **누적 전력량** | kWh | 🚗 **주행거리계 (ODO)** - 출발지부터 총 누적 거리 | `25.5` |
| `Power.Active.Import` | **현재 전력** | kW | 🚗 **속도계 (SPEED)** - 지금 이 순간의 속도 | `7.2` |
| `Current.Import` | **전류** | A | 💧 **수도관의 유속** - 물이 흐르는 속도 | `32.5` |
| `Voltage` | **전압** | V | 💧 **수압** - 물이 밀리는 압력 | `220.5` |
| `SoC` | **배터리 충전률** | % | 📱 **스마트폰 배터리 %** - 얼마나 찼는지 | `45` |
| `Temperature` | **온도** | Celsius | 🌡️ **체온계** - 발열 체크 | `35` |

**핵심 구분**:

```
전력량 (Energy) vs 전력 (Power)
━━━━━━━━━━━━━━━━━━━━━━━━━━
Energy.Active.Import.Register = 주행거리 (총 누적)
  - "얼마나 많이" 사용했나
  - 충전 시작부터 현재까지 누적값
  - 과금 기준! 💰
  - 예: 25.5 kWh

Power.Active.Import = 속도 (현재 속도)
  - "얼마나 빠르게" 충전하나
  - 현재 순간의 충전 속도
  - 실시간 표시용
  - 예: 7.2 kW

전력량 = 전력 × 시간
25.5 kWh = 7.2 kW × 3.54 시간
```

```
전류 (Current) vs 전력 (Power)
━━━━━━━━━━━━━━━━━━━━━━━━━━
Current.Import = 물의 유속
  - 전기가 흐르는 양
  - 안전 관리용 (과전류 방지)
  - 예: 32 A

Power.Active.Import = 수차의 회전력
  - 실제 일하는 능력
  - 충전 속도
  - 예: 7.2 kW

관계식:
전력(P) = 전압(V) × 전류(I)
7.2 kW = 220V × 32A
```

#### 2️⃣ **context** (왜 측정했나?)

| Context | 의미 | 설명 |
|---------|------|------|
| `Sample.Periodic` | 주기적 측정 | 60초마다 자동 전송 |
| `Transaction.Begin` | 충전 시작 시점 | 초기값 기록 |
| `Transaction.End` | 충전 종료 시점 | 최종값 기록 (과금 계산) |

#### 3️⃣ **phase** (어느 선로?)

| Phase | 의미 | 설명 |
|-------|------|------|
| `L1`, `L2`, `L3` | 3상 전기의 각 선로 | Line 1, 2, 3 |
| `L1-N`, `L2-N`, `L3-N` | 각 선로와 중성선 사이 전압 | Phase Voltage (230V) |
| `L1-L2`, `L2-L3`, `L3-L1` | 선로 간 전압 | Line Voltage (400V) |
| `N` | 중성선 | Neutral |

**3상 vs 단상**:

```
단상 (Single-phase) - 완속 충전 (7kW)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
L1 ──────────┐
             │  220V
N ───────────┘

전력 = 220V × 32A = 7kW
용도: 가정용, 완속 충전


3상 (Three-phase) - 급속 충전 (50kW)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
L1 ──────────┐
             │
L2 ──────────┼  400V (선간)
             │  230V (상전압, 각 L-N)
L3 ──────────┤
             │
N ───────────┘

전력 = √3 × 400V × 80A ≈ 55kW
용도: 산업용, 급속 충전

왜 3개로 나눌까?
→ 부하 분산! 각 선로의 균형을 체크하기 위해
```

#### 4️⃣ **location** (어디서 측정?)

| Location | 의미 | 설명 |
|----------|------|------|
| `Inlet` | 충전기 입력단 | 전기가 들어오는 곳 |
| `Outlet` | 충전기 출력단 | 케이블 끝 (주로 사용) |
| `Body` | 충전기 본체 | 온도 측정 등 |
| `Cable` | 케이블 | 케이블 온도 등 |
| `EV` | 전기차 내부 | SoC 등 |

---

### 🎯 중요 규칙

#### ✅ **같은 measurand 중복 가능한 경우**

```
phase가 다를 때:
  - Current.Import + L1
  - Current.Import + L2
  - Current.Import + L3

location이 다를 때:
  - Current.Import + Inlet
  - Current.Import + Outlet

조합이 다를 때:
  - Current.Import + L1 + Inlet
  - Current.Import + L1 + Outlet
  - Current.Import + L2 + Inlet
  - Current.Import + L2 + Outlet
```

#### ❌ **중복 불가**

```json
// ❌ 똑같은 조합은 안 됨!
{
  "sampledValue": [
    {
      "measurand": "Current.Import",
      "phase": "L1",
      "location": "Outlet",
      "value": "80.5"
    },
    {
      "measurand": "Current.Import",
      "phase": "L1",
      "location": "Outlet",
      "value": "85.0"  // ❌ 중복!
    }
  ]
}
```

---

### 📊 실제 예시

#### 시나리오 1: 단상 완속 충전기 (7kW)

```json
{
  "connectorId": 1,
  "transactionId": 12345,
  "meterValue": [
    {
      "timestamp": "2026-01-13T10:05:00Z",
      "sampledValue": [
        {
          "value": "2.5",
          "measurand": "Energy.Active.Import.Register",
          "unit": "kWh",
          "context": "Sample.Periodic",
          "location": "Outlet"
        },
        {
          "value": "7.2",
          "measurand": "Power.Active.Import",
          "unit": "kW",
          "context": "Sample.Periodic",
          "location": "Outlet"
        },
        {
          "value": "32.0",
          "measurand": "Current.Import",
          "phase": "L1",
          "unit": "A",
          "location": "Outlet"
        },
        {
          "value": "220.5",
          "measurand": "Voltage",
          "phase": "L1-N",
          "unit": "V",
          "location": "Outlet"
        },
        {
          "value": "45",
          "measurand": "SoC",
          "unit": "Percent",
          "context": "Sample.Periodic",
          "location": "EV"
        }
      ]
    }
  ]
}
```

#### 시나리오 2: 3상 급속 충전기 (50kW)

```json
{
  "connectorId": 1,
  "transactionId": 12345,
  "meterValue": [
    {
      "timestamp": "2026-01-13T10:05:00Z",
      "sampledValue": [
        // 1. 누적 전력량 (과금 기준!)
        {
          "value": "25.5",
          "measurand": "Energy.Active.Import.Register",
          "unit": "kWh",
          "context": "Sample.Periodic",
          "location": "Outlet"
        },
        
        // 2. 현재 전력 (충전 속도)
        {
          "value": "55.2",
          "measurand": "Power.Active.Import",
          "unit": "kW",
          "context": "Sample.Periodic",
          "location": "Outlet"
        },
        
        // 3. 전류 - L1, L2, L3 각각!
        {
          "value": "80.5",
          "measurand": "Current.Import",
          "phase": "L1",
          "unit": "A",
          "location": "Outlet"
        },
        {
          "value": "79.8",
          "measurand": "Current.Import",
          "phase": "L2",
          "unit": "A",
          "location": "Outlet"
        },
        {
          "value": "80.2",
          "measurand": "Current.Import",
          "phase": "L3",
          "unit": "A",
          "location": "Outlet"
        },
        
        // 4. 전압 - L1-N, L2-N, L3-N 각각!
        {
          "value": "230.5",
          "measurand": "Voltage",
          "phase": "L1-N",
          "unit": "V",
          "location": "Outlet"
        },
        {
          "value": "229.8",
          "measurand": "Voltage",
          "phase": "L2-N",
          "unit": "V",
          "location": "Outlet"
        },
        {
          "value": "230.2",
          "measurand": "Voltage",
          "phase": "L3-N",
          "unit": "V",
          "location": "Outlet"
        },
        
        // 5. 배터리 충전률
        {
          "value": "45",
          "measurand": "SoC",
          "unit": "Percent",
          "context": "Sample.Periodic",
          "location": "EV"
        },
        
        // 6. 온도
        {
          "value": "35",
          "measurand": "Temperature",
          "unit": "Celsius",
          "context": "Sample.Periodic",
          "location": "Body"
        }
      ]
    }
  ]
}
```

---

### 💰 실시간 요금 계산

#### 방법

```typescript
// 실시간 요금 계산
실시간 요금 = (현재 Energy.Active.Import.Register - meterStart) 
            × 요금 (원/kWh)

// 예시
현재 Energy = 25.5 kWh  (MeterValues에서 수신)
시작 Energy = 0 kWh     (StartTransaction의 meterStart)
요금 = 300원/kWh

실시간 요금 = (25.5 - 0) × 300 = 7,650원
```

#### 타임라인 (60초 간격 예시)

```
시간   | Energy | 누적 요금
━━━━━━━━━━━━━━━━━━━━━━━━
10:00  | 0 kWh    | ₩0
10:01  | 0.12 kWh | ₩36
10:02  | 0.24 kWh | ₩72
10:03  | 0.36 kWh | ₩108
10:04  | 0.48 kWh | ₩144
10:05  | 0.60 kWh | ₩180
...
10:30  | 3.6 kWh  | ₩1,080
...
11:00  | 7.2 kWh  | ₩2,160 (최종)
```

#### 모바일 앱 UI 예시

```
┌─────────────────────────────┐
│   ⚡ 충전 중...             │
├─────────────────────────────┤
│                             │
│   사용량: 25.5 kWh          │
│   현재 요금: ₩7,650         │
│                             │
│   충전 시간: 5분            │
│   현재 전력: 55.2 kW        │
│                             │
│   ─────────────────────     │
│   │████████░░░░░░░░│ 45%   │
│   ─────────────────────     │
│                             │
│   예상 최종 요금: ₩15,000   │
│                             │
│   [ 충전 중지 ]             │
└─────────────────────────────┘
```

---

### 📐 Phase별 측정이 필요한 measurand

| Measurand | Phase 구분 필요? | 이유 |
|-----------|-----------------|------|
| `Current.Import` | ✅ **필요** | 3상 부하 균형 확인 |
| `Current.Export` | ✅ **필요** | 3상 각각 측정 |
| `Voltage` | ✅ **필요** | 각 상의 전압 모니터링 |
| `Power.Active.Import` | △ **선택** | 총합 또는 상별 |
| `Energy.Active.Import.Register` | ❌ **불필요** | 전체 누적값만 |
| `Temperature` | ❌ **불필요** | Phase 개념 없음 |
| `SoC` | ❌ **불필요** | 배터리 전체 상태 |

---

### 🎯 사용 목적별 정리

| 목적 | 사용 Measurand | 주기 |
|------|---------------|------|
| **과금 계산** | `Energy.Active.Import.Register` | Transaction.End |
| **실시간 요금 표시** | `Energy.Active.Import.Register` | Sample.Periodic (60초) |
| **충전 속도 표시** | `Power.Active.Import` | Sample.Periodic |
| **안전 모니터링** | `Current.Import`, `Voltage`, `Temperature` | Sample.Periodic |
| **부하 균형 체크** | `Current.Import` (L1, L2, L3) | Sample.Periodic |
| **배터리 상태 표시** | `SoC` | Sample.Periodic |

---

### 💡 핵심 정리

```
MeterValues = 충전 상태 종합 보고서

📍 언제: 60초마다 or 충전 시작/종료
📍 무엇: 전력량, 전력, 전류, 전압, 배터리, 온도 등
📍 왜: 과금, 모니터링, 안전 관리

핵심 공식:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
전력량(kWh) = 전력(kW) × 시간(h)
전력(kW) = 전압(V) × 전류(A) ÷ 1000
과금 = Energy.Active.Import.Register × 요금

핵심 개념:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ Energy.Active.Import.Register 
   → 누적 사용량 (과금 기준) 💰
   → 충전 시작부터 현재까지 총합
   → 주행거리계처럼 계속 증가

✅ Power.Active.Import 
   → 현재 충전 속도
   → 속도계처럼 순간값

✅ phase별 측정 가능 (L1, L2, L3)
   → 3상 충전기는 각 선로별로 측정
   → 부하 균형 확인용

✅ 같은 measurand도 phase/location 다르면 여러 개 OK
   → Current.Import + L1
   → Current.Import + L2
   → Current.Import + L3
```

---

## 📚 참고

- 원문: `/ocpp_16_analyze/04_Operations_Initiated_by_Charge_Point.md` - 4.7 Meter Values
- 검수 완료: ✅ 2026-01-13

---

**다음 작성 예정**: 4.1 Authorize, 4.3 Data Transfer, 4.8 Start Transaction, ...  
*(번역 검수하면서 하나씩 추가)*

---

## 🚦 4.9 Status Notification (상태 알림)

### 📌 핵심 개념

**Status Notification**은 충전기의 **상태 변화**를 중앙 시스템에 실시간으로 알리는 메시지입니다.

> **OCPP 1.6에서 가장 복잡한 메시지!** 9개 상태 × 9개 상태 = 81가지 전환 시나리오

### 🧒 어린이용 설명

```
🚦 신호등처럼 충전기 상태를 색깔로 표시해요

🟢 Available      → "비어 있어요, 충전 가능!"
🟡 Preparing      → "준비 중이에요"
🔵 Charging       → "충전하고 있어요"
🟠 SuspendedEV    → "차가 잠깐 멈췄어요"
🟣 SuspendedEVSE  → "충전기가 잠깐 멈췄어요"
🟤 Finishing      → "거의 끝났어요"
🔴 Reserved       → "예약됐어요"
⚫ Unavailable    → "고장났거나 사용 불가"
⚠️ Faulted        → "에러 발생!"

상태가 바뀔 때마다 중앙 시스템에 알려줘요!
```

---

### 🎯 9가지 상태 완전 정리

#### 1️⃣ Available (사용 가능)

```
🟢 비어 있고 충전 가능한 상태

🔌 커넥터: 비어 있음
⚡ 충전: 가능
👤 사용자: 기다림
```

**다음 상태로 전환 가능:**
- `Preparing` → 사용자가 플러그 꽂거나 카드 태그 (A2)
- `Charging` → 인증 없는 충전기에서 바로 충전 시작 (A3)
- `Reserved` → 예약 메시지 수신 (A7)
- `Unavailable` → Change Availability 명령 (A8)
- `Faulted` → 오류 발생 (A9)

**실제 예시:**
```json
{
  "connectorId": 1,
  "status": "Available",
  "errorCode": "NoError",
  "timestamp": "2026-01-14T10:00:00Z"
}
```

---

#### 2️⃣ Preparing (준비 중)

```
🟡 충전 준비 중

🔌 커넥터: 플러그 꽂힘 or 카드 태그됨
⚡ 충전: 아직 시작 안 됨
👤 사용자: 인증 대기 or 차량 준비 대기
```

**다음 상태로 전환 가능:**
- `Available` → 사용자가 플러그 빼거나 타임아웃 (B1)
- `Charging` → 모든 조건 충족, 충전 시작 (B3)
- `SuspendedEV` → 조건은 OK, EV가 충전 안 함 (B4)
- `SuspendedEVSE` → 조건은 OK, EVSE가 충전 안 함 (B5)
- `Finishing` → idTag 타임아웃 (B6)
- `Faulted` → 오류 발생 (B9)

**실제 예시 (카드 태그 후):**
```json
{
  "connectorId": 1,
  "status": "Preparing",
  "errorCode": "NoError",
  "info": "idTag presented, waiting for vehicle",
  "timestamp": "2026-01-14T10:05:00Z"
}
```

---

#### 3️⃣ Charging (충전 중)

```
🔵 정상 충전 중!

🔌 커넥터: 플러그 연결됨
⚡ 충전: 진행 중
👤 사용자: 없음 (충전 진행)
💰 요금: 과금 중
```

**다음 상태로 전환 가능:**
- `Available` → EV 측에서 케이블 제거 (C1)
- `SuspendedEV` → EV가 충전 중지 요청 (C4)
- `SuspendedEVSE` → EVSE가 충전 일시중지 (스마트 충전) (C5)
- `Finishing` → 사용자가 중지 버튼 or Remote Stop (C6)
- `Unavailable` → 충전 종료 + 커넥터 Unavailable 예약 (C8)
- `Faulted` → 오류 발생 (C9)

**실제 예시:**
```json
{
  "connectorId": 1,
  "status": "Charging",
  "errorCode": "NoError",
  "timestamp": "2026-01-14T10:10:00Z"
}
```

---

#### 4️⃣ SuspendedEV (EV가 일시중지)

```
🟠 차량이 충전을 멈춤

🔌 커넥터: 연결됨
⚡ 충전: 일시중지 (EV 요청)
👤 사용자: 없음
📊 이유: EV 배터리 관리, 차량 설정 등
```

**다음 상태로 전환 가능:**
- `Available` → 충전 세션 종료 (D1)
- `Charging` → EV가 충전 재개 (D3)
- `SuspendedEVSE` → EVSE도 일시중지 (D5)
- `Finishing` → 트랜잭션 중지 (D6)
- `Faulted` → 오류 발생 (D9)

**실제 예시 (EV 배터리 80% 도달):**
```json
{
  "connectorId": 1,
  "status": "SuspendedEV",
  "errorCode": "NoError",
  "info": "EV requested pause (battery management)",
  "timestamp": "2026-01-14T10:30:00Z"
}
```

**특수 케이스 - EV 측 케이블 분리:**
```json
{
  "connectorId": 1,
  "status": "SuspendedEV",
  "errorCode": "NoError",
  "info": "EV side disconnected",
  "timestamp": "2026-01-14T10:35:00Z"
}
```
→ `StopTransactionOnEVSideDisconnect = false` 설정 시 (트랜잭션 계속!)

---

#### 5️⃣ SuspendedEVSE (EVSE가 일시중지)

```
🟣 충전기가 충전을 멈춤

🔌 커넥터: 연결됨
⚡ 충전: 일시중지 (EVSE 요청)
👤 사용자: 없음
📊 이유: 스마트 충전, 부하 분산, 전력 제한
```

**우선순위 규칙:**
> ⚠️ EV + EVSE 둘 다 일시중지 → **SuspendedEVSE 우선!** (SHALL)

**다음 상태로 전환 가능:**
- `Available` → 충전 세션 종료 (E1)
- `Charging` → EVSE 제한 해제 (E3)
- `SuspendedEV` → EVSE는 OK, EV가 충전 안 함 (E4)
- `Finishing` → 트랜잭션 중지 (E6)
- `Faulted` → 오류 발생 (E9)

**실제 예시 (스마트 충전):**
```json
{
  "connectorId": 1,
  "status": "SuspendedEVSE",
  "errorCode": "NoError",
  "info": "Smart charging limit: 0A (peak time)",
  "timestamp": "2026-01-14T14:00:00Z"
}
```

---

#### 6️⃣ Finishing (종료 중)

```
🟤 충전 끝, 사용자 액션 대기

🔌 커넥터: 플러그 꽂혀 있음
⚡ 충전: 완료/중지됨
👤 사용자: 케이블 제거 필요
💰 요금: 정산 완료
```

**다음 상태로 전환 가능:**
- `Available` → 사용자가 모든 액션 완료 (F1)
- `Preparing` → 사용자가 케이블 재연결 (새 트랜잭션!) (F2)
- `Unavailable` → 모든 액션 완료 + 커넥터 Unavailable 예약 (F8)
- `Faulted` → 오류 발생 (F9)

**실제 예시:**
```json
{
  "connectorId": 1,
  "status": "Finishing",
  "errorCode": "NoError",
  "info": "Transaction stopped, please remove cable",
  "timestamp": "2026-01-14T11:00:00Z"
}
```

**특수 케이스 - EV 측 분리로 인한 종료:**
```json
{
  "connectorId": 1,
  "status": "Finishing",
  "errorCode": "NoError",
  "info": "EV side disconnected",
  "timestamp": "2026-01-14T11:05:00Z"
}
```
→ `StopTransactionOnEVSideDisconnect = true` 설정 시 (트랜잭션 종료!)

---

#### 7️⃣ Reserved (예약됨)

```
🔴 특정 사용자만 사용 가능

🔌 커넥터: 비어 있음
⚡ 충전: 예약된 사용자만 가능
👤 사용자: 예약자 대기
⏰ 만료: expiryDate까지
```

**다음 상태로 전환 가능:**
- `Available` → 예약 만료 or Cancel Reservation (G1)
- `Preparing` → 예약 idTag 제시 (G2)
- `Unavailable` → 예약 만료 + 커넥터 Unavailable 예약 (G8)
- `Faulted` → 오류 발생 (G9)

**실제 예시:**
```json
{
  "connectorId": 1,
  "status": "Reserved",
  "errorCode": "NoError",
  "info": "Reserved for idTag: ABCD1234",
  "timestamp": "2026-01-14T09:00:00Z"
}
```

---

#### 8️⃣ Unavailable (사용 불가)

```
⚫ 사용할 수 없는 상태

🔌 커넥터: 비활성화
⚡ 충전: 불가
👤 사용자: 사용 불가
🔧 이유: 관리자 설정, 펌웨어 업데이트 등
```

**특징:**
- Change Availability 명령으로 설정
- **재부팅 후에도 유지** (MUST)
- 관리자가 Available로 바꿔야 사용 가능

**다음 상태로 전환 가능:**
- `Available` → Change Availability로 Available 설정 (H1)
- `Preparing` → Available 설정 + 사용자 인터랙션 (H2)
- `Charging` → Available 설정 + 즉시 충전 (H3)
- `SuspendedEV` → H3 + EV가 충전 안 함 (H4)
- `SuspendedEVSE` → H3 + EVSE가 충전 안 함 (H5)
- `Faulted` → 오류 발생 (H9)

**실제 예시:**
```json
{
  "connectorId": 1,
  "status": "Unavailable",
  "errorCode": "NoError",
  "info": "Disabled by admin for maintenance",
  "timestamp": "2026-01-14T08:00:00Z"
}
```

---

#### 9️⃣ Faulted (오류)

```
⚠️ 오류 발생, 충전 불가

🔌 커넥터: 상황에 따라 다름
⚡ 충전: 불가
👤 사용자: 사용 불가
🔧 조치: 오류 해결 필요
```

**다음 상태로 전환 가능:**
- 오류 해결 시 → 오류 발생 이전 상태로 복귀 (I1~I8)

**실제 예시 (과전류):**
```json
{
  "connectorId": 1,
  "status": "Faulted",
  "errorCode": "OverCurrentFailure",
  "info": "Circuit breaker tripped",
  "timestamp": "2026-01-14T12:00:00Z"
}
```

**경고 vs 에러:**
```json
// ⚠️ 경고 (충전 가능)
{
  "connectorId": 1,
  "status": "Charging",
  "errorCode": "HighTemperature",
  "timestamp": "2026-01-14T13:00:00Z"
}

// 🚨 에러 (충전 불가)
{
  "connectorId": 1,
  "status": "Faulted",
  "errorCode": "OverCurrentFailure",
  "timestamp": "2026-01-14T13:05:00Z"
}
```

---

### 📊 상태 전환 다이어그램

#### 정상 충전 시나리오

```
🟢 Available
    ↓ (사용자 카드 태그)
🟡 Preparing
    ↓ (차량 연결, 인증 완료)
🔵 Charging
    ↓ (배터리 완충)
🟠 SuspendedEV
    ↓ (사용자 중지 버튼)
🟤 Finishing
    ↓ (케이블 제거)
🟢 Available
```

#### 스마트 충전 시나리오

```
🔵 Charging (20kW)
    ↓ (피크 시간, 부하 분산)
🟣 SuspendedEVSE (0kW)
    ↓ (피크 종료)
🔵 Charging (20kW 재개)
    ↓ (충전 완료)
🟤 Finishing
```

#### EV 측 분리 시나리오

**StopTransactionOnEVSideDisconnect = false:**
```
🔵 Charging
    ↓ (EV 측 케이블 분리)
🟠 SuspendedEV
    ↓ (케이블 재연결)
🔵 Charging (같은 트랜잭션!)
```

**StopTransactionOnEVSideDisconnect = true:**
```
🔵 Charging
    ↓ (EV 측 케이블 분리)
🟤 Finishing (트랜잭션 종료)
    ↓ (케이블 재연결)
🟡 Preparing (새 트랜잭션!)
```

---

### ⚙️ MinimumStatusDuration (상태 지속 최소 시간)

#### 문제 상황

OCPP 1.6에서는 `Occupied` 1개 상태가 5개로 분할되어 StatusNotification이 폭증할 수 있습니다:

```
실제 충전기 동작 (1초 안에):
10:00:00.0 → Preparing
10:00:00.2 → SuspendedEV
10:00:00.5 → SuspendedEVSE
10:00:00.8 → Charging
```
→ 0.8초 안에 StatusNotification 4번 전송? 😱

#### 해결책: MinimumStatusDuration

**짧은 시간만 유지되는 상태는 전송하지 않기!**

```json
{
  "key": "MinimumStatusDuration",
  "value": "2"  // 단위: 초
}
```

**동작 방식:**

```
제조업체 기본 필터: 0.3초 (하드웨어 노이즈 제거)
설정값: 2초
총 대기 시간 = 0.3 + 2 = 2.3초

10:00:00.0 → Preparing (0.2초만 유지)
             → 0.2 < 2.3 ❌ 전송 안 함

10:00:00.2 → SuspendedEV (0.3초만 유지)
             → 0.3 < 2.3 ❌ 전송 안 함

10:00:00.5 → SuspendedEVSE (0.3초만 유지)
             → 0.3 < 2.3 ❌ 전송 안 함

10:00:00.8 → Charging (계속 유지)
             → 2.3초 경과 후 ✅ 전송!
```

#### 핵심 개념

**1. 총 대기 시간 = 제조업체 기본값 + MinimumStatusDuration**

```typescript
shouldSendNotification(stateDuration) {
  const manufacturerDefault = 0.3; // 하드웨어 필터 (300ms)
  const configured = getConfig('MinimumStatusDuration') || 0;
  
  const totalMinimum = manufacturerDefault + configured;
  
  return stateDuration >= totalMinimum;
}
```

**2. 제조업체 기본값을 없앨 수 없음**

```
MinimumStatusDuration = 0으로 설정해도
→ 제조업체 기본값(예: 0.3초)은 유지됨!
→ 하드웨어 노이즈는 여전히 필터링됨
```

**3. 너무 크게 설정하면 모든 알림이 지연됨**

```
MinimumStatusDuration = 10초로 설정

문제:
10:00:00 → Charging 시작
10:00:05 → 사용자 중지 (실제로는 Finishing 상태)
10:00:10 → Charging 상태 전송 😱 (이미 끝났는데!)

→ 실시간성 상실!
```

#### 실전 예시

**적절한 설정 (1~2초):**
```
MinimumStatusDuration = 1

→ 총 대기: 0.3 + 1 = 1.3초
→ 중간 과정(Preparing, Suspended 등) 필터링
→ 실제 충전 상태(Charging)만 전송
→ ✅ 트래픽 감소 + 실시간성 유지
```

**너무 작은 설정 (0초):**
```
MinimumStatusDuration = 0

→ 총 대기: 0.3 + 0 = 0.3초 (제조업체 기본만)
→ 하드웨어 노이즈만 필터링
→ 짧은 상태 변화도 대부분 전송
→ ⚠️ StatusNotification 과다
```

**너무 큰 설정 (10초):**
```
MinimumStatusDuration = 10

→ 총 대기: 0.3 + 10 = 10.3초
→ 모든 상태가 10초 이상 지연
→ ❌ 실시간 모니터링 불가
```

---

### 🔌 ConnectorId 0의 특별한 규칙

#### ConnectorId 0 = 충전기 전체

```
📦 충전기 (ConnectorId 0)
    ├─ 🔌 커넥터 1
    ├─ 🔌 커넥터 2
    └─ 🔌 커넥터 3
```

**제한된 상태 (3개만):**
- ✅ Available
- ✅ Unavailable
- ✅ Faulted

**사용 불가 (6개):**
- ❌ Preparing, Charging, SuspendedEV, SuspendedEVSE, Finishing, Reserved

**독립성:**
- ConnectorId 0 상태 ≠ 개별 커넥터 상태

---

### 🔄 오프라인 재연결 시 규칙

#### 규칙 1: 현재 상태만 전송 (SHOULD)

```
오프라인 중: Available → Preparing → Charging (현재)

재연결 후:
✅ StatusNotification (Charging) 전송
❌ Available, Preparing 전송 안 함
```

**예시:**
```json
// 재연결 시 현재 상태만 전송
{
  "connectorId": 1,
  "status": "Charging",  // 현재 상태
  "errorCode": "NoError",
  "timestamp": "2026-01-14T12:00:00Z"
}
```

---

#### 규칙 2: 오류는 전송 가능 (MAY)

오프라인 중 발생한 **오류**는 특별히 보고할 수 있습니다.

**예시:**
```json
{
  "connectorId": 1,
  "status": "Faulted",
  "errorCode": "GroundFailure",
  "info": "Error occurred while offline at 03:00",
  "timestamp": "2026-01-14T03:00:00Z"
}
```

---

#### 규칙 3: 과거 이벤트는 전송 안 함 (SHOULD NOT)

현재 상태나 오류를 알리지 않는 과거 상태 변경은 전송하지 않습니다.

**예시:**
```
오프라인 중:
Available → Reserved → Preparing → Available (현재)

재연결 후:
✅ StatusNotification (Available) 전송
❌ Reserved, Preparing 전송 안 함
   (이미 지나간 과거 상태)
```

---

#### 규칙 4: 순서 보장 (MUST)

**전송하기로 결정한** 메시지들은 발생 순서대로 전송해야 합니다.

**케이스 1: 현재 상태만 전송 (일반적)**
```
오프라인 중:
Available → Preparing → Charging (현재)

재연결 후:
✅ StatusNotification (Charging, 현재)
❌ Available, Preparing 전송 안 함
```

**케이스 2: 오류 보고 결정 시 (선택적)**
```
오프라인 중:
Charging → Faulted (OverCurrent, 10시) → Available (11시, 현재)

재연결 후 - 옵션 A (오류 보고 안 함):
✅ StatusNotification (Available, 11시)  // 현재만

재연결 후 - 옵션 B (오류 보고함):
1️⃣ StatusNotification (Faulted, OverCurrent, 10시)  // 순서 보장!
2️⃣ StatusNotification (Available, 11시)
```

**케이스 3: 여러 오류 보고 시**
```
오프라인 중:
Charging → Faulted (OverCurrent, 10시) 
        → Faulted (GroundFailure, 11시, 현재)

재연결 후 전송 순서 (모두 보고하기로 결정):
1️⃣ StatusNotification (Faulted, OverCurrent, 10시)
2️⃣ StatusNotification (Faulted, GroundFailure, 11시)

⚠️ 순서가 중요! 시간순으로 전송해야 함
```

**케이스 4: 현재 오류 상태**
```
오프라인 중:
Charging → Faulted (10시, 현재 계속)

재연결 후:
✅ StatusNotification (Faulted, 현재)
   // 현재 상태이므로 반드시 전송
```

---

### 💡 EVCommunicationError 특수 규칙

#### 사용 가능한 상태 (4개만)

- ✅ Preparing
- ✅ SuspendedEV
- ✅ SuspendedEVSE
- ✅ Finishing

**사용 불가:**
- ❌ Available, Charging, Reserved, Unavailable, Faulted

**경고로 처리:**
```json
{
  "connectorId": 1,
  "status": "Preparing",
  "errorCode": "EVCommunicationError",
  "info": "Cannot communicate with vehicle",
  "timestamp": "2026-01-14T10:15:00Z"
}
```

---

### 💡 핵심 요약

```
📌 상태는 9개
   → Available, Preparing, Charging
   → SuspendedEV, SuspendedEVSE
   → Finishing, Reserved
   → Unavailable, Faulted

📌 ConnectorId 0은 3개만
   → Available, Unavailable, Faulted

📌 우선순위
   → EV+EVSE 동시 일시중지 → SuspendedEVSE

📌 최적화
   → MinimumStatusDuration으로 필터링

📌 오프라인 재연결
   → 현재 상태만 전송, 순서 보장

📌 경고 vs 에러
   → status ≠ Faulted → 경고
   → status = Faulted → 에러
```

