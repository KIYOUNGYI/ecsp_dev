# 2. Terminology and Conventions (용어 및 표기법)

> 출처: ocpp-1.6 edition 2.pdf, Chapter 2 (p.10-12)
> 작업 유형: 완전 번역 (Full Translation) - 스크린샷 기반

## 2.1. Conventions (표기법)

본 명세서에서 사용되는 키워드 "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", "
OPTIONAL"은 RFC 2119 [RFC2119]에 설명된 대로 해석되어야 하며, 다음의 추가 명확화 조항이 적용된다:

"SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED" 용어의 사용과 관련하여 "특정 상황에서의 타당한 이유(valid reasons in particular
circumstances)"라는 문구는 기술적으로 타당한 이유를 의미하는 것으로 해석되어야 한다. 예를 들어, 필요한 기능의 부재, 충전 설계 목적을 위한 이 명세의 일부가 특정적으로 상업적 또는 기타 비기술적
근거에 기반한 결정을 배제하는 경우(예: 구현 비용 또는 사용 가능성) 등이 있다.

"Scope" 및 "Terminology and Conventions"를 제외한 모든 섹션과 부록은 명시적으로 정보 제공용이라고 표시되지 않는 한 규범적이다.

## 2.2. Definitions (정의)

이 섹션에는 본 문서 전반에 걸쳐 사용되는 용어가 포함되어 있다.

| 용어                              | 정의                                                                                                                                                                                        |
|---------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Central System**              | Charge Point Management System: 충전기를 관리하고 사용자가 충전기를 사용하도록 인증하기 위한 정보를 보유한 중앙 시스템.                                                                                                         |
| **CiString**                    | Case Insensitive String. ASCII 문자만 허용됨.  (ex> Hello = hello = HELLO)                                                                                                                      |
| **Charge Point**                | 전기차를 충전할 수 있는 물리적 시스템. 충전기는 하나 이상의 커넥터를 가질 수 있다.                                                                                                                                          |
| **Charging Profile**            | 다양한 유형의 프로파일에 사용되는 일반 충전 프로파일. 프로파일에 대한 정보와 충전 스케줄을 보유한다. 미래 버전의 OCPP는 1개 이상의 충전 스케줄을 보유할 수 있다.                                                                                           |
| **Charging Schedule**           | 충전 프로파일의 일부. 충전 전력 또는 전류 한계의 블록을 정의한다. 시작 시간과 기간을 포함할 수 있다.                                                                                                                               |
| **Charging Session**            | 충전 세션은 사용자 또는 EV와의 첫 번째 상호작용이 발생할 때 시작된다. 이것은 카드 스와이프, 트랜잭션의 원격 시작, 케이블 및/또는 EV의 연결, 주차 구역 점유 감지기 등이 될 수 있다.                                                                              |
| **Composite Charging Schedule** | 충전기에 의해 계산된 충전 스케줄. 충전소에 존재하는 모든 활성 스케줄과 가능한 로컬 한계의 계산 결과이다.                                                                                                                              |
| **Connector**                   | 이 명세서에서 사용되는 "Connector"라는 용어는 충전기(Charge Point)의 독립적으로 운영되고 관리되는 전기 콘센트를 의미한다. 일반적으로 이것은 단일 물리적 커넥터에 해당하지만, 경우에 따라 단일 콘센트가 다양한 차량 유형(예: 4륜 EV 및 전기 스쿠터)을 지원하기 위해 여러 물리적 소켓 유형 및/또는 일체형 케이블/커넥터 조합을 가질 수 있다. |
| **Control Pilot signal**        | IEC 61851-1에 정의된 충전기가 충전 전력 또는 전류를 알리기 위해 사용하는 신호.                                                                                                                                        |
| **Energy Offer Period**         | 에너지 제공 기간은 EVSE가 에너지 공급 준비가 되어 있고 기꺼이 공급할 때 시작된다.                                                                                                                                         |
| **Energy Offer SuspendPeriod**  | 트랜잭션 중에 에너지 제공이 EV에 의해 일시 중단될 수 있는 기간이 있을 수 있다. 예를 들어 스마트 충전 또는 부하 평준화로 인해.                                                                                                               |
| **Energy Transfer Period**      | EV가 에너지를 오프로드하거나 반환하기로 선택하는 시간. 여러 에너지 전송 기간이 트랜잭션 중에 가능하다.                                                                                                                               |
| **Local Controller**            | 로컬 컨트롤러는 여러 충전기를 관리할 수 있는 장치이다. 충전기와 중앙 시스템 사이에 연결될 수 있다. 로컬 컨트롤러를 이해하고 OCPP 메시지를 말한다. 전력 또는 전류를 중앙 시스템 또는 다른 OCPP 스마트 충전 메시지를 사용하여 더 정교한 방법으로 관리할 수 있다.                                  |
| **OCPP-J**                      | OCPP via JSON over WebSocket                                                                                                                                                              |
| **OCPP-S**                      | OCPP via SOAP                                                                                                                                                                             |
| **Phase Rotation**              | 전기 미터 또는 EV(없는 경우), 그리드 연결 및 EVSE 사이의 위상 배선 순서를 정의한다.                                                                                                                                     |
| **Transaction**                 | 모든 관련 사전 조건(예: 인증, 플러그 삽입)이 충족되고 충전기가 물리적으로 상태를 저장할 때 종료되는 충전 프로세스의 일부.                                                                                                                   |
| **String**                      | Case Sensitive String. ASCII 문자만 허용됨. 메시지와 열거형의 모든 문자열은 명시적으로 달리 언급되지 않는 한 대소문자를 구분한다.                                                                                                    |

