---
title: "Computer Network (1)"
date: 2020-04-26T15:02:48+09:00
categories:
- Computer Science
tags:
- lecture
- computer science
- computer network
keywords:
- tech
math: true
#thumbnailImage: //example.com/image.jpg
---

건국대학교 컴퓨터네트워크 강의노트  
Computer Networking: A Top down Approach
<!--more-->

# Chapter 1: Introduction
* 인터넷, 프로토콜
* 네트워크 엣지: 엔드시스템, 액세스 망, 링크
* 네트워크 코어: 패킷스위칭, 서킷스위칭, 네트워크 구조
* 성능: 딜레이, 손실, throughput
* 프로토콜 레이어, 서비스 모델
* 보안
* * *
## 1. 인터넷
![IMAGE](/images/kucse-computer-network/introduction-network.png)
- 수십 억대의 컴퓨팅 디바이스.  
  - PC, Server, wireless laptop, smartphone  
  - 호스트: end systems  
  - 네트워크 앱을 실행한다. (apache, nginx ... )
- 커뮤니케이션 링크  
  - fiber, copper, radio, satlelite  
  - 전송율: bandwidth
- 패킷 스위치  
  - 라우터, 스위치
- 인터넷: "network of networks"  
  - ISP가 서로 연결되어 있음
- 프로토콜: Control sending/receiving of messages  
  - TCP, IP, HTTP - 국제표준 오픈 프로토콜, 소유권 x  
  - Skype - Property 프로토콜. 
- 인터넷 표준
  - RFC
  - IETF
- Service View
  - 애플레키에션에 서비스를 제공하는 인프라  
  Web, VoIP, email, gaems, e-commerce, social nets, ...
  - 애플레키에션에 프로그래밍 인터페이스를 제공
- 프로토콜이란  
packet -> header | payload  
header -> dest, source ...  
  - format 정의 
  - 메세지 전송/수신 순서 정의  
  - 어떤 메세지를 받았을 때 취하는 액션 정의
![IMAGE](/images/kucse-computer-network/introduction-internet-protocol.png)


## 2. 네트워크 엣지
![IMAGE](/images/kucse-computer-network/introduction-network.png)
- 네트워크 엣지:
  - 호스트: 클라이언트와 서버
  - 서버는 보통 데이터센터
- 액세스 네트워크, 물리 매체
  - wired/wireless communication links
- 네트워크 코어
  - Interconnected routers
  - "network of networks"
- 엣지 시스템을 엣지라우터에 연결?
  - residential access nets
  - institutional access networks
  - mobile access networks
- Keep in mind
  - Bandwidth(bits/second) of access network?
  - Shared of dedicated?
    - shared: 공유
    - dedicated: 각자의 bandwidth

### Access network: digital subscriber line (DSL)
![IMAGE](/images/kucse-computer-network/introduction-access-net-dsl.png)
- ADSL: Asymetrical. 다운로드 > 업로드. common case.
- SDSL: Symetrical. 다운로드 = 업로드. ex) Youtube streamer

### Access network: cable network
![IMAGE](/images/kucse-computer-network/introduction-access-net-cable.png)

### Access network: home network
![IMAGE](/images/kucse-computer-network/introduction-access-net-home.png)

### Access network: Enterprise (Ethernet)
![IMAGE](/images/kucse-computer-network/introduction-access-net-ethernet.png)
- 라우터를 통해 L3로
- 왼쪽 Ethernet switch
  - 부분적으로 보면 Access network
  - 전체적으로 보면 Edge network
- 서버
  - 클라우드나 데이터센터에 위치
  - 메일 서버, 웹서버
- L3 라우터
- L2 스위치

### Access network: Wireless
- shared wireless acess network는 엔드시스템을 라우터에 연결 시킨다.
- WireLess LAN
![IMAGE](/images/kucse-computer-network/introduction-access-net-wireless-1.png)
  - 공유기

- Wide-area wireless access
![IMAGE](/images/kucse-computer-network/introduction-access-net-wireless-2.png)
  - 기지국
  - 3G, 4G:LTE


### Hosts: 데이터 패킷을 보낸다
- 패킷 전송지연
$$ (패킷전송delay) = (L-bit패킷전송에 필요한시간) = L(bits) / R(bits/sec) $$
- R: transmission rate, bandwidth
  - ex) 10G, 1G/s => 10초 걸린다.

### 물리 매체
- bit
- physical link
- wired
  - guided media: 유선
    - 구리, 광섬유, 동축케이블(coax)
    - twisted pair
      - 절연 구리줄, 구리줄 8개  
      - 꼬아놔야 간섭을 안한다.
      - category 5: 100Mbps, 1Gbps
      - category 6: 10Gbps
  - ungided media: 무선
    - 라디오
- coax, coaxical cable
  - 동축케이블
  - broadband라고 불린다. 주파수 분할을 하므로
  - HFC. 가까이 올때까지 fiber, 와서는 copper
  - 구리줄 -> 전파장. 간섭 및 에러율 증가. 
- fiber optic cable
  - 광섬유 -> 빛. 에러율 감소.
    - 중간에 세기가 약하진다.
    - 리피터가 필요하다.
    - 노이즈가 적다
