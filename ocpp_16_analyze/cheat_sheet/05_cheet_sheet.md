# OCPP 1.6 - Chapter 5 Cheat Sheet
## Operations Initiated by Central System

> **목적**: 중앙 시스템에서 시작하는 주요 OCPP 메시지들의 핵심 동작 이해  
> **작성일**: 2026-01-19

---

## 📋 목차

1. [5.11 Remote Start Transaction (원격 트랜잭션 시작)](#511-remote-start-transaction-원격-트랜잭션-시작)
2. [5.12 Remote Stop Transaction (원격 트랜잭션 중지)](#512-remote-stop-transaction-원격-트랜잭션-중지)

---

## 🚀 5.11 Remote Start Transaction (원격 트랜잭션 시작)

### 📌 핵심 개념

**Remote Start Transaction**은 중앙 시스템이 충전기에게 "이 사용자를 위해 충전 시작해줘!"라고 원격으로 명령하는 메시지입니다.

### 🧒 어린이용 설명 (쉬운 비유)

```
🎮 게임 리모컨으로 장난감 자동차 조종하기

📱 엄마가 스마트폰 앱으로 충전 시작 버튼 클릭!
    "충전기야! USER001 카드로 충전 시작해줘!"
    ↓
🔌 충전기 (장난감 자동차)
    "알겠습니다! 명령 받았어요!"
    ↓
    
    🔍 AuthorizeRemoteTxRequests = true?
    
    ┌─ ✅ true (안전 모드)
    │   "먼저 카드가 진짜인지 확인해볼게요!"
    │   1. 로컬 목록 확인 (집에 있는 명단)
    └─ ❌ false (빠른 모드)
        "일단 바로 충전 시작할게요!"
        → 충전 시작! ⚡
        → StartTransaction 보내면서 중앙이 나중에 확인
```

**실생활 비유:**

1. **AuthorizeRemoteTxRequests = true (은행 ATM)**
   ```
   📱 엄마: "철수야, 용돈 찾아가라"
   👦 철수: ATM 가서 카드 넣음
   🏦 ATM: "카드 확인 중... 비밀번호 입력하세요"
   ✅ 인증 완료 후 돈 인출
   ```

2. **AuthorizeRemoteTxRequests = false (집 냉장고)**
   ```
   📱 엄마: "철수야, 냉장고에서 우유 꺼내 마셔"
   👦 철수: 바로 냉장고 열고 우유 마심
   ✅ 나중에 엄마가 확인 "철수가 우유 마셨네"
   ```

---

### 📋 메시지 구조 (JSON 예시)


```json
{
  "idTag": "USER_CARD_001",
  "connectorId": 1,  // 선택사항: 특정 커넥터 지정
  "chargingProfile": {  // 선택사항: 충전 프로파일
    "chargingProfileId": 1,
    "stackLevel": 0,
    "chargingProfilePurpose": "TxProfile",
    "chargingProfileKind": "Relative",
    "chargingSchedule": {
      "chargingRateUnit": "W",
      "chargingSchedulePeriod": [
        {
          "startPeriod": 0,
          "limit": 7200  // 7.2kW
        }
      ]
    }
  }
}
```

#### 응답 (RemoteStartTransaction.conf)

```json
{
  "status": "Accepted"  // or "Rejected"
}
```

**Status 값:**
- **Accepted**: ✅ 충전 시작 시도함
- **Rejected**: ❌ 거부 (커넥터 사용 중, 고장 등)

---

### 🎭 역할별 책임 매트릭스

#### 📋 누가 무엇을 해야 하나?

| 단계 | 중앙 시스템 (CSMS) 👩‍💻 | 충전기 (CP) 👨‍💻 |
|------|----------------------|----------------|
| **1️⃣ 원격 시작 명령** | ✅ RemoteStartTransaction.req 전송<br>```json<br>{<br>  "idTag": "USER001",<br>  "connectorId": 1<br>}<br>``` | - |
| **2️⃣ 요청 수신** | - | ✅ 요청 수신<br>✅ connectorId 유효성 검증<br>✅ 커넥터 상태 확인 |
| **3️⃣ 즉시 응답** | - | ✅ RemoteStartTransaction.conf 전송<br>```json<br>{<br>  "status": "Accepted"<br>}<br>``` |
| **4️⃣ 인증 처리<br>(AuthorizeRemoteTxRequests=true)** | ✅ Authorize.req 수신 시 검증<br>✅ idTag 유효성 확인<br>✅ Authorize.conf 응답 | ✅ Local List 확인<br>✅ Authorization Cache 확인<br>✅ 없으면 Authorize.req 전송<br>✅ 인증 완료 대기 |
| **5️⃣ 즉시 시작<br>(AuthorizeRemoteTxRequests=false)** | - | ✅ 인증 없이 즉시 시작<br>✅ StartTransaction.req 전송 |
| **6️⃣ 트랜잭션 시작** | ✅ StartTransaction.req 수신<br>✅ **이때 idTag 최종 검증!**<br>✅ transactionId 생성<br>✅ StartTransaction.conf 응답 | ✅ StartTransaction.req 전송<br>```json<br>{<br>  "connectorId": 1,<br>  "idTag": "USER001",<br>  "meterStart": 5000,<br>  "timestamp": "..."<br>}<br>``` |
| **7️⃣ 상태 알림** | - | ✅ StatusNotification.req<br>(status: Charging) |
| **8️⃣ 미터값 전송** | - | ✅ MeterValues.req 주기적 전송 |

---

### 🔐 AuthorizeRemoteTxRequests 설정별 동작

#### ✅ true (안전 모드) - 로컬 인증 먼저

| 단계 | 동작 | 설명 |
|------|------|------|
| 1 | 📨 RemoteStartTransaction.req 수신 | idTag: "USER001" |
| 2 | 🔍 Local Authorization List 확인 | "USER001"이 목록에 있나? |
| 3 | 🔍 Authorization Cache 확인 | 최근에 본 적 있나? |
| 4 | 📤 Authorize.req 전송 (없으면) | CS에 물어보기 |
| 5 | 📥 Authorize.conf 수신 | status: Accepted/Blocked |
| 6 | ✅ 인증 성공 시 시작 | StartTransaction.req 전송 |
| 7 | ❌ 인증 실패 시 중단 | 충전 시작 안 함 |

#### ❌ false (빠른 모드) - 일단 시작

| 단계 | 동작 | 설명 |
|------|------|------|
| 1 | 📨 RemoteStartTransaction.req 수신 | idTag: "USER001" |
| 2 | ⚡ 즉시 충전 시작 | 인증 검사 없이 바로 시작 |
| 3 | 📤 StartTransaction.req 전송 | CS에 알림 |
| 4 | 🔍 CS에서 idTag 검증 | **이때 최종 검증!** |
| 5 | 📥 StartTransaction.conf 수신 | status: Accepted/Blocked |
| 6 | ❌ Blocked이면? | 즉시 StopTransaction.req<br>reason: "DeAuthorized" |

---

### 💡 핵심 체크리스트

#### 중앙 시스템 (CSMS) 개발자 체크리스트

- [ ] **요청 전 검증**
  - [ ] idTag가 유효한 형식인가?
  - [ ] connectorId 지정 시 해당 충전기에 존재하는 커넥터인가?
  - [ ] 충전기가 온라인 상태인가?

- [ ] **ChargingProfile 포함 시**
  - [ ] chargingProfilePurpose = "TxProfile"로 설정했는가?
  - [ ] transactionId는 **설정하지 않았는가?** (SHALL NOT)
  - [ ] 충전기가 Smart Charging을 지원하는가?

  - [ ] status: Rejected 받으면 사용자에게 알림
  - [ ] status: Accepted 받으면 StartTransaction.req 대기
  - [ ] 타임아웃 설정 (예: 30초 내 StartTransaction 안 오면?)

- [ ] **StartTransaction.req 수신 시**
  - [ ] **idTag 최종 검증 수행!** (DB에서 재확인)
  - [ ] 차단/만료 여부 체크
  - [ ] transactionId 생성 및 저장

#### 충전기 (CP) 개발자 체크리스트

- [ ] **요청 수신 시**
  - [ ] connectorId 유효성 검증
  - [ ] 커넥터 상태 확인 (Available? Reserved? Faulted?)
  - [ ] 차량 연결 여부 확인

- [ ] **connectorId 없는 요청**
  - [ ] 자동 커넥터 선택 로직 구현
  - [ ] 또는 Rejected 응답 (구현 선택)

- [ ] **AuthorizeRemoteTxRequests = true**
  - [ ] Local List 확인 구현
  - [ ] Authorization Cache 확인 구현
  - [ ] Authorize.req 전송 로직
  - [ ] 인증 실패 시 충전 시작 안 함

- [ ] **AuthorizeRemoteTxRequests = false**
  - [ ] 즉시 충전 시작 로직
  - [ ] StartTransaction.req 즉시 전송
  - [ ] StartTransaction.conf에서 Blocked 받으면 즉시 중단

- [ ] **ChargingProfile 수신 시**
  - [ ] Smart Charging 지원 여부 확인
  - [ ] 미지원 시 무시 (SHOULD ignore)
  - [ ] 지원 시 프로파일 적용

- [ ] **오류 처리**
  - [ ] 차량 연결 안 됨 → status: Rejected
  - [ ] 커넥터 Faulted → status: Rejected
  - [ ] 커넥터 Occupied → status: Rejected

---

### 🎬 실전 시나리오

#### 시나리오 1: 모바일 앱으로 충전 시작 (안전 모드)

```typescript
=== 설정 ===
AuthorizeRemoteTxRequests = true

=== 흐름 ===
14:00:00 → 📱 사용자가 모바일 앱에서 "충전 시작" 버튼 클릭
           idTag: "MOBILE_USER_001"

14:00:01 → 🏢 CSMS: RemoteStartTransaction.req 전송
           {
             "idTag": "MOBILE_USER_001",
             "connectorId": 1
           }

14:00:02 → 🔌 CP: 요청 수신
           → connectorId 1 확인: ✅ Available
           → 차량 연결 확인: ✅ 연결됨
           
14:00:03 → 🔌 CP: RemoteStartTransaction.conf
           {
             "status": "Accepted"
           }

14:00:04 → 🔌 CP: 인증 시작 (AuthorizeRemoteTxRequests=true)
           → Local List 확인: ❌ 없음
           → Authorization Cache 확인: ❌ 없음
           → Authorize.req 전송

14:00:05 → 🏢 CSMS: Authorize.req 수신
           → DB 확인: ✅ 유효한 사용자
           → Authorize.conf
           {
             "idTagInfo": {
               "status": "Accepted",
               "expiryDate": "2027-12-31T23:59:59Z"
             }
           }

14:00:06 → 🔌 CP: 인증 성공!
           → StartTransaction.req 전송
           {
             "connectorId": 1,
             "idTag": "MOBILE_USER_001",
             "meterStart": 5000,
             "timestamp": "2026-01-19T14:00:06Z"
           }

14:00:07 → 🏢 CSMS: StartTransaction.req 수신
           → idTag 최종 검증 (DB 재확인)
           → ✅ 유효
           → transactionId 생성: 12345
           → StartTransaction.conf
           {
             "transactionId": 12345,
             "idTagInfo": {
               "status": "Accepted"
             }
           }

14:00:08 → 🔌 CP: 충전 시작! ⚡
           → StatusNotification.req (status: Charging)

14:00:09 → 📱 사용자 앱: "충전이 시작되었습니다!" 알림
```

---

#### 시나리오 2: 헬프데스크 원격 지원 (빠른 모드)

```typescript
=== 설정 ===
AuthorizeRemoteTxRequests = false

=== 상황 ===
고객: "충전이 안 시작돼요! 😢"
헬프데스크: "제가 원격으로 시작해드릴게요!"

=== 흐름 ===
15:30:00 → 👨‍💼 헬프데스크 운영자: 원격 시작 명령
           idTag: "HELP_DESK_OVERRIDE"

15:30:01 → 🏢 CSMS: RemoteStartTransaction.req
           {
             "idTag": "HELP_DESK_OVERRIDE",
             "connectorId": 2
           }

15:30:02 → 🔌 CP: 요청 수신
           → connectorId 2 확인: ✅ Available
           → RemoteStartTransaction.conf
           {
             "status": "Accepted"
           }

15:30:03 → 🔌 CP: 즉시 충전 시작! (인증 없이)
           → StartTransaction.req 전송
           {
             "connectorId": 2,
             "idTag": "HELP_DESK_OVERRIDE",
             "meterStart": 3200,
             "timestamp": "2026-01-19T15:30:03Z"
           }

15:30:04 → 🏢 CSMS: StartTransaction.req 수신
           → **이때 idTag 최종 검증!**
           → DB 확인: ✅ 헬프데스크 권한 유효
           → transactionId 생성: 12346
           → StartTransaction.conf
           {
             "transactionId": 12346,
             "idTagInfo": {
               "status": "Accepted"
             }
           }

15:30:05 → 🔌 CP: ✅ 충전 계속 진행
           → StatusNotification.req (status: Charging)

15:30:06 → 👨‍💼 헬프데스크: "충전 시작되었습니다!"
           📞 고객: "감사합니다! 😊"
```

---

#### 시나리오 3: 차단된 사용자 (빠른 모드 → 중단)

```typescript
=== 설정 ===
AuthorizeRemoteTxRequests = false

=== 상황 ===
사용자가 모바일 앱으로 충전 시작
하지만... 카드가 3일 전에 차단됨! (도난 신고)

=== 흐름 ===
16:00:00 → 📱 사용자: 앱에서 "충전 시작"
           idTag: "STOLEN_CARD_999"

16:00:01 → 🏢 CSMS: RemoteStartTransaction.req
           {
             "idTag": "STOLEN_CARD_999"
           }

16:00:02 → 🔌 CP: RemoteStartTransaction.conf
           {
             "status": "Accepted"
           }

16:00:03 → 🔌 CP: 즉시 충전 시작! (인증 안 함)
           → StartTransaction.req
           {
             "idTag": "STOLEN_CARD_999",
             "meterStart": 1000
           }

16:00:04 → 🏢 CSMS: StartTransaction.req 수신
           → DB 확인: 🚨 차단된 카드!
           → StartTransaction.conf
           {
             "transactionId": 12347,
             "idTagInfo": {
               "status": "Blocked"  // ❌ 차단!
             }
           }

16:00:05 → 🔌 CP: Blocked 응답 수신
           → 🚨 즉시 충전 중단!
           → StopTransaction.req
           {
             "transactionId": 12347,
             "meterStop": 1000,  // 거의 충전 안 됨
             "reason": "DeAuthorized"
           }

16:00:06 → 🔌 CP: StatusNotification.req
           (status: Available)

16:00:07 → 📱 사용자 앱: "충전이 거부되었습니다"
           🚨 CSMS: 보안 알림 발송
```

---

#### 시나리오 4: ChargingProfile 포함 (Smart Charging)

```typescript
=== 상황 ===
전력 부하 분산이 필요한 충전소
최대 7.2kW로 제한하여 충전 시작

=== 흐름 ===
17:00:00 → 🏢 CSMS: RemoteStartTransaction.req
           {
             "idTag": "USER_CARD_002",
             "connectorId": 1,
             "chargingProfile": {
               "chargingProfileId": 100,
               "stackLevel": 0,
               "chargingProfilePurpose": "TxProfile",  // ✅ 필수!
               "chargingProfileKind": "Relative",
               "chargingSchedule": {
                 "chargingRateUnit": "W",
                 "chargingSchedulePeriod": [
                   {
                     "startPeriod": 0,
                     "limit": 7200  // 7.2kW 제한
                   }
                 ]
               }
             }
           }

17:00:01 → 🔌 CP: Smart Charging 지원 확인
           → ✅ 지원함
           → 프로파일 저장

17:00:02 → 🔌 CP: RemoteStartTransaction.conf
           {
             "status": "Accepted"
           }

17:00:03 → 🔌 CP: 인증 및 충전 시작
           → StartTransaction.req 전송

17:00:04 → 🏢 CSMS: StartTransaction.conf
           {
             "transactionId": 12348
           }

17:00:05 → 🔌 CP: 충전 시작!
           → **최대 7.2kW로 제한 적용** ✅
           → StatusNotification.req (Charging)

17:05:00 → 🔌 CP: MeterValues.req
           {
             "transactionId": 12348,
             "meterValue": [{
               "sampledValue": [{
                 "value": "7200",  // 7.2kW (제한 준수)
                 "measurand": "Power.Active.Import",
                 "unit": "W"
               }]
             }]
           }
```

---

#### 시나리오 5: 커넥터 자동 선택

```typescript
=== 상황 ===
connectorId 지정 없이 요청
충전기가 자동으로 빈 커넥터 선택

=== 충전기 상태 ===
Connector 1: Charging (사용 중)
Connector 2: Available (사용 가능)
Connector 3: Faulted (고장)

=== 흐름 ===
18:00:00 → 🏢 CSMS: RemoteStartTransaction.req
           {
             "idTag": "USER_CARD_003"
             // connectorId 없음!
           }

18:00:01 → 🔌 CP: 자동 선택 로직
           → Connector 1: ❌ Charging (사용 중)
           → Connector 2: ✅ Available (선택!)
           → Connector 3: ❌ Faulted (고장)

18:00:02 → 🔌 CP: RemoteStartTransaction.conf
           {
             "status": "Accepted"
           }

18:00:03 → 🔌 CP: Connector 2에서 충전 시작
           → StartTransaction.req
           {
             "connectorId": 2,  // 자동 선택됨
             "idTag": "USER_CARD_003",
             "meterStart": 2500
           }

18:00:04 → 🏢 CSMS: "Connector 2에서 시작되었네요!"
```

---

## 🛑 5.12 Remote Stop Transaction (원격 트랜잭션 중지)

### 📌 핵심 개념

**Remote Stop Transaction**은 중앙 시스템이 "지금 충전 멈춰!"라고 충전기에 원격으로 명령하는 메시지입니다. 충전기가 스스로 멈추는 것(로컬 중지)과 똑같이 동작합니다.

### 🧒 어린이용 설명 (쉬운 비유)

```
📱 엄마가 스마트폰 앱에서 "충전 멈춰!" 버튼을 누름
    ↓
🏢 중앙 시스템 (엄마)
    "충전기야! 123번 충전 중인 거 멈춰줘!"
    ↓
🔌 충전기 (장난감 자동차)
    "네! 지금 멈출게요!"
    ↓
🚗 충전 중이던 자동차: 충전이 멈춤
🔓 (필요하면) 케이블 잠금도 해제!
```

**실생활 비유:**

- 엄마가 놀이터에서 노는 아이에게 "이제 집에 가자!"라고 말하면, 아이가 바로 집에 가는 것과 비슷해요.
- 충전기가 스스로 멈추는 것(로컬 버튼)과, 엄마(중앙 시스템)가 멀리서 "멈춰!"라고 하는 것(원격 중지)은 결과가 똑같아요.

### 📋 메시지 흐름 (쉬운 순서도)

1. 중앙 시스템이 충전기에게 RemoteStopTransaction.req(트랜잭션 번호 포함) 전송
2. 충전기는 해당 트랜잭션이 진행 중이면 "Accepted"로 응답
3. 충전기는 즉시 충전 중지 → StopTransaction.req 전송
4. (필요하면) 커넥터 잠금 해제
5. 중앙 시스템은 StopTransaction.req를 받고 과금/정산 처리

### 💡 한눈에 보는 포인트

- 원격 중지는 로컬 중지와 완전히 동일하게 동작 (OCPP 스펙)
- 충전기가 반드시 StopTransaction.req를 보내야 함 (SHALL)
- 커넥터 잠금 해제도 필요시 자동 처리
- 주로 앱/헬프데스크에서 "충전 멈춰!" 요청에 사용

### 📝 예시 메시지 (JSON)

#### 요청 (RemoteStopTransaction.req)
```json
{
  "transactionId": 12345
}
```

#### 응답 (RemoteStopTransaction.conf)
```json
{
  "status": "Accepted"
}
```

#### 충전기 → 중앙 시스템 (StopTransaction.req)
```json
{
  "transactionId": 12345,
  "meterStop": 15750,
  "timestamp": "2026-01-19T15:00:00Z",
  "reason": "Remote"
}
```

---

**작성일**: 2026-01-19  
**버전**: 1.0  
**상태**: ✅ 5.11 Complete