## 2.3. Abbreviations (약어)

| 약어          | 설명                                                                           |
|-------------|------------------------------------------------------------------------------|
| **CSL**     | Comma Separated List                                                         |
| **CPO**     | Charge Point Operator                                                        |
| **DNS**     | Domain Name System                                                           |
| **DST**     | Daylight Saving Time                                                         |
| **EV**      | Electrical Vehicle (this can be BEV (Battery EV) or PHEV (plugin Hybrid EV)) |
| **EVSE**    | Electric Vehicle Supply Equipment (IEC61851-1)                               |
| **FTP(S)**  | File Transport Protocol (Secure)                                             |
| **HTTP(S)** | HyperText Transport Protocol (Secure)                                        |
| **ICCID**   | Integrated Circuit Card Identifier                                           |
| **IMSI**    | International Mobile Subscription Identity                                   |
| **JSON**    | JavaScript Object Notation                                                   |
| **NAT**     | Native Address Translation                                                   |
| **PDU**     | Protocol Data Unit                                                           |
| **SC**      | Smart Charging                                                               |
| **SOAP**    | Simple Object Access Protocol                                                |
| **URL**     | Uniform Resource Locator                                                     |
| **RST**     | 3 phase power connection, Standard Reference Phasing                         |
| **RTS**     | 3 phase power connection, Reversed Reference Phasing                         |
| **SRT**     | 3 phase power connection, Reversed 240 degree rotation                       |
| **STR**     | 3 phase power connection, Standard 120 degree rotation                       |
| **TRS**     | 3 phase power connection, Standard 240 degree rotation                       |
| **TSR**     | 3 phase power connection, Reversed 120 degree rotation                       |
| **UTC**     | Coordinated Universal Time                                                   |

## 2.4. References (참조)

| 참조               | 설명                                                                                                                                                               |
|------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **[IEC61851-1]** | "IEC 61851-1 2010: Electric vehicle conductive charging system - Part 1: General requirements" https://webstore.iec.ch/p/preview/info_iec61851-1%7Bed2.0%7Db.pdf |
| **[OCPP1.5]**    | "OCPP 1.5: Open Charge Point Protocol 1.5" http://www.openchargealliance.org/downloads/                                                                          |
| **[OCPP-1.6CT]** | "OCPP 1.6 Compliance testing" http://www.openchargealliance.org/downloads/                                                                                       |
| **[OCPP-IMP_J]** | "OCPP JSON Specification" http://www.openchargealliance.org/downloads/                                                                                           |
| **[OCPP-IMP_S]** | "OCPP SOAP Specification" http://www.openchargealliance.org/downloads/                                                                                           |
| **[RFC2119]**    | "Key words for use in RFCs to Indicate Requirement Levels" S. Bradner. March 1997. http://www.ietf.org/rfc/rfc2119.txt                                           |