- radio
  - 양방향
  - 소리를통해
  - 반사 때문에 작은신호(노이즈)가 이따금씩
  - 장애물, 간섭
  - types
    - 지상파 microwave
    - LAN
    - wide-area: 4G
    - satelite: 인공위성
      - kbps ~ 45Mbps
      - 270ms e2e delay.
      - 전자위성 vs 저궤도 위성

## 3. 네트워크 코어
- 패킷 스위칭
  - 애플리케이션 layer 메세지를 패킷으로 쪼갠다.
  - store-and-forward
![IMAGE](/images/kucse-computer-network/introduction-core-packet-store-forward.png)
    - 링크가 available일 시 전송
    - 라우터는 목적지 계산 (라우팅)
      - 링크가 available 될 떄까지 저장 - store
      - 링크가 available 일 시 전송 - forward
  - queueing delay, loss
![IMAGE](/images/kucse-computer-network/introduction-core-packet-queue.png)
    - 큐: 저장
    - 전송 속도가 따라가지 못해서 버퍼가 가득 차면 패킷 드랍
    - UDP라면 다시 보내지 않는다. error
    - TCP라면 다시 보낸다.
  - routing, forwarding
![IMAGE](/images/kucse-computer-network/introduction-core-packet-routing.png)
    - routing: source-destination 경로 결정
    - forwarding: router input을 적절한 output으로 packet을 옮긴다.
- 서킷 스위칭
![IMAGE](/images/kucse-computer-network/introduction-core-circuit.png)
  - 사용중인 회선은 다른 곳에서 쓸 수 없다.
  - 대신 bandwidth가 다 자기꺼.
  - traditional telephone networks
    - 요즘은 VoIP
  - FDM vs TDM
![IMAGE](/images/kucse-computer-network/introduction-core-circuit-fdm-tdm.png)
    - FDM: 주파수 분할 다중화
    - TDM: 시분할 다중화
- 패킷 스위칭 vs 서킷 스위칭
![IMAGE](/images/kucse-computer-network/introduction-core-packet-vs-circuit.png)
  - 패킷 스위칭이 더 많은 유저를 쓸 수 있게 해준다.
  - ex)
    - 1Mb/s link
    - each user
      - 100kb/s when "active"
      - active 10% of time
    - circuit switching
      - 10 users
    - packet switching
      - 10명이 넘어가도 store and forward
      - 35명의 유저라도 10개선을 동시에 쓸 확률은 낮다. (0.0004 ?)

- 패킷 스위칭이 승자다.
  - bursty data
    - 리소스 sharing
    - 간단, UDP의 경우 call setup이 없다.
  - excessive **congesting** possbile
    - 패킷 지연, 손실 가능성
    - 프로토콜이 packet switching을 염두에 두어야 한다.
      - error, retry
  - circuit-like behavior?
    - VoIP, video transfer
    - bandwidth 보장이 필요

- Internet structure: network of networks
  - 액세스 망, access ISP
  - 경제, 국가 정책에 따라 발전
  - 각 ISP마다 connect?
  ![IMAGE](/images/kucse-computer-network/introduction-structure-networks.png)
    - O(n^2)연결
  - Global ISP?
  ![IMAGE](/images/kucse-computer-network/introduction-structure-global-ISP.png)
    - 각 나라마다 정책
    - 전세계 cover ISP는 불가능
  - 지역별 ISP
  ![IMAGE](/images/kucse-computer-network/introduction-structure-regional-ISP.png)
    - ISP끼리 연결이 안된다.
  - 지역별 Interconnected ISP - IXP
  ![IMAGE](/images/kucse-computer-network/introduction-structure-regional-IXP.png)
    - 같은 계층에 속한 라우터 - peering
    - ISP가 복잡해지면?
    - Internet exchange point
      - 2개 이상 ISP연결. 다자간 연결
      - IXP 센터
      - 서울 POP, 대전 POP ...
  - 아직 전세계 cover는 되지 않는다. + regional net.
  ![IMAGE](/images/kucse-computer-network/introduction-structure-regional-net.png)
    - 미국, 러시아, 동남아시아 ... 
    - 미국 서부 regional net + 미국 동부 regional net
    - 미국 전역 cover ISP
    - 아시아 전역 cover ISP
    - 동북아시아 regional net
    - (+) backbone 망. 고속도로 역할
  - (+) Content provider network. CDN 
  ![IMAGE](/images/kucse-computer-network/introduction-structure-cdn.png)
    - Google, Microsoft, Akamai

## 4. loss와 delay는 어떻게 발생하는가
- 라우터 buffer에서 packet queue
![IMAGE](/images/kucse-computer-network/introduction-delay.png)  
  - 전송 중 패킷 -> delay
  - 패킷 queueing -> delay
  - 버퍼 -> loss
  - output link capacity.
    - 나가는 양보다 들어오는 양이 많으면.

![IMAGE](/images/kucse-computer-network/introduction-delay-calc.png)  
$$ d(nodal) = d(proc) + d(queue) + d(trans) + d(prop) $$
- d(proc) : nodal processing
  - 어느 라우터에 보낼 것인지 판단
