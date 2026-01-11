# 1. Introduction (소개)

> 출처: ocpp-j-1.6-specification.pdf, Chapter 1
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

## 1.1. Purpose of this document (문서의 목적)

이 문서의 목적은 독자에게 올바른 상호 운용 가능한 OCPP JSON 구현(OCPP-J)을 생성하는 데 필요한 정보를 제공하는 것이다. 우리는 우리 자신의 경험을 바탕으로 무엇이 필수적인지, 무엇이 좋은 관행으로 간주되는지, 무엇을 해서는 안 되는지를 설명하고자 한다. 의심할 여지없이 오해나 모호성이 남아있겠지만, 이 문서를 통해 가능한 한 이를 방지하는 것을 목표로 한다.

## 1.2. Intended audience (대상 독자)

이 문서는 올바르고 상호 운용 가능한 방식으로 OCPP JSON을 이해하거나 구현하려는 개발자를 대상으로 한다. 서버 또는 임베디드 장치에서 웹 서비스를 구현하는 데 대한 기초적인 지식이 있다고 가정한다.

## 1.3. OCPP-S and OCPP-J

OCPP 1.6의 도입으로 OCPP에는 두 가지 다른 유형이 있다. SOAP 기반 구현 외에도 훨씬 더 컴팩트한 JSON 대안을 사용할 수 있는 가능성이 있다. 구현 유형에 대한 커뮤니케이션에서 혼동을 피하기 위해 JSON 또는 SOAP을 나타내기 위해 고유한 접미사 -J 및 -S를 사용하는 것을 권장한다. 일반적인 용어로 JSON의 경우 OCPP-J, SOAP의 경우 OCPP-S가 된다.

버전별 용어는 OCPP1.6J 또는 OCPP1.2S가 된다. OCPP 1.2 또는 1.5에 대해 접미사가 지정되지 않은 경우 SOAP 구현으로 가정해야 한다. 릴리스 1.6부터는 더 이상 암묵적일 수 없으며 항상 명확하게 해야 한다. 시스템이 JSON 및 SOAP 변형을 모두 지원하는 경우 단순히 OCPP1.6이 아닌 OCPP1.6JS로 표시하는 것이 좋은 관행으로 간주된다.

이 문서는 OCPP-J를 설명하며, OCPP-S에 대해서는 다음을 참조: [OCPP_IMP_S]

## 1.4. Conventions (규약)

이 문서의 키워드 "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", "OPTIONAL"은 [RFC2119]에 설명된 대로 해석되어야 한다.

## 1.5. Definitions & Abbreviations (정의 및 약어)

| 용어 | 설명 |
|------|------|
| **IANA** | Internet Assigned Numbers Authority ([www.iana.org](http://www.iana.org)). |
| **OCPP-J** | OCPP communication over WebSocket using JSON. 특정 OCPP 버전은 J 확장자로 표시되어야 한다. OCPP1.5J는 1.5의 JSON/WebSocket 구현을 의미한다. |
| **OCPP-S** | OCPP communication over SOAP and HTTP(S). 버전 1.6부터는 명시적으로 언급되어야 한다. 이전 버전은 명확하게 지정되지 않는 한 S로 가정된다. 예: OCPP1.5는 OCPP1.5S와 동일하다. |
| **RPC** | Remote procedure call |
| **WAMP** | WAMP는 비동기 데이터를 처리하기 위한 메시징 패턴을 제공하는 오픈 WebSocket 하위 프로토콜이다. |

## 1.6. References (참조)

| 참조 | 설명 |
|------|------|
| **[JSON]** | http://www.json.org/ |
| **[OCPP_IMP_S]** | OCPP SOAP implementation specification |
| **[RFC2119]** | "Key words for use in RFCs to Indicate Requirement Levels". S. Bradner. March 1997.<br>http://www.ietf.org/rfc/rfc2119.txt |
| **[RFC2616]** | "Hypertext Transfer Protocol — HTTP/1.1".<br>http://tools.ietf.org/html/rfc2616 |
| **[RFC2617]** | "HTTP Authentication: Basic and Digest Access Authentication".<br>http://tools.ietf.org/html/rfc2617 |
| **[RFC3629]** | "UTF-8, a transformation format of ISO 10646".<br>http://tools.ietf.org/html/rfc3629 |
| **[RFC3986]** | "Uniform Resource Identifier (URI): Generic Syntax".<br>http://tools.ietf.org/html/rfc3986 |
| **[RFC5246]** | "The Transport Layer Security (TLS) Protocol; Version 1.2".<br>http://tools.ietf.org/html/rfc5246 |
| **[RFC6455]** | "The WebSocket Protocol".<br>http://tools.ietf.org/html/rfc6455 |
| **[WAMP]** | http://wamp.ws/ |
| **[WIKIWS]** | http://en.wikipedia.org/wiki/WebSocket |
| **[WS]** | http://www.websocket.org/ |
