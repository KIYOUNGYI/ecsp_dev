# 2. Benefits & Issues (이점 및 문제점)

> 출처: ocpp-j-1.6-specification.pdf, Chapter 2
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

WebSocket 프로토콜은 [RFC6455]에 정의되어 있다. 초기 드래프트 WebSocket 명세의 작동 구현이 존재하지만, OCPP-J 구현은 [RFC6455]에 설명된 프로토콜을 사용해야 한다(SHOULD).

WebSocket은 TCP 위에 자체 메시지 구조를 정의한다는 점을 유의해야 한다. WebSocket을 통해 전송되는 데이터는 TCP 수준에서 헤더가 있는 WebSocket 프레임으로 래핑된다. 프레임워크를 사용할 때 이것은 완전히 투명하다. 그러나 임베디드 시스템에서 작업할 때는 WebSocket 라이브러리를 사용할 수 없을 수 있으며, 이 경우 [RFC6455]에 따라 메시지를 올바르게 프레임화해야 한다.