- d(queue) : queueing delay
  - 버퍼에 쌓여서 전송을 기다리는 시간
  - congestion이 있다면 delay up, 없다면 delay down
- d(trans) : transmission delay
  - L: packet length
  - R: link bandwidth
  - d(trans): L/R
- d(prop) : propagation delay
  - 빛의 속도로 전파.
  - d: length of physical link
  - s: propagation speed (빛의 속도. 2x10^8m/s)
  - d(prop): d/s
  - d(prop)과 d(trans)는 많이 다르다.

- Queueing delay
  - R: link bandwidth (bps)
  - L: packet length (bits)
  - a: average packet arrival rate
  - La/R ~ 0: avg.queueing delay small
  - La/R -> 1: avg.queueing delay large
  - La/R > 1: 전송할 수 있는 양보다 더 많이 오고있다. delay infinite.

- "Real" Internet delays and routes
  - 진짜 Internet delay & loss 는?
  - `traceroute`: delay 측정. 네트워크 디버깅
    - 목적지까지 가는 경로의 모든 라우터에 3개의 패킷을 전송
    - 3개 보낼때 마다 i번째 라우터가 현재 받은 시간을 보낸다.

- Packet loss
![IMAGE](/images/kucse-computer-network/introduction-delay-packet-loss.png)  
  - 쌓이다 넘치면 drop.

- Throughput
![IMAGE](/images/kucse-computer-network/introduction-delay-throughput.png)
  - 시작점과 목적지 사이에 단위시간당 처리한 양. rate (bits / time unit)
    - instaneous: 한 순간의 throughput
    - average: 평균
  - R(s) < R(c)
    - R(s) 1초. R(c) 10초 -> 10초
  - R(s) > R(c)
    - R(s) 10초, R(c) 1초 -> 10초
  - bottle enck link.
    - bottleneck 용량이 전체 용량.
    - bottleneck을 찾아서 해결하는 게 중요하다.
  - Internet scenario.
![IMAGE](/images/kucse-computer-network/introduction-delay-scenario.png)
    - Connection 별 end-end throughput은
      - min(R(c), R(s), R/10)
      - R(c) 또는 R(s)가 보통 bottleneck.
    
## 5. 프로토콜 레이어    
- 네트워크는 복잡하고, 많은 조각들이 존재
  - hosts
  - routers
  - links of various media
  - applications
  - protocols
  - hardware, software
  - 어떻게 구성해서 네트워크를 형성하지?
    - 복잡한 문제를 해결하기 위해 "divide and conquer"
    - layering, 각 층마다 identification
    - "change in gate procedure doesn't affect rest of system"  
      한 모듈 바꾸는 게 다른 시스템에 영향을 주지 않는다.
- Internet protocol stack
![IMAGE](/images/kucse-computer-network/introduction-layering-protocol-stack.png)
  - Application
    - FTP, SMTP,
  - Transport: 전송계층
    - TCP, UDP
  - Network: 망계층
    - IP, routing protocols
  - link
    - 이더넷
  - physical
    - bits "on the wire"
- ISO/OSI reference model
![IMAGE](/images/kucse-computer-network/introduction-layering-protocol-osi-reference.png)
  - presentation: 표현계층
    - 암호화, 압축
    - machine-specific conventions
  - session
    - synchronization
    - checkpointing: 영화 다운로드, recovery of data exchange
  - 이런건 application에서 구현하자.
- Encapsulation / Decapsulation
![IMAGE](/images/kucse-computer-network/introduction-layering-encapsulation.png)

## 6. 보안
- 네트워크 보안
  - 침입자가 어떻게 공격할 것인가.
  - 어떻게 방어할 것인가
  - 어떻게 면역있게 설계할 것인가.
  - 다시 설계는 안되고 ...
    - block chain.
    - 단점도 있다.
  - 필요할 때마다 필요한 계층에 security를 고려할 것
- Bad guy: 인터넷을 통해 호스트에 malware를 넣었다. 
  - virus: 이메일... 사람이 직접 클릭해야 한다.
  - worm: 자가복제로 감염. 수동적으로 또는 자동적으로 수행
  - spyware: 패스워드, 크레딧카드, 
  - botnet: span, DDos
- Bad guy: 서버, 네트워크 인프라 공격
![IMAGE](/images/kucse-computer-network/introduction-security-dos.png)
  - Dos (Denial of Service)
    - 타겟을 고르고
    - botnet
    - 타겟에게 패킷을
- Bad guy: 패킷 스니핑
![IMAGE](/images/kucse-computer-network/introduction-security-packet-sniff.png)
  - wireshark
- Bad guy: fake address 사용
![IMAGE](/images/kucse-computer-network/introduction-security-fake-address.png)

# Chapter 2: Application Layer
- principle
- web, http
- SMTP, POP3, IMAP
- DNS
- P2P
- Video Streaming, CDN
- TCP, UDP
* * *
## 1. principle
- Creating a network app
  - 엔드시스템에 올라가는 프로그램
  - network에서 통신
  - network-core device를 위한 소프트웨어를 작성할 필요는 없다.
    - network-core device는 user application을 실행하지 않는다.
