---
layout: single
title:  모든 개발자를 위한 HTTP 웹 기본 지식 - 001
categories: 
  - learning http basic by younghan kim
tags: 
  - http
---

## 인터넷 네트워크

### 인터넷 통신

복잡한 인터넷 안에서 두 대의 컴퓨터는 어떻게 서로 통신할까?
 
### IP(인터넷 프로토콜)

IP는 각 컴퓨터에 할당된 고유의 주소

지정한 IP 주소에 데이터 전달

패킷(Packet)이라는 통신 단위로 데이터 전달

IP 패킷은 전송 데이터를 감싼 것, 출발지 IP와 목적지 IP 등을 가지고 있음

#### IP 프로토콜의 한계

- 비연결성 : 패킷을 받을 대상이 없거나 서비스 불능 상태여도 패킷을 전송함
- 비신뢰성 : 전송중에 패킷이 사라지거나, 패킷의 순서가 바뀌었을 때 대응할 수 없음
- 한 IP를 사용하는 컴퓨터에서 서버와 통신이 필요한 어플리케이션이 둘 이상인 경우

### TCP, UDP

#### 인터넷 프로토콜 스택의 4계층

- 어플리케이션 계층 - HTTP, FTP
- 전송 계층 - TCP, UDP
- 인터넷 계층 - IP
- 네트워크 인터페이스 계층

#### 프로토콜 계층

1. 어플리케이션 
    1. 웹 브라우저, 네트워크 게임 등 프로그램이 Hello, World! 메시지 생성
    - SOCKET 라이브러리를 통해 전달
1. OS
    - TCP 메시지를 포함한 TCP 정보 생성
    - IP(Internet Protocol) 에서 TCP 정보를 포함한 IP 패킷 생성
1. 네트워크 인터페이스
    - LAN 드라이버
    - LAN 장비, LAN 카드를 통해 인터넷으로 발송

IP 패킷 정보 : 출발지 IP, 목적지 IP, 기타 ...

TCP/IP 패킷 정보 : 출발지 PORT, 목적지 PORT, 전송제어, 순서, 검증정보 ...

#### TCP 특징

- 전송 제어 프로토콜 (Transmission Control Protocol)
- 연결지향 : TCP 3 Way handshake(가상 연결)
- 데이터 전달 보증
- 순서 보장
- 신뢰할 수 있는 프로토콜로 현재는 대부분 TCP 사용

#### TCP 3 way handshake

1. 클라이언트 -> 서버 : SYN
1. 서버 -> 클라이언트 : SYN + ACK
1. 클라이언트 -> 서버 : ACK + 데이터 전송

#### UDP

- 사용자 데이터그램 프로토콜 (User Datagram Protocol)
- 하얀 도화지에 비유(기능이 거의 없음)
- 연결지향 : TCP 3 Way handshake 아님
- 데이터 전달 보증 아님
- 순서 보장 안함
- 단순하고 빠름
- IP + PORT + 체크섬
- 어플리케이션에서 추가 작업 필요

### PORT

한 컴퓨터에서 둘 이상의 서버와 연결해야 한다면? PORT 정보를 이용해서 처리가능

패킷 정보에는 출발지 IP 와 PORT, 도착지 IP 와 PORT 가 있음

PORT 를 이용하여 같은 IP 내에서 프로세스를 구분

PORT 는 0 ~ 65535 할당 가능하지만, 0 ~ 1023은 잘 알려진 포트로 사용하지 않는 것이 좋음

- FTP : 20, 21
- TELNET : 23
- HTPP : 80
- HTTPS : 443

### DNS

도메인 네임 시스템 (Domain Name System)

IP 는 기억하기 어렵고, 변경될 수 있음

따라서 DNS 를 통해 도메인명을 IP 주소로 변환하여 사용

1. 클라이언트 -> DNS 서버 : 도메인명 질의
1. DNS -> 클라이언트 : 도메인명에 해당하는 IP 응답
1. 클라이언트 -> 서버 : 접속

## 참고

- [https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC](https://www.inflearn.com/course/http-%EC%9B%B9-%EB%84%A4%ED%8A%B8%EC%9B%8C%ED%81%AC)