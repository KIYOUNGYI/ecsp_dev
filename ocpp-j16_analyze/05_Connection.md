# 5. Connection (연결)

> 출처: ocpp-j-1.6-specification.pdf, Chapter 5
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

## 5.1. Compression (압축)

JSON은 매우 컴팩트하므로 WebSocket [RFC6455] 명세의 일부로 허용되는 형태 이외의 압축을 사용하지 않는 것이 권장된다. 그렇지 않으면 상호 운용성이 손상될 수 있다.

## 5.2. Data integrity (데이터 무결성)

데이터 무결성을 위해 기본 TCP/IP 전송 계층 메커니즘에 의존한다.

## 5.3. WebSocket Ping in relation to OCPP Heartbeat (OCPP Heartbeat와 관련된 WebSocket Ping)

WebSocket 명세는 원격 엔드포인트가 여전히 응답하는지 확인하는 데 사용되는 Ping 및 Pong 프레임을 정의한다. 실제로 이 메커니즘은 네트워크 운영자가 일정 기간 비활성 후 기본 네트워크 연결을 조용히 닫는 것을 방지하는 데도 사용된다. 이 WebSocket 기능은 대부분의 OCPP Heartbeat 메시지를 대체할 수 있지만 모든 기능을 대체할 수는 없다.

Heartbeat 응답의 중요한 측면은 시간 동기화이다. Ping 및 Pong 프레임은 이를 위해 사용될 수 없으므로 충전기에서 올바른 시계 설정을 보장하기 위해 하루에 최소 하나의 원래 Heartbeat 메시지를 보내는 것이 권장된다.

## 5.4. Reconnecting (재연결)

재연결할 때 충전기는 마지막 연결 이후 BootNotification의 요소 중 하나 이상이 변경되지 않은 한 BootNotification을 보내서는 안 된다. 이전 SOAP 기반 솔루션의 경우 이것이 좋은 관행으로 간주되었지만 WebSocket을 사용할 때 서버는 연결이 설정되는 순간 이미 ID와 통신 채널 간의 일치를 만들 수 있다. 추가 메시지가 필요하지 않다.

## 5.5. Network node hierarchy (네트워크 노드 계층)

물리적 네트워크 토폴로지는 JSON 또는 SOAP 선택의 영향을 받지 않는다. 그러나 JSON의 경우 충전기가 중앙 시스템에 대한 TCP 연결을 열고 중앙 시스템이 시작한 통신을 위해 이 연결을 열어 두도록 함으로써 NAT(Network Address Translation) 문제가 해결되었다. 따라서 중앙 시스템과 충전기 사이에 SOAP 호출을 해석하고 리디렉션할 수 있는 스마트 장치를 가질 필요가 더 이상 없다.