- Client-Server architecture
  - server
    - always-on host
    - 고정 IP
    - 데이터센터. 수십개의 논리적 1서버
  - clients
    - 서버와 통신
    - 붙었다가 끊어졌다가.
    - Dynamic IP addresses
    - 클라이언트끼리 직접 통신 x
      - 화상회의 -> 중간에 controller가 있다.
- P2P
  - always-on server가 없다.
  - peer가 서버도 되고 클라이언트도 되고.
  - self-scalable
  - intermittently connected, and change IP addresses
    - 복잡한 management
- process: host에서 실행되는 프로그램
  - 동일 호스트에서 2개 이상 프로세스와 통신 - IPC
    - client process: 연결을 맺는다.
    - server process: 연결을 기다린다.
- Addressing processes
![IMAGE](/images/kucse-computer-network/application-principle-sockets.png)
  - 서로 다른 호스트
  - 클라이언트가 서버를 찾아야 한다. (IP)
  - 소켓
    - IP를 알아야한다. 32 bit address
    - port도 알아야한다. -> TCP/UDP. 80(http), 25(mail)
- App-layer protocol defines
  - types
    - ex) request, response
  - message syntax
  - message sementics
  - rules
    - 언제 어떻게 프로세스가 메세지를 전송/수신 할 것인가.
  - open protocols
    - RFC
    - interoperability (상호 운용성)
    - ex) HTTP, SMTP
  - proprietary protocol
    - ex) skype
- App에서 필요한 transport service?
  - data integrity
    - 앱에 따라 100% reliable data transfer를 필요로 한다.
    - 어떤 앱은 손실이 있어도 괜찮다.
  - timing
    - 앱에 따라 low delay를 요구 (Internet telephony, interactive games)
  - throughput -> bandwidth
    - multimedia 앱은 throughput 양을 좀 많이 필요
    - ftp, email은 좀 기다려도 된다.
  - security
    - encryption, data integrity, ...


## 2. Web and HTTP
- Web page
  - object로 구성
  - object -> HTML, JPEG, Java applet, audio file, ...
  - URL로 접근
```
www.someschool.edu/someDept/pic.gif

host name : www.someschool.edu
path name : /someDept/pic.gif
```
- HTTP: hypertext transfer protocol
![IMAGE](/images/kucse-computer-network/application-http-overview.png)
  - Web의 application layer protocol
  - client: browser. request
  - server: response. reply
  - TCP 사용
    - reliable. 80 port
    - client - initiate TCP connection to server
    - server - accept TCP connection
    - HTTP 메세지 교환
    - TCP Connection closed.
  - statless - request, response
    - 어떤 state도 기억하지 않음
    - <-> stateful. 복잡하다
      - 과거의 상태를 기억
      - 돌아가는 point (check point), sync.
      - 관리가 복잡하다
- HTTP connections
  - non-persistent HTTP
    - one object - one connection
    - object 보내고 끊고, 보내고 끊고...
    - 10 object - 10 connection
  - persistent HTTP
    - single TCP connection
- Non-persistent HTTP: response time
![IMAGE](/images/kucse-computer-network/application-http-response-time.png)
  - RTT (Round Trip Time)
  - HTTP response time: 2 RTT + file transmission time
  - 2 RTT per object
- Persistent HTTP
  - server - connection
  - subsequent HTTP messages
  - 1 RTT
- HTTP Request messages
  - request, response
  - ASCII
```text
GET /index.html HTTP/1.1\r\n
Host: www-net.cs.umass.edu\r\n
User-Agent: Firefox/3.6.10\r\n
Accept: text/html,application/xhtml+xml\r\n
Accept-Language: en-us,en;q=0.5\r\n
Accept-Encoding: gzip,deflate\r\n
Accept-Charset: ISO-8859-1,utf-8;q=0.7\r\n
Keep-Alive: 115\r\n
Connection: keep-alive\r\n
```
  - general format
![IMAGE](/images/kucse-computer-network/application-http-general-format.png)
  - Uploading form input
    - POST
      - 보통 form input을 포함
      - input은 모두 body에.
  - Method types
    - HTTP/1.0
      - GET. POST, HEAD
    - HTTP/1.1
      - GET, POST, HEAD
      - PUT: server upload
      - DELETE: delete file
- HTTP response message
```text
HTTP/1.1 200 OK\r\n
Date: Sun, 26 Sep 2010 20:09:20 GMT\r\n
Server: Apache/2.0.52 (CentOS)\r\n
Last-Modified: Tue, 30 Oct 2007 17:00:02
GMT\r\n
ETag: "17dc6-a5c-bf716880"\r\n
Accept-Ranges: bytes\r\n
Content-Length: 2652\r\n
Keep-Alive: timeout=10, max=100\r\n
Connection: Keep-Alive\r\n
Content-Type: text/html; charset=ISO-8859-
1\r\n
\r\n
data data data data data ...
```
  - HTTP response status codes
    - 200 OK
    - 301 Moved Permanently
    - 400 Bad Request
    - 404 Not Found
    - 505 HTTP Version Not Supported
- User-server state: cookies
  - authorization (뭘 할 수 있나) (유의. authentication - 신분 확인)
  - shopping cart
  - recommendation 
  - user session state
  - cookies and privacy. 보안문제
