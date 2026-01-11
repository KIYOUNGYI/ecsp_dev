# 8. Firmware and Diagnostics File Transfer (펌웨어 및 진단 파일 전송)

> 출처: ocpp-1.6 edition 2.pdf, Chapter 8
> 작업 유형: 완전 번역 (Full Translation) - 텍스트 복붙 기반

이 섹션은 규범적이다.

지원되는 전송 프로토콜은 구성 키 SupportedFileTransferProtocols에 의해 제어된다. FTP, FTPS, HTTP, HTTPS (CSL)

## 8.1. Download Firmware (펌웨어 다운로드)

충전기가 새 펌웨어에 대한 알림을 받으면 이 펌웨어를 다운로드할 수 있어야 한다. 중앙 시스템은 요청에서 펌웨어를 다운로드할 수 있는 URL을 제공한다. URL에는 펌웨어를 다운로드하는 데 사용해야 하는 프로토콜도 포함되어 있다.

펌웨어는 FTP 또는 FTPS를 통해 다운로드하는 것이 권장된다. FTP(S)는 HTTP보다 대용량 이진 데이터에 더 최적화되어 있다. 또한 FTP(S)에는 다운로드를 재개할 수 있는 기능이 있다. 다운로드가 중단된 경우 충전기는 이미 다운로드한 부분 이후부터 다운로드를 재개할 수 있다. FTP URL은 다음 형식이다: `ftp://user:password@host:port/path` 여기서 `user:password@`, `:password` 또는 `:port` 부분은 제외될 수 있다.

올바른 펌웨어가 다운로드되도록 하기 위해 펌웨어도 디지털 서명되는 것이 권장된다(RECOMMENDED).

## 8.2. Upload Diagnostics (진단 업로드)

충전기가 진단 파일을 업로드하도록 요청받으면 중앙 시스템은 요청에서 충전기가 파일을 업로드해야 하는 URL을 제공한다. URL에는 파일을 업로드하는 데 사용해야 하는 프로토콜도 포함되어 있다.

진단 파일은 FTP 또는 FTPS를 통해 다운로드하는 것이 권장된다. FTP(S)는 HTTP보다 대용량 이진 데이터에 더 최적화되어 있다. 또한 FTP(S)에는 업로드를 재개할 수 있는 기능이 있다. 업로드가 중단된 경우 충전기는 이미 업로드한 부분 이후부터 업로드를 재개할 수 있다. FTP URL은 다음 형식이다: `ftp://user:password@host:port/path` 여기서 `user:password@`, `:password` 또는 `:port` 부분은 제외될 수 있다.
