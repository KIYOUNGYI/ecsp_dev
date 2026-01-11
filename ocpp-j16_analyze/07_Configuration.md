# 7. Configuration (구성)

> 출처: ocpp-j-1.6-specification.pdf, Chapter 7
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

JSON/WebSocket 동작을 제어하기 위해 OCPP Get/ChangeConfiguration 메시지에 다음 항목이 추가된다:

## Table 8: Additional OCPP Keys (추가 OCPP 키)

| Key | Value |
|-----|-------|
| WebSocketPingInterval | integer 값 0은 클라이언트 측 WebSocket Ping/Pong을 비활성화한다. 이 경우 Ping/Pong이 전혀 없거나 서버가 Ping을 시작하고 클라이언트가 Pong으로 응답한다. 양수 값은 Ping 간의 초 수로 해석된다. 음수 값은 허용되지 않는다.<br><br>ChangeConfiguration은 REJECTED 결과를 반환할 것으로 예상된다. |