- Web caches
![IMAGE](/images/kucse-computer-network/application-http-web-caches.png)
  - 있으면 proxy에서 가져온다. (cache)
  - (+) bandwidth 절약
  - (-) proxy 병목. SPoF
  - Why web caching?
    - response time down
    - bandwidth / traffic down
    - Content provider에게 유리
  - Proxy
    - client / server 역할
- Conditional GET
![IMAGE](/images/kucse-computer-network/application-http-conditional-get.png)
  - `If-modified-since: <date>`

## 3. 전자 메일
- Three major components
  - user agents -> 웹브라우저, outlook
  - mail server - `mail.konuk.ac.kr`, `gmail.com`
  - SMTP: simple maill transfer protocol
  ![IMAGE](/images/kucse-computer-network/application-mail-smtp.png)
- User agent
  - "mail reader"
  - composing, editing, reading mail
- Mail server
  - mail box
  - message queue
  - SMTP
    - client: sending mail server
    - "server": receiving mail server
- SMTP [RFC 2821]
  - TCP, port 25
  - direct transfer
  - persistent. 100개 있으면 다 받고 connection을 끊는다.
    - handshaking
    - transfer of message
    - closure.
  - command/response interaction (HTTP 처럼)
    - command: ASCII text
    - response: status code
  - 시나리오
![IMAGE](/images/kucse-computer-network/application-mail-smtp-scenario.png) 
  - SMTP interaction
```
S: 220 hamburger.edu
C: HELO crepes.fr
S: 250 Hello crepes.fr, pleased to meet you
C: MAIL FROM: <alice@crepes.fr>
S: 250 alice@crepes.fr... Sender ok
C: RCPT TO: <bob@hamburger.edu>
S: 250 bob@hamburger.edu ... Recipient ok
C: DATA
S: 354 Enter mail, end with "." on a line by itself
C: Do you like ketchup?
C: How about pickles?
C: .
S: 250 Message accepted for delivery
C: QUIT
S:221 hamburger.edu closing connection
```
- SMTP와 HTTP 비교
  - HTTP: pull
  - SMTP: push. 서버가 먼저 요청
  - 둘다 ASCII command / response interaction, status code
- SMTP: final words
  - persistent connection
  - message (header & body) in 7 bit ASCII
  - CRLF.CRLF - end of message
- Mail message format
![IMAGE](/images/kucse-computer-network/application-mail-message-format.png)
  - RFC 822
    - TO:
    - From:
    - Subject:
    - SMTP 시스템이 만들어주는 커맨드 FROM, RCPT TO와는 다르다.
    - Body: the "message" . ASCII only. (요즘은 멀티미디어 확장)
- Maill access protocols
![IMAGE](/images/kucse-computer-network/application-mail-protocols.png) 
  - POP [RFC 1939]. authorization
  - IMAP [RFC 1730]. 정교한 메세지 조작.
- POP3 protocol
![IMAGE](/images/kucse-computer-network/application-mail-pop3.png) 
  - authorization phase
    - client commands:
      - user
      - pass
    - server
      - +OK
      - -ERR
  - transaction phase
    - list: list message numbers
    - retr: retrieve message
    - dele
    - quit
- POP3 vs IMAP
  - POP3
    - "download and delete"
    - "download and keep"
    - stateless
  - IMAP
    - mail box에 폴더를 많이, manipulation
    - stateful
## 4. DNS
- DNS: domain name system
  - Internet hosts, routers
    - IP address (32 bit). 외우기 어렵다
      - addressing datagram 에 쓰인다.
    - "name"
      - ex) www.yahoo.com -> 사람이 쓰는거 
  - 분산 DB
    - 계층적으로
  - application layer
    - name server와 통신해서 resolve
    - application-layer protocols
    - network edge
- DNS services
  - hostnames -> IP address translation. 주요한 기능
  - host aliasing
    - canonical -> 정식이름, aliase -> 별명
  - mail server aliasing
  - load distribution
    - replicated web servers (google.com)  
      많은 IP address가 있는데, 딱 1개의 name
  - Why not centralize?
    - Single point of failure
    - traffic volume -> 호스트가 무진장 많을거고, 요청도 무진장 많을거고
    - distant centralized database. 요청자와의 거리 고려
    - maintance
- DNS: distributed hierarchical database
![IMAGE](/images/kucse-computer-network/application-dns-hierarchical.png)
  - client -> www.amazon.com
    - client queries `root` server to find `.com` DNS
    - client queries `.com` DNS server to get `amazon.com` DNS
    - client queries `amazon.com` DNS server to get IP address for www.amazon.com
  - 13개의 root name server
    - UDP 패킷사이즈... 13이 maximum
    - 복사판은 수십개
  - top-level domain (TLD) servers
    - CCTLD: `.uk`, `.fr`, `.ca`, `.jp`
    - gTLD: `.com`, `.org`, `.net`
    - `Network solutions`이 .com을 관리한다
    - `Educase`가 .edu를 관리한다
  - authoritative name server
    - 수많은 host. 
      - `engineering.konkuk.ac.kr`
      - `cse.konkuk.ac.kr` 
    - organization이 관리한다.
    - 등록해놓은 authoritative name server.
      - `ns.konkuk.ac.kr`
      - 이를 외부에 알린다.
