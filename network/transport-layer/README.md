# 전송 계층(Transport Layer)
1. 물리 계층(Physical Layer)
   - 0과 1의 비트 정보를 회선에 보내기 위한 전기적 신호 변환
   - 프로토콜: RS-232C
   - 전송 단위: Bit
   - 장비: 허브, 리피터
2. 데이터링크 계층(Data Link Layer)
   - 인접 시스템 간 데이터 전송, 전송 오류 제어
   - 동기화, 오류 제어, 흐름 제어, 회선 제어
   - 프로토콜
       - HDLC
       - PPP
   - 전송단위: 프레임(Frame)
   - 장비
       - 브리지
       - 스위치
3. 네트워크 계층(Network Layer)
   - 단말기 간 데이터 전송을 위한 최적화된 경로 제공
   - 프로토콜
       - IP
       - ICMP
   - 전송단위: 패킷(Packet)
   - 장비: 라우터
4. 전송 계층(Transport Layer)
   - 송수신 프로세스 간의 연결
   - 신뢰성 있는 통신 보장
   - 데이터 분할, 재조립, 흐름 제어, 오류 제어, 혼잡 제어
   - 프로토콜
     - TCP
     - UDP
   - 전송단위: 세그먼트(Segment)
   - 장비
     - L4
     - 스위치
5. 세션 계층(Session Layer)
   - 송수신 간의 논리적인 연결
   - 연결 접속, 동기제어
   - 프로토콜
     - RPC
     - NetBIOS
   - 전송단위: 데이터(Data)
   - 장비: 호스트(PC 등)
6. 표현 계층(Presentation Layer)
   - 데이터 형식 설정, 부호 교환, 암복호화
   - 프로토콜
     - JPEG
     - MPEG
   - 전송단위: 데이터(Data)
   - 장비: 호스트(PC 등)
7. 응용 계층(Application Layer)
   - 사용자와 네트워크 간 응용 서비스 연결, 데이터 생성
   - 프로토콜
     - HTTP
     - FTP
   - 전송단위: 데이터(Data)
   - 장비: 호스트(PC 등)

