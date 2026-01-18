# OCPP 1.6 - Chapter 4 Cheat Sheet v2

> **목적**: 번역 검수하면서 프로토콜 이해를 돕는 예시 작성  
> **작성일**: 2026-01-13  
> **업데이트**: 2026-01-14 (4.9 Status Notification 추가)  
> **방식**: 단계별로 검수하면서 채워나가기

---

## 📋 목차

- [4.2 Boot Notification](#-42-boot-notification-부팅-알림)
- [4.7 Meter Values](#-47-meter-values-미터-값)
- [4.8 Start Transaction](#-48-start-transaction-트랜잭션-시작) ⭐ NEW
- [4.9 Status Notification](#-49-status-notification-상태-알림)

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
  "interval": 300
  // 5분마다 Heartbeat
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
  "interval": 60
  // 1분 후 다시 BootNotification
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
  "interval": 3600
  // 1시간 후 재시도
}
```

---

### 🔑 interval 값의 의미

| Status       | interval의 의미            | 전송 메시지                 | 예시          |
|--------------|-------------------------|------------------------|-------------|
| **Accepted** | Heartbeat 주기            | `Heartbeat.req`        | 300초(5분)마다  |
| **Pending**  | BootNotification 재전송 주기 | `BootNotification.req` | 60초마다       |
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

### 🎭 역할별 책임 매트릭스

#### 📋 프로세스 단계별 책임

| 단계                     | 충전기 (CP) 개발자 👨‍💻                                                                                                                                               | 중앙 시스템 (CSMS) 개발자 👩‍💻                                                                                                                             |
|------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------|
| **1️⃣ 부팅 및 요청**        | ✅ **MUST**: 재부팅마다 BootNotification 전송<br>✅ **MUST**: 정확한 제품 정보 전송<br>- chargePointVendor<br>- chargePointModel<br>- chargePointSerialNumber<br>- firmwareVersion | -                                                                                                                                                   |
| **2️⃣ 검증 및 Status 결정** | -                                                                                                                                                                | ✅ **SHOULD**: BootNotification에 응답<br>✅ 충전기 등록 여부 확인<br>✅ Status 결정:<br>- **Accepted**: 정상 등록<br>- **Pending**: 설정 동기화 필요<br>- **Rejected**: 미등록/차단 |
| **3️⃣ 응답 및 시간 동기화**    | ✅ **MUST**: currentTime 수신 후 시간 동기화<br>✅ **MUST**: interval 값 저장                                                                                                 | ✅ **MUST**: currentTime 포함 응답<br>✅ **MUST**: interval 값 포함<br>(Heartbeat 주기 또는 재시도 주기)                                                              |
| **4️⃣ Accepted 시**     | ✅ **MUST**: interval마다 Heartbeat 전송<br>✅ 모든 기능 활성화<br>✅ 충전 가능 상태                                                                                                 | -                                                                                                                                                   |
| **5️⃣ Pending 시**      | ✅ **MUST NOT**: 트랜잭션 시작 불가<br>(Authorize, Start/StopTransaction)<br>✅ **MUST**: CSMS 요청에 응답<br>✅ **MUST**: interval 후 BootNotification 재전송                       | ✅ Pending 응답 전송<br>✅ 이후 Accepted로 변경 가능                                                                                                            |
| **6️⃣ Rejected 시**     | ✅ **MUST**: 모든 기능 비활성화<br>✅ **MUST**: interval 후 재시도<br>✅ **MUST NOT**: 재시도 포기 금지                                                                                | ✅ Rejected 응답 전송<br>✅ 충분히 긴 interval 설정 권장<br>(예: 3600초)                                                                                             |

---

#### ⏰ Interval 관리

| Status       | 충전기 (CP)                                                                 | 중앙 시스템 (CSMS)                        |
|--------------|--------------------------------------------------------------------------|--------------------------------------|
| **Accepted** | ✅ **MUST**: interval마다 **Heartbeat** 전송                                  | ✅ Heartbeat 주기 설정<br>- 권장: 300초 (5분) |
| **Pending**  | ✅ **MUST**: interval 후 **BootNotification** 재전송                          | ✅ 재전송 주기 설정<br>- 권장: 30~60초          |
| **Rejected** | ✅ **MUST**: interval 후 **BootNotification** 재시도<br>✅ **MUST NOT**: 포기 금지 | ✅ 긴 재시도 주기 설정<br>- 권장: 3600초 (1시간)   |

---

#### 🔐 필수 검증 항목

| 항목         | 충전기 (CP)                                                                                                             | 중앙 시스템 (CSMS)                                                                                       |
|------------|----------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------|
| **제품 정보**  | ✅ **MUST**: 정확한 정보 전송<br>- chargePointVendor<br>- chargePointModel<br>- chargePointSerialNumber<br>- firmwareVersion | ✅ **MUST**: 모든 요청을 이벤트 로그로 저장<br>✅ 등록 여부 확인<br>✅ 상태 테이블 업데이트 (조회용)                              |
| **시간 동기화** | ✅ **MUST**: currentTime으로 시간 동기화                                                                                     | ✅ **MUST**: 정확한 currentTime 제공 (UTC)                                                                |
| **이벤트 추적** | ✅ 응답 받으면 재전송 중지                                                                                                      | ✅ **MUST**: 모든 요청/응답 저장<br>✅ 문제 추적용 감사 로그 유지<br>✅ CP/CSMS 문제 분리 추적<br>✅ 상태 일관성 유지 (UPSERT 상태 테이블) |

---

#### ⚠️ Pending 상태 제한사항

| 기능                     | Accepted     | Pending        | Rejected     |
|------------------------|--------------|----------------|--------------|
| **충전기가 전송 가능**         |              |                |              |
| Authorize              | ✅            | ❌ **MUST NOT** | ❌            |
| StartTransaction       | ✅            | ❌ **MUST NOT** | ❌            |
| StopTransaction        | ✅            | ❌ **MUST NOT** | ❌            |
| Heartbeat              | ✅ interval마다 | ❌              | ❌            |
| BootNotification       | ✅ 재부팅 시      | ✅ interval마다   | ✅ interval마다 |
| **CSMS가 전송 가능**        |              |                |              |
| GetConfiguration       | ✅            | ✅ **MAY**      | ❌            |
| ChangeConfiguration    | ✅            | ✅ **MAY**      | ❌            |
| Reset                  | ✅            | ✅ **MAY**      | ❌            |
| UpdateFirmware         | ✅            | ✅ **MAY**      | ❌            |
| RemoteStartTransaction | ✅            | ❌              | ❌            |

**핵심**: Pending 상태에서도 CSMS는 **설정 관련 명령을 보낼 수 있습니다!**

---

#### 📌 핵심 체크리스트

**충전기 개발자 ✅**

- [ ] **MUST**: 재부팅마다 BootNotification 전송
- [ ] **MUST**: 정확한 제품 정보 전송
- [ ] **MUST**: currentTime으로 시간 동기화
- [ ] **MUST**: Accepted 시 interval마다 Heartbeat 전송
- [ ] **MUST NOT**: Pending 시 트랜잭션 시작 금지
- [ ] **MUST**: Pending 시 interval 후 재전송
- [ ] **MUST NOT**: Rejected 시 재시도 포기 금지

**중앙 시스템 개발자 ✅**

- [ ] **SHOULD**: BootNotification에 응답
- [ ] **MUST**: currentTime 포함
- [ ] **MUST**: interval 값 포함
- [ ] 등록 여부 확인 후 Status 결정
- [ ] Pending 시 설정 동기화 수행
- [ ] 멱등성 보장 (중복 요청 처리)

---

#### 💡 핵심 정리

```
BootNotification = 충전기 등록 및 시간 동기화

MUST (필수):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ CP: 재부팅마다 전송
✅ CP: currentTime으로 시간 동기화
✅ CP: Accepted 시 interval마다 Heartbeat
✅ CP: Pending/Rejected 시 interval 후 재전송
✅ CSMS: BootNotification.conf 응답 전송 (SHALL)
✅ CSMS: currentTime, interval 포함 응답
✅ CSMS: 모든 요청/응답 이벤트 로그 저장

MUST NOT (금지):
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
❌ CP: Pending 시 트랜잭션 시작
❌ CP: Rejected 시 재시도 포기

참고:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
💡 SHALL = MUST (OCPP 스펙 용어)
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

| 타이밍       | Context             | 설명                      |
|-----------|---------------------|-------------------------|
| **주기적**   | `Sample.Periodic`   | 60초마다 자동 전송 (설정 가능)     |
| **충전 시작** | `Transaction.Begin` | StartTransaction과 함께 전송 |
| **충전 종료** | `Transaction.End`   | StopTransaction과 함께 전송  |

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

| Measurand                       | 의미          | 단위      | 비유                                 | 예시      |
|---------------------------------|-------------|---------|------------------------------------|---------|
| `Energy.Active.Import.Register` | **누적 전력량**  | kWh     | 🚗 **주행거리계 (ODO)** - 출발지부터 총 누적 거리 | `25.5`  |
| `Power.Active.Import`           | **현재 전력**   | kW      | 🚗 **속도계 (SPEED)** - 지금 이 순간의 속도   | `7.2`   |
| `Current.Import`                | **전류**      | A       | 💧 **수도관의 유속** - 물이 흐르는 속도         | `32.5`  |
| `Voltage`                       | **전압**      | V       | 💧 **수압** - 물이 밀리는 압력              | `220.5` |
| `SoC`                           | **배터리 충전률** | %       | 📱 **스마트폰 배터리 %** - 얼마나 찼는지        | `45`    |
| `Temperature`                   | **온도**      | Celsius | 🌡️ **체온계** - 발열 체크                | `35`    |

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

| Context             | 의미       | 설명             |
|---------------------|----------|----------------|
| `Sample.Periodic`   | 주기적 측정   | 60초마다 자동 전송    |
| `Transaction.Begin` | 충전 시작 시점 | 초기값 기록         |
| `Transaction.End`   | 충전 종료 시점 | 최종값 기록 (과금 계산) |

#### 3️⃣ **phase** (어느 선로?)

| Phase                     | 의미              | 설명                   |
|---------------------------|-----------------|----------------------|
| `L1`, `L2`, `L3`          | 3상 전기의 각 선로     | Line 1, 2, 3         |
| `L1-N`, `L2-N`, `L3-N`    | 각 선로와 중성선 사이 전압 | Phase Voltage (230V) |
| `L1-L2`, `L2-L3`, `L3-L1` | 선로 간 전압         | Line Voltage (400V)  |
| `N`                       | 중성선             | Neutral              |

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

| Location | 의미      | 설명            |
|----------|---------|---------------|
| `Inlet`  | 충전기 입력단 | 전기가 들어오는 곳    |
| `Outlet` | 충전기 출력단 | 케이블 끝 (주로 사용) |
| `Body`   | 충전기 본체  | 온도 측정 등       |
| `Cable`  | 케이블     | 케이블 온도 등      |
| `EV`     | 전기차 내부  | SoC 등         |

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
      "value": "85.0"
      // ❌ 중복!
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
실시간
요금 = (현재
Energy.Active.Import.Register - meterStart
)
× 요금(원 / kWh)

// 예시
현재
Energy = 25.5
kWh(MeterValues에서
수신
)
시작
Energy = 0
kWh(StartTransaction의
meterStart
)
요금 = 300
원 / kWh

실시간
요금 = (25.5 - 0) × 300 = 7, 650
원
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

| Measurand                       | Phase 구분 필요? | 이유           |
|---------------------------------|--------------|--------------|
| `Current.Import`                | ✅ **필요**     | 3상 부하 균형 확인  |
| `Current.Export`                | ✅ **필요**     | 3상 각각 측정     |
| `Voltage`                       | ✅ **필요**     | 각 상의 전압 모니터링 |
| `Power.Active.Import`           | △ **선택**     | 총합 또는 상별     |
| `Energy.Active.Import.Register` | ❌ **불필요**    | 전체 누적값만      |
| `Temperature`                   | ❌ **불필요**    | Phase 개념 없음  |
| `SoC`                           | ❌ **불필요**    | 배터리 전체 상태    |

---

### 🎯 사용 목적별 정리

| 목적            | 사용 Measurand                               | 주기                    |
|---------------|--------------------------------------------|-----------------------|
| **과금 계산**     | `Energy.Active.Import.Register`            | Transaction.End       |
| **실시간 요금 표시** | `Energy.Active.Import.Register`            | Sample.Periodic (60초) |
| **충전 속도 표시**  | `Power.Active.Import`                      | Sample.Periodic       |
| **안전 모니터링**   | `Current.Import`, `Voltage`, `Temperature` | Sample.Periodic       |
| **부하 균형 체크**  | `Current.Import` (L1, L2, L3)              | Sample.Periodic       |
| **배터리 상태 표시** | `SoC`                                      | Sample.Periodic       |

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

**다음 작성 예정**: 4.1 Authorize, 4.3 Data Transfer, ...  
*(번역 검수하면서 하나씩 추가)*

---

## 📝 4.8 Start Transaction (트랜잭션 시작)

### 📌 핵심 개념

**Start Transaction**은 충전기가 "충전 시작했어요! 요금 계산 시작해주세요!" 하고 중앙 시스템에 알리는 메시지입니다.

### 🧒 어린이용 설명

```
🚗 차량 도착, RFID 카드 태그
    ↓
🔐 Authorize.req → ✅ Accepted
    ↓
🔌 플러그 연결, 충전 시작!
    ↓
📢 StartTransaction.req
    "충전 시작했어요! 요금 계산해주세요!"
    - connectorId: 1번 커넥터
    - idTag: ABCD1234
    - meterStart: 0 kWh (시작 전력량)
    - timestamp: 2026-01-17T10:00:00Z
    ↓
🏢 Central System이 확인
    ✅ "카드 정말 유효한가?" (다시 한번 검증!)
    ✅ "혹시 차단된 카드 아닌가?"
    ✅ "Authorization Cache에 있는 구버전 정보 아닌가?"
    ↓
📩 StartTransaction.conf
    - transactionId: 12345 (요금 계산용 ID)
    - idTagInfo.status: Accepted
    ↓
⚡ 충전 시작! (MeterValues 전송 시작)
```

---

### 🔑 핵심 필드

#### 요청 (StartTransaction.req)

```typescript
{
    connectorId: 1,           // 어떤 커넥터?
        idTag
:
    "ABCD1234",        // 누구?
        meterStart
:
    0,            // 시작 시점 전력량 (kWh)
        timestamp
:
    "2026-01-17T10:00:00Z",  // 언제?
        reservationId ? : 5         // 예약 종료? (선택)
}
```

#### 응답 (StartTransaction.conf)

```typescript
{
    transactionId: 12345,     // 트랜잭션 ID (요금 계산용)
        idTagInfo
:
    {
        status: "Accepted",     // 최종 인증 상태
            expiryDate ? : "2027-12-31T23:59:59Z",
            parentIdTag ? : "MASTER001"
    }
}
```

---

### 🎯 핵심 규칙

#### 1️⃣ **예약 종료 처리**

예약으로 시작된 트랜잭션은 `reservationId`를 **반드시** 포함해야 합니다!

**시나리오: 예약 후 충전**

```
09:00 → 예약 생성 (ReserveNow.req)
        reservationId: 5
        idTag: ABCD1234

10:00 → 예약자 도착, 카드 태그
        ↓
        StartTransaction.req
        {
          "connectorId": 1,
          "idTag": "ABCD1234",
          "meterStart": 0,
          "reservationId": 5  // ✅ 필수!
        }
        ↓
        예약 자동 종료 ✅
```

---

#### 2️⃣ **중앙 시스템의 2차 검증 (중요!)**

충전기가 로컬에서 이미 인증했어도, 중앙 시스템이 **다시 한번** 검증합니다!

**왜?** 충전기의 Authorization Cache가 **오래된 정보**일 수 있기 때문!

**실제 케이스:**

```
📅 1월 1일
   → idTag "ABCD1234"가 충전기 Authorization Cache에 추가
   → status: Accepted

📅 1월 10일
   → 중앙 시스템에서 카드 차단 (분실 신고)
   → 하지만 충전기는 모름! (Cache는 그대로)

📅 1월 15일
   → 사용자가 차단된 카드로 충전 시도
   → 충전기: Authorize.req → Accepted (Cache 사용)
   → 충전기: StartTransaction.req
   → 중앙 시스템: 🚨 "이 카드는 차단됐어!"
   → StartTransaction.conf
     {
       "transactionId": 12345,
       "idTagInfo": {
         "status": "Blocked"  // ❌ 거부!
       }
     }
   → 충전기: 트랜잭션 즉시 중지!
```

---

#### 3️⃣ **Authorization Cache 업데이트**

충전기가 Authorization Cache를 사용하는 경우, 응답을 받으면 **캐시를 업데이트**해야 합니다!

**조건:**

- ✅ Authorization Cache 구현됨
- ✅ idTag가 **Local Authorization List에 없음**

**동작:**

```typescript
// StartTransaction.conf 수신
{
    "transactionId"
:
    12345,
        "idTagInfo"
:
    {
        "status"
    :
        "Blocked",
            "expiryDate"
    :
        "2027-12-31T23:59:59Z"
    }
}

// 충전기가 Authorization Cache 업데이트
cache.update("ABCD1234", {
    status: "Blocked",
    expiryDate: "2027-12-31T23:59:59Z",
    lastUpdated: "2026-01-17T10:00:00Z"
});

// 다음 Authorize.req부터 차단됨!
```

---

#### 4️⃣ **정상성 검사 vs 응답**

중앙 시스템이 데이터 이상을 발견해도 **반드시 응답**해야 합니다!

**잘못된 데이터 예시:**

```json
// 이상한 StartTransaction.req
{
  "connectorId": 999,
  // ⚠️ 존재하지 않는 커넥터
  "idTag": "ABCD1234",
  "meterStart": -100,
  // ⚠️ 음수 전력량?
  "timestamp": "invalid"
  // ⚠️ 잘못된 형식
}
```

**중앙 시스템의 올바른 처리:**

```typescript
// ❌ 잘못된 처리 - 응답 안 함
if (connectorId > maxConnectors) {
    return; // 🚨 절대 안 됨!
}

// ✅ 올바른 처리 - 항상 응답
if (connectorId > maxConnectors) {
    logger.warn("Invalid connectorId:", connectorId);
    // 그래도 응답은 보냄!
    return {
        transactionId: generateId(),
        idTagInfo: {status: "Invalid"}
    };
}
```

**왜?**

- 응답 안 하면 충전기가 **계속 재전송**함 (무한 반복)
- 네트워크 부하 증가
- 충전기 동작 멈춤

---

### 📊 실전 시나리오

#### 시나리오 1: 정상 충전 시작

```
10:00:00 → 사용자 카드 태그
           ↓
10:00:01 → CP: Authorize.req
           {
             "idTag": "ABCD1234"
           }
           ↓
10:00:02 → CS: Authorize.conf
           {
             "idTagInfo": { "status": "Accepted" }
           }
           ↓
10:00:05 → 사용자 플러그 연결
           ↓
10:00:06 → CP: StatusNotification.req (Preparing)
           ↓
10:00:10 → 충전 시작!
           ↓
           CP: StartTransaction.req
           {
             "connectorId": 1,
             "idTag": "ABCD1234",
             "meterStart": 0,
             "timestamp": "2026-01-17T10:00:10Z"
           }
           ↓
10:00:11 → CS: StartTransaction.conf
           {
             "transactionId": 12345,
             "idTagInfo": { "status": "Accepted" }
           }
           ↓
10:00:12 → CP: StatusNotification.req (Charging)
           ↓
10:00:20 → CP: MeterValues.req (첫 번째 측정값)
```

---

#### 시나리오 2: 로컬 인증 성공 → 중앙 시스템 거부

```
10:00:00 → 충전기가 로컬 Cache로 인증
           (Cache: ABCD1234 = Accepted, 1주일 전 데이터)
           ↓
10:00:01 → CP: Authorize.req
           → Cache Hit! → Accepted (로컬 응답)
           ↓
10:00:05 → 충전 시작
           ↓
           CP: StartTransaction.req
           {
             "idTag": "ABCD1234"
           }
           ↓
10:00:06 → CS: 중앙 DB 확인
           → 🚨 "이 카드 3일 전에 차단됐네?"
           ↓
           CS: StartTransaction.conf
           {
             "transactionId": 12345,
             "idTagInfo": { 
               "status": "Blocked"  // ❌
             }
           }
           ↓
10:00:07 → CP: Authorization Cache 업데이트
           ABCD1234 → Blocked
           ↓
           CP: StopTransaction.req (즉시 중지)
           {
             "transactionId": 12345,
             "reason": "DeAuthorized"
           }
```

---

#### 시나리오 3: 예약 사용

```
09:00:00 → CS: ReserveNow.req
           {
             "connectorId": 1,
             "expiryDate": "2026-01-17T11:00:00Z",
             "idTag": "ABCD1234",
             "reservationId": 5
           }
           ↓
09:00:01 → CP: ReserveNow.conf (Accepted)
           ↓
           CP: StatusNotification.req (Reserved)
           
           // 예약 시간 동안 대기
           
10:30:00 → 예약자 도착, 카드 태그 (ABCD1234)
           ↓
10:30:01 → CP: Authorize.req
           → Accepted
           ↓
10:30:05 → CP: StatusNotification.req (Preparing)
           ↓
10:30:10 → 충전 시작
           ↓
           CP: StartTransaction.req
           {
             "connectorId": 1,
             "idTag": "ABCD1234",
             "meterStart": 0,
             "reservationId": 5  // ✅ 예약 ID 포함!
           }
           ↓
10:30:11 → CS: StartTransaction.conf
           {
             "transactionId": 12345,
             "idTagInfo": { "status": "Accepted" }
           }
           ↓
           예약 자동 취소됨!
           ↓
10:30:12 → CP: StatusNotification.req (Charging)
```

---

#### 시나리오 4: 잘못된 데이터 → 그래도 응답

```
10:00:00 → CP: StartTransaction.req
           {
             "connectorId": 999,    // ⚠️ 존재하지 않음
             "idTag": "ABCD1234",
             "meterStart": -100,    // ⚠️ 음수
             "timestamp": "invalid" // ⚠️ 잘못된 형식
           }
           ↓
10:00:01 → CS: 정상성 검사
           - connectorId 999: ⚠️ 존재하지 않음
           - meterStart -100: ⚠️ 음수
           - timestamp: ⚠️ 파싱 실패
           ↓
           ❌ 잘못된 처리: 응답 안 함
           → 충전기가 계속 재전송! (무한 반복)
           
           ✅ 올바른 처리: 그래도 응답!
           CS: StartTransaction.conf
           {
             "transactionId": 12345,
             "idTagInfo": { "status": "Invalid" }
           }
           ↓
10:00:02 → CP: 응답 받음 → 재전송 중지 ✅
           → 사용자에게 오류 메시지 표시
```

---

### 🎭 역할별 책임 매트릭스

#### 📋 누가 무엇을 해야 하나?

| 단계                        | 충전기 (CP) 개발자 👨‍💻                                                                                                                                                                          | 중앙 시스템 (CSMS) 개발자 👩‍💻                                                                                                                    |
|---------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| **1️⃣ 충전 시작 감지**          | ✅ 플러그 연결, 차량 준비 완료 감지<br>✅ meterStart 값 측정 (kWh)<br>✅ timestamp 기록                                                                                                                          | -                                                                                                                                          |
| **2️⃣ 요청 전송**             | ✅ StartTransaction.req 전송<br>```json<br>{<br>  "connectorId": 1,<br>  "idTag": "ABCD1234",<br>  "meterStart": 0,<br>  "timestamp": "...",<br>  "reservationId": 5<br>}<br>```               | -                                                                                                                                          |
| **3️⃣ 요청 수신 및 검증**        | -                                                                                                                                                                                           | ✅ idTag 최신 상태 DB 확인<br>✅ 차단/만료 여부 확인<br>✅ 예약 ID 검증 (있으면)<br>⚠️ 데이터 정상성 검사<br>⚠️ **검사 실패해도 반드시 응답!**                                        |
| **4️⃣ Transaction ID 생성** | -                                                                                                                                                                                           | ✅ 고유 transactionId 생성<br>✅ DB에 트랜잭션 레코드 생성<br>✅ 시작 시간, meterStart 저장                                                                       |
| **5️⃣ 응답 전송**             | -                                                                                                                                                                                           | ✅ StartTransaction.conf 전송<br>```json<br>{<br>  "transactionId": 12345,<br>  "idTagInfo": {<br>    "status": "Accepted"<br>  }<br>}<br>``` |
| **6️⃣ 응답 수신 및 처리**        | ✅ transactionId 저장<br>✅ **Authorization Cache 업데이트**<br>(Local List에 없으면)<br><br>⚠️ status가 Blocked이면:<br>  └─ StopTransaction 즉시 전송<br><br>✅ status가 Accepted이면:<br>  └─ MeterValues 전송 시작 | -                                                                                                                                          |
| **7️⃣ 예약 처리**             | ✅ reservationId 포함 시<br>→ 예약 자동 취소 처리                                                                                                                                                       | ✅ reservationId 받으면<br>→ DB에서 예약 제거<br>→ 예약 상태 업데이트                                                                                        |

---

### 🔐 보안 및 검증 책임

| 항목            | 충전기 (CP)                                                                                                      | 중앙 시스템 (CSMS)                                      |
|---------------|---------------------------------------------------------------------------------------------------------------|----------------------------------------------------|
| **로컬 인증**     | ✅ Authorization Cache 확인<br>✅ Local Authorization List 확인<br>⚠️ 구버전 데이터 가능성 인식                                | -                                                  |
| **최종 인증**     | -                                                                                                             | ✅ **최신 DB로 재검증**<br>✅ 차단/만료 확인<br>✅ 사용 제한 확인       |
| **Cache 동기화** | ✅ 응답의 idTagInfo로 업데이트<br>```ts<br>if (!inLocalList(idTag)) {<br>  cache.update(idTag, idTagInfo);<br>}<br>``` | ✅ 정확한 idTagInfo 제공<br>✅ expiryDate, status 포함      |
| **재시도 방지**    | ✅ 중복 전송 방지<br>✅ 응답 받으면 재전송 중지                                                                                 | ✅ **항상 응답 전송**<br>(데이터 이상해도!)<br>⚠️ 응답 안 하면 무한 재전송 |

---

### ⚠️ 오류 처리 책임

| 오류 상황       | 충전기 (CP)                                                               | 중앙 시스템 (CSMS)                                                                                                                              |
|-------------|------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| **네트워크 오류** | ✅ 재전송 (Error responses 규칙)<br>✅ 오프라인 큐에 저장                             | ✅ 멱등성 처리<br>(같은 요청 여러번 와도 OK)                                                                                                              |
| **잘못된 데이터** | -                                                                      | ✅ 로그 기록<br>✅ **그래도 응답 전송!**<br>```json<br>{<br>  "transactionId": 12345,<br>  "idTagInfo": {<br>    "status": "Invalid"<br>  }<br>}<br>``` |
| **차단된 카드**  | ✅ Blocked 응답 받으면<br>→ 즉시 StopTransaction<br>→ Cache 업데이트<br>→ 사용자에게 안내 | ✅ status: "Blocked" 응답<br>✅ DB에 차단 이력 기록                                                                                                   |
| **DB 장애**   | -                                                                      | ✅ Fallback 로직<br>✅ 임시 승인 or 거부 결정<br>✅ **반드시 응답!**                                                                                         |

---

### 📊 데이터 관리 책임

| 데이터               | 충전기 (CP)                                                                       | 중앙 시스템 (CSMS)                                         |
|-------------------|--------------------------------------------------------------------------------|-------------------------------------------------------|
| **transactionId** | ✅ 저장 (메모리/DB)<br>✅ 이후 모든 메시지에 포함<br>- MeterValues.req<br>- StopTransaction.req | ✅ 생성 및 관리<br>✅ DB에 트랜잭션 레코드 생성<br>✅ 상태 추적 (진행중/완료/중단) |
| **meterStart**    | ✅ 측정 및 전송<br>✅ 정확한 값 보장                                                        | ✅ 저장<br>✅ StopTransaction의 meterStop과 비교              |
| **timestamp**     | ✅ 정확한 시각 기록<br>✅ NTP 동기화 권장                                                    | ✅ 저장<br>✅ 시간대 변환 처리                                   |
| **reservationId** | ✅ 예약 사용 시 포함<br>✅ 예약 취소 처리                                                     | ✅ 검증<br>✅ 예약 상태 업데이트<br>✅ DB에서 제거                     |

---

### 🎯 성능 및 확장성 책임

| 항목         | 충전기 (CP)                           | 중앙 시스템 (CSMS)                                                                                                                              |
|------------|------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| **응답 시간**  | ✅ 타임아웃 설정 (30초 권장)<br>✅ 타임아웃 시 재전송 | ✅ **빠른 응답** (1초 이내)<br>✅ 비동기 처리 고려<br>```ts<br>// 즉시 응답, 후처리는 비동기<br>respond(transactionId, idTagInfo);<br>await processAsync(...);<br>``` |
| **동시성 처리** | -                                  | ✅ 동시 다발 요청 처리<br>✅ DB 트랜잭션 관리<br>✅ ID 중복 방지 (UUID 등)                                                                                       |
| **모니터링**   | ✅ 로그 기록<br>✅ 오류 카운터                | ✅ 트랜잭션 통계<br>✅ 성능 메트릭<br>✅ 알람 설정                                                                                                           |

---

### 💻 코드 예시

#### 충전기 (Charge Point) 코드

```typescript
class ChargePoint {
    async startTransaction(connectorId: number, idTag: string) {
        // 1. 사전 검증
        const connector = this.getConnector(connectorId);
        if (!connector.isPlugged()) {
            throw new Error("Cable not connected");
        }

        // 2. 측정값 수집
        const meterStart = connector.getMeterValue();
        const timestamp = new Date().toISOString();

        // 3. 예약 확인
        const reservationId = connector.getReservationId();

        // 4. 요청 전송
        const response = await this.send({
            type: "StartTransaction",
            connectorId,
            idTag,
            meterStart,
            timestamp,
            reservationId  // 있으면 포함
        });

        // 5. 응답 처리
        const {transactionId, idTagInfo} = response;

        // 6. Cache 업데이트 (Local List에 없으면)
        if (this.hasAuthCache() && !this.isInLocalList(idTag)) {
            this.authCache.update(idTag, idTagInfo);
        }

        // 7. 상태 확인
        if (idTagInfo.status === "Blocked") {
            // 즉시 중지
            await this.stopTransaction(transactionId, "DeAuthorized");
            throw new Error("Card blocked by Central System");
        }

        // 8. 트랜잭션 시작
        this.currentTransaction = {
            id: transactionId,
            connectorId,
            idTag,
            meterStart,
            startTime: timestamp
        };

        // 9. MeterValues 전송 시작
        this.startMeterValueTimer();

        return transactionId;
    }
}
```

#### 중앙 시스템 (CSMS) 코드

```typescript
class CentralSystem {
    async handleStartTransaction(chargePointId: string, request: any) {
        const {connectorId, idTag, meterStart, timestamp, reservationId} = request;

        try {
            // 1. 데이터 정상성 검사
            if (connectorId < 0 || !idTag) {
                logger.warn("Invalid data", request);
                // ⚠️ 그래도 응답은 보냄!
                return {
                    transactionId: generateId(),
                    idTagInfo: {status: "Invalid"}
                };
            }

            // 2. idTag 최신 상태 확인 (중요!)
            const tagInfo = await db.getIdTagInfo(idTag);

            if (!tagInfo) {
                return {
                    transactionId: generateId(),
                    idTagInfo: {status: "Invalid"}
                };
            }

            if (tagInfo.status === "Blocked") {
                return {
                    transactionId: generateId(),
                    idTagInfo: {
                        status: "Blocked",
                        expiryDate: tagInfo.expiryDate
                    }
                };
            }

            if (tagInfo.expiryDate && new Date(tagInfo.expiryDate) < new Date()) {
                return {
                    transactionId: generateId(),
                    idTagInfo: {status: "Expired"}
                };
            }

            // 3. 예약 검증
            if (reservationId) {
                const reservation = await db.getReservation(reservationId);
                if (!reservation || reservation.idTag !== idTag) {
                    return {
                        transactionId: generateId(),
                        idTagInfo: {status: "Invalid"}
                    };
                }

                // 예약 제거
                await db.removeReservation(reservationId);
            }

            // 4. Transaction 생성
            const transactionId = generateUUID();
            await db.createTransaction({
                id: transactionId,
                chargePointId,
                connectorId,
                idTag,
                meterStart,
                startTime: timestamp,
                status: "InProgress"
            });

            // 5. 성공 응답
            return {
                transactionId,
                idTagInfo: {
                    status: "Accepted",
                    expiryDate: tagInfo.expiryDate,
                    parentIdTag: tagInfo.parentIdTag
                }
            };

        } catch (error) {
            logger.error("StartTransaction failed", error);

            // ⚠️ 에러 발생해도 반드시 응답!
            return {
                transactionId: generateId(),
                idTagInfo: {status: "Invalid"}
            };
        }
    }
}
```

---

### 📌 핵심 체크리스트

#### 충전기 개발자 체크리스트 ✅

- [ ] meterStart 정확히 측정
- [ ] timestamp NTP 동기화
- [ ] 예약 사용 시 reservationId 포함
- [ ] transactionId 안전하게 저장
- [ ] Authorization Cache 업데이트 (Local List 우선)
- [ ] Blocked 응답 시 즉시 중지
- [ ] 네트워크 오류 시 재전송
- [ ] 로그 및 모니터링 구현

#### 중앙 시스템 개발자 체크리스트 ✅

- [ ] **항상 응답 전송** (데이터 이상해도!)
- [ ] idTag 최신 DB로 재검증
- [ ] 고유 transactionId 생성 (UUID 권장)
- [ ] 정확한 idTagInfo 제공
- [ ] 예약 검증 및 제거
- [ ] DB 트랜잭션 관리
- [ ] 빠른 응답 시간 (1초 이내)
- [ ] 에러 처리 및 Fallback
- [ ] 멱등성 보장
- [ ] 로그 및 모니터링 구현

---

### 💡 핵심 정리

```
StartTransaction = 요금 계산 시작!

📍 언제: 충전 시작 시점
📍 목적: 
   1. 트랜잭션 ID 받기 (과금 추적)
   2. 중앙 시스템의 최종 인증 확인
   3. Authorization Cache 업데이트

핵심 포인트:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ 예약 종료 시 reservationId 필수
✅ 중앙 시스템이 2차 검증 (Cache 구버전 대비)
✅ 응답의 idTagInfo로 Cache 업데이트
✅ 잘못된 데이터도 반드시 응답!

트랜잭션 ID:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
- 이후 모든 메시지에 포함
- MeterValues.req → transactionId: 12345
- StopTransaction.req → transactionId: 12345
- 요금 계산 및 추적의 핵심!

보안 포인트:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🔐 로컬 인증 성공 ≠ 최종 인증
   → 중앙 시스템이 마지막 관문!
   
🔐 Cache 신뢰 X
   → 항상 최신 상태 확인
   
🔐 차단된 카드도 로컬에서는 통과 가능
   → StartTransaction.conf에서 최종 판단
```

---

### 🔄 상태 전환

```
Available
    ↓ (카드 태그)
Preparing
    ↓ (플러그 연결, 인증 완료)
📢 StartTransaction.req 전송!
    ↓ (응답 받음)
Charging (StatusNotification.req)
    ↓
MeterValues.req 전송 시작 (60초마다)
```

---

## 📚 참고

- 원문: `/ocpp_16_analyze/04_Operations_Initiated_by_Charge_Point.md` - 4.8 Start Transaction
- 검수 완료: ✅ 2026-01-17

---

**다음 작성 예정**: 4.1 Authorize, 4.3 Data Transfer, ...

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
  "value": "2"
  // 단위: 초
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
shouldSendNotification(stateDuration)
{
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
  "status": "Charging",
  // 현재 상태
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