- Local DNS name server
  - 내부적으로도 운용할 수 일다. (hierarchy를 엄격히 지키지 않는다)
  - 각 지역 ISP(residential, company, university)는 1개 갖고 있다
    - "default name server"
  - local DNS Server에 제일 처음으로 물어본다
    - local cache를 한다. (TTL 존재, 지나면 expire)
    - proxy로 작요
- DNS name resolution example
  - iterated
![IMAGE](/images/kucse-computer-network/application-dns-resolve-iterated.png)
    - 처음에 contact 하는 DNS는 local DNS server
    - 많이 쓰인다.
  - recursive query
![IMAGE](/images/kucse-computer-network/application-dns-resolve-recursive.png)
    - root에 큰 load가 걸린다.
- DNS caching, updating reords
  - cache timeout after some time (TTL)
  - TLD는 local name server에 캐시되어 있다. root를 방문할 필요가 잘 없다.
  - 호스트 IP가 바뀌어도 TTL expire까지는 모른다.
- DNS records
  - DNS: store resource records (RR)
  - RR Format: (name, value, type, ttl)
  - type=A:
    - (www.konkuk.ac.kr, 203.252.180.1, A, 24)
    - name: hostname
    - value: IP address
  - type=NS: name server
    - (konkuk.ac.kr, ns.konkuk.ac.kr, ns, 24)
    - name: domain
    - value: host name
  - type=CNAME:
    - (www.konkuk.ac.kr, web.konkuk.ac.kr, CNAME, 24)
    - name is alias for some real name
  - type=MX: mail server
    - (konkuk.ac.kr, mail.konkuk.ac.kr, MX, 24)
- DNS protocol, messages
![IMAGE](/images/kucse-computer-network/application-dns-message.png)
  - query and reply
  - identification
  - flags
    - query or reply
    - recursion
    - recursion available
    - reply is authoritative
- Inserting records into DNS
  - ex) new startup "Network Utopia"
    - DNS registrar에 `networkutopia.com` 등록
      - 2개의 RR이 등록된다.
        - (networkutopia.com, dns1.networkutopia.com, NS)
        - (dns1.networkutopia.com, 212.212.212.1, A)
    - type A: `www.networkuptopia.com`
    - type MX: `mail.networkuptopia.com`
    - 더 필요하다면 authoritative server에 등록
- Attacking DNS
  - DDoS attacks
    - root server
    - 많이 쓰는 TLD는 캐싱된다.
    - 많이 쓰는 TLD가 더 위험하다.
  - redirect attacks
    - main-in-middle
      - 가로채서 변환해서 query
      - 가로채서 reply
    - 이상한 사이트에 접속되고, 크레딧 카드를 훔쳐간다..
  - exploit DNS for DDoS
    - spoofed source
  - DNS는 공격을 많이받는다.

## 5. P2P application
- Pure P2P architecture
  - no always-on server. 
    - 항상 켜져있는 서버는 없다. 
    - 어떤 node건 서버가 된다.
  - arbitrary end system directly communicate
    - 임의의 end system이 통신이 가능하다.
  - peers are intermittently connected
    - 연결되었다가 끊어졌다가. IP가 바뀔수도 있다.
  - examples
    - BitTorrent -> file distribution
    - KanKan -> Streaming
    - Skype -> VoIP
- File distribution: client-server vs P2P
![IMAGE](/images/kucse-computer-network/application-p2p-file-distribution.png)
  - size F file을 1서버에서 N개의 peer에게 보낼 때 시간이 얼마나 걸릴까?
  - client-server
    - server 전송
      - one copy: F/u(s)
      - N copies: N * F / u(s)
    - client
      - d(min): min client download rate
      - F/d(min)
    - N에 비례하여 느려진다.
  $$ D(c~s) >= max(N*F/u(s), F/d(min) $$
  - P2P
    - 처음 one copy: F/u(s)
    - min client download: F/d(min)
    - upload: NF/(u)
    - N에 비례하여 업로드도 늘어난다.
  $$ D(P2P) >= max{F/u(s), F/d(min), N*F/(u(s)+\sum_{i=1}^nu(i))} $$
![IMAGE](/images/kucse-computer-network/application-p2p-file-graph.png)
- P2P file distribution: BitTorrent
![IMAGE](/images/kucse-computer-network/application-p2p-torrent.png)
  - 파일을 256kb Chunk로 나눈다.
    - chunk단위로 file을 공유 (send/receive)
  - 토렌트에 참여하는 peer
    - 처음에는 chunk가 없지만 시간이 지나면서 자기가 없는 chunk를 받는다.
    - tracker를 통해서 neighbor를 찾고, chunk를 upload/download
    - churn. 피어가 올 수도, 떠날 수도
    - peer가 전체 파일을 받으면
      - (selfishishly) 떠나거나
      - (altruistically) 공유를 위해 토렌트에 남거나
- BitTorrent: requesting, sending file chunks
  - requesting chunks
    - 주어진 시간에 다른 피어가 다른 파일 chunkset을 가지고 있다.
    - 주기적으로, Alice가 각 peer에게 chunk를 요청
    - 요청은 rarest first. 희귀한거 먼저.
  - sending chunks: tit-for-tat
    - chunk를 보낼 땐 아무한테나 주는게 아닌, 자신한테 가장 많이준 4개 peer에게 전송 (10초마다 바뀜)
    - 30초마다 임의의 다른 peer를 고르고 chunk를 보낸다.
  - 줄때와 받을 때의 policy가 다르다
## 6. Video Streaming and CDN
- Video streaming
  - Youtube, Netflix, hulu, kankan, akamai
  - Video traffic이 점점 늘어난다.
  - 코로나때문에 더 각광
  - zoom, skype, teams, hangout
  - challenge: 10억명의 유저를 scale
    - single-mega-video 서버는 불가능
    - 사용자측에선 1서버라고 생각하게 된다. 1 access point
  - challenge: heterogeneity. 
    - 사용자별 bandwidth가 다르다.
    - wifi, LAN ...
  - Solution: distributed, application-level infrastructure
- Multimedia: Video
  - 가장 bandwidth를 많이 먹는다.
  - 해상도도 많다. 4K, 8K, Full HD, standard HD, ...
  - 순차적 이미지 display, 24 images / sec
  - 하나하나가 digial image. 1920x1080x24...
  - 압축을 해야한다. lossless, lossy, ...
  - coding: image 내, image 간 중복제거
    - spatial: 이미지 내 같은 픽셀, RLE -> 1920b
    - temporal: it1과 it2 이미지의 차이만 저장 (압축에 유리)
  - CBR (constant bit ratio)
    - 단위시간당 encoding율이 동일 (정지화면이건 뛰는 화면이건)
    - bandwidth가 10Mbps로 보장이 되면 CBR해도 괜찮다.
  - VBR (variable bit rate)
    - 단위시간당 encoding율이 가변적 (정지화면이면 압축률 up)
  - examples.
    - MPEG1 -> CBR. 1.5Mbps
    - MPEG2 -> VBR.
    - MPEG4 -> 인터넷에서 많이 쓰임
- Streaming stored video
![IMAGE](/images/kucse-computer-network/application-video-streaming.png)
- DASH
  - Dynamic Adaptive Streaming over HTTP
  - server
    - video file을 multiple chunk로 나눈다.
    - 각 chunk는 다른 rate로 인코딩
    - manifest file. 다른 chunk의 URL 제공.
      - 사용자가 원할때마다 여기서 찾아가라. => 해상도 변경 등
  - client
    - 주기적으로 server-to-client bandwidth 측정
    - consulting manifest, 한 순간에 한 chunk를 요청
      - 현재 bandwidth에 따라서 알맞은 chunk를 갖고 온다.
      - 시간마다 갖고 오는 coding rate가 달라질 수 있다.
    - "Intelligence" at client. 클라이언트가 결정한다.
      - 클라이언트가 언제 청할지
      - 어떤 encoding rate를 요청할지
      - 어디서 가져올 것인지. (bandwidth에 따라 manifest)
- Content distribution networks
  - 서버가 분산, 복사판이 많이 있다
  - 클라이언트는 가장 가까운 곳에서 가져온다
  - challenge: 수십만 동시 유저에게 어떻게 스트리밍 하지? 어떻게 분산, 배분하지?
  - Option 1: single, large "mega-server"
    - single point of failure
    - point of network congestion
    - long path to distant clients (미국 서버 - 한국 클라이언트)
    - multiple copies of video sent over outgong link (반복적으로 같은 영상 copy)
  - Option 2: CDN
    - 여러 지리적 복사판을 만들자.
    - enter deep: access network에 CDN 서버를 두자.
      - 사용자에게 가까이
      - used by Akamai
    - bring home: access 망에 가까이 있는 전화국에 CDN 서버를 둔다.
      - access 망보단 느리다.
      - used by Limelight
  - CDN: CDN노드에 컨텐츠 카피를 저장해둔다.
  - Subscriber는 가까운 곳에서 가져온다.
![IMAGE](/images/kucse-computer-network/application-video-cdn-path.png)
  - "OTT": over the top. OTT 서비스.
    - TV만 볼게 아니라 셋탑박스로 보자.
  - 어떤 원리로 가장 가까운 서버를 찾아주는가?
![IMAGE](/images/kucse-computer-network/application-video-cdn-example.png)
  
## 7. Socket programming
![IMAGE](/images/kucse-computer-network/application-socket-overview.png)
- socket: door between application process end-end transport protocol.
  - UDP: unreliable datagram.
  - TCP: reliable, byte stream-oriented
- UDP
  - "connection"이 없다
  - no handshaking
  - sender (client) IP port
  - receiver (server) IP port
  - 데이터 손실, 순서가 바뀔 가능성
  - Application viewpoint
    - unreliable transfer. 신뢰성 없는 서비스
  - Interaction
![IMAGE](/images/kucse-computer-network/application-socket-udp-interaction.png)
- TCP
  - client must contact server
    - server는 먼저 실행되어야 한다.
    - server는 socket을 만들어둬야 한다.
  - client
    - TCP socket을 만들고 IP, port specify
    - client가 socket을 만들때 client TCP - server TCP connection 성립
    - client가 오면 connection. server TCP는 socket을 만들고 특정 client와 통신
  - application view point
    - reliable
    - in-order byte-stream
  - Interaction
![IMAGE](/images/kucse-computer-network/application-socket-tcp-interaction.png)

# Chapter 3: Transport Layer
- transport-layer services
- Multiplexing, demultiplexing (TCP, UDP)
- reliable data transfer (TCP)
- Flow control. buffer가 넘치지 않도록 받을 수 있을 만큼만 보낸다.
- Congestion Control. 패킷 네트워크 성능 저하 방지.
* * * 
## 1. transport-layer services
- Transport services and protocol
  - logical end-to-end communication 제공. (physical은 복잡하다)
  - 긴 message를 짤라서 보낸다. (segment)
  - reassemble. segment를 다시 모은다.
  - TCP, UDP, SCTP
- Transport vs Network layer
  - host는 1개 (ip), 프로세스는 여러개 (port)
    - host까지 찾아가는 network layer
    - process까지 찾아가는 transport layer. 
- Internet transport-layer protocol
  - reliable, in-order delivery (TCP)
    - congestion control
    - flow control
    - connection setup
  - unreliable, unordered (UDP)
    - IP에서 그닥 더 하는건 없다.
  - not availble
     - delay guarantee
     - bandwidth guarantee
## 2. Multiplexing / demultiplexing
![IMAGE](/images/kucse-computer-network/transport-multiplexing.png)
- Connection을 2개 이상 만든다.
  - multiplexing. 
    - 전송할 때 transport header 추가 (나중에 demultiplexing할때 까볼 header)
  - demultiplexing
    - 들어온 패킷을 까서 header info를 보고 올바른 socket에 데이터 전달
![IMAGE](/images/kucse-computer-network/transport-multiplexing-demultiplex.png)
  

## 3. UDP
- UDP: User Datagram Protocol [RFC 768]
  - "no frills". IP에서 더 해주는 게 거의 없다.
  - "best effort". 최선을 다하겠다. guarantee 는 못하겠다.
    - lost
    - out of order
  - connectionless
    - no handshaking. 바로 보낸다.
    - UDP segment가 독자적으로 간다. connection 다라 가는게 없다.
  - UDP use
    - streaming multimedia apps. 
      - loss tolerant. 패킷이 없어져도 큰 문제는 아니다
      - rate sensitive. 대역폭 guarantee 필요
    - DNS
    - SNMP 망관리프로토콜 (Simple Network Management Protocol)
  - UDP 위의 reliable transfer
    - application layer에서 reliability
    - application-specific error handling
- UDP segment header
![IMAGE](/images/kucse-computer-network/transport-udp-segment.png)
  - no connection establishment (connection은 delay 가능성이 있으니)
  - simple: no connection state at sender, receiver
  - small header size (TCP 에 비해서)
  - no congestion control. (빨리 보낼 수 있는대로 보낸다.)
- UDP checksum
  - 계산이 빨라야한다
  - sender
    - segment content/header를 16비트 단위로 짤라서 계산
    - checksum: 더해서 1의 보수를 취한다.
    - 그리고 header에 put.
  - receiver
    - received segment에서 checksum 계산.
    - 비교해서 값이 다르면 error
      - DNS -> 에러처리
      - VoIP -> 에러가 없어도 크게 지장이 없다.
  - example
![IMAGE](/images/kucse-computer-network/transport-udp-checksum-example.png)


## 4. Principles of reliable data transfer
![IMAGE](/images/kucse-computer-network/transport-reliable-implement.png)
- sender가 주는 그대로 receiver가 받아야한다.
  - rdt_send()
  - udt_send()
  - rdt_rcv()
  - deliver_data()
- Finite State Machine (FSM)
  - rdt 1.0: reliable transfer over a reliable channel
    - underlying channel은 완벽히 믿을 수 있다.
      - no bit errors
      - no loss of packets
![IMAGE](/images/kucse-computer-network/transport-reliable-rdt1.0.png)
  - rdt 2.0: Channel with bit errors
    - reliable channel이 아니다
    - 에러 발생 가능성
      - checksum으로 에러탐지
      - how to recover?
        - 다시 보내자.
      - ACK: 에러 없음
      - NAK: 에러 있음
![IMAGE](/images/kucse-computer-network/transport-reliable-rdt2.0.png)
    - Fatal flow
      - ACK, NAK가 오는 것도 에러가 있을 수 있다
      - 아니면 ACK duplicate
      - 각 패킷에 seq number를 붙이자
      - Stop and Wait 프로토콜
        - sender가 1 패킷을 보내고 receiver response를 기다린다.
      - ACK -> 다음꺼
      - NAK -> 지금꺼 다시
  - rdt 2.1: sender
![IMAGE](/images/kucse-computer-network/transport-reliable-rdt2.1-sender.png)
  - rdt 2.1: receiver
![IMAGE](/images/kucse-computer-network/transport-reliable-rdt2.1-receiver.png)
  - rdt 2.2: NAK-free protocol
    - 보통 ACK만 쓴다.
![IMAGE](/images/kucse-computer-network/transport-reliable-rdt2.2.png)


  