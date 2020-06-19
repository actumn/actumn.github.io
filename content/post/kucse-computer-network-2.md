---
title: "Computer Network (2)"
date: 2020-06-19T17:58:01+09:00
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

건국대학교 컴퓨터네트워크 강의노트 2  
Computer Networking: A Top down Approach
<!--more-->


# Chapter 3: Transport Layer (Cont'd)

- Finite State Machine (FSM)
  - rdt 3.0: Timing
    - sender가 ACK를 기다리는데, 어느정도 기다려도 오지 않으면 timeout
    - reasonable timeout
    - ACK가 그냥 delay 되는 거라면 retransmission은 중복. seq가 있어서 receiverㅏ 처리
![IMAGE](/images/kucse-computer-network-2/transport-reliable-rdt3.0.png)
![IMAGE](/images/kucse-computer-network-2/transport-reliable-rdt3.0-action1.png)
![IMAGE](/images/kucse-computer-network-2/transport-reliable-rdt3.0-action2.png)
    - Performance of rdt
      - 동작은 좋은데 성능이 구리다
      - ex) 1Gbpgs link, 15ms prop delay, 8000bit packet
$$ D(trans) = L / R = 8 microsecs $$
$$ U(sender) = (L / R) / (RTT + L / R) = 0.008 / 30.008 = 0.00027 $$
![IMAGE](/images/kucse-computer-network-2/transport-reliable-rdt3.0-stop-and-wait.png)
      - 점유율이 0.00027. bandwidth 낭비. RTT동안 전송이 없으니까.


- Pipelined protocols.
  - 병렬로 여러개 보낸다. buffer가 여러개 필요하다
    - go-back-N
    - Selective Repeat
  - Pipelining: increased utilization
![IMAGE](/images/kucse-computer-network-2/transport-pipelining-utilization.png)
$$ U(sender) = (3L / R) / (RTT + L / R) = 0.024 / 30.008 = 0.00081 $$
    - 3배 좋아짐.

  - Go-back-N
    - n개: window size. [0 ~ n-1]
    - ack없이 n개까지 보낸다.
    - **Cummulative ack**
      - 1, 2, 3, 4 있으면 4만 ack.
    - 가장 처음 패킷 timer 측정
    ack
  - Selective Repeat
    - ack없이 n개까지 보낸다.
    - **Inidividual ack**
      - 1, 3 패킷마다 개별적 ack
    - 매 패킷 timer 측정

- Go-Back-N: sender
  - sender도 reciever도 window 를 운영.
![IMAGE](/images/kucse-computer-network-2/transport-gbn-sender-window.png)
  - Sliding window: ACK를 받을때 마다 window가 움직인다. 
  - duplicate ACK 가능성.
  - 제일 오래된 패킷 기준 timeout. 다 다시 보낸다.
  - sender extended FSM
  - receiver extended FSM
    - ACK-only: 가장 큰거
    - out-of-order pkt:
      - discard: no receiver buffering
      - re-ACK: seq 가장 높은거.
  - GBN in action
![IMAGE](/images/kucse-computer-network-2/transport-gbn-action.png)
    - 3, 4, 5 잘 갔는데 다시보낸다. 조금 비효율적.
- Selective repeat
  - GBN은 timeout시 다시보내는 패킷이 많다. (순서가 바뀌어오는거 다 버린다.)
  - error난 packet만 재전송
  - receiver: individually ack
    - 매 패킷마다 timer
  - window
![IMAGE](/images/kucse-computer-network-2/transport-selectrepeat-window.png)
    - sedner
      - 데이터 available window, 패킷 전송
      - timeout: 에러난 패킷만 재전송
      - ACK
        - 나머지는 놔둔다
  - Select repeat in action
![IMAGE](/images/kucse-computer-network-2/transport-selectrepeat-action.png)
  - 단점
    - 패킷마다 timer
    - buffer management
    - 딜레마
      - 윈도우 사이즈가 충분히 커야한다.

## 5. TCP
프로토콜이 제법 복잡하다.
- Overview
  - point-to-point: 1 sender - 1 receiver
  - reliable, in-order-byte
    - 시간이 좀 걸린다.
    - bandwidth 보장 x
  - pipelined (window size N)
    - TCP congestion / flow control
  - full duplex data
    - 양방향 통신
    - MSS: maximum segment size. 패킷 최대 길이 2^16
  - Connection-ordiented
    - handshaking 한다.
  - flow controlled

- TCP segment structure
![IMAGE](/images/kucse-computer-network-2/transport-tcp-segment.png)
  - 총 크기가 TCP는 20 bytes, UDP는 8 bytes. 옵션이 있으면 더 길다.
  - sequence number: 0 ~ 2^31-1
  - U: 급하다
  - A: ACK only
  : P: Push. buffer가 차지 않았다.
  - R: RST. 비정상종료
  - S: SYN
  - F: FIN
  - receive window: window size N은 가변적. 
    - 매 윈도우 size는 0 ~ 2^16-1
    - 3 -> 3만 넘치지 않게 보내라
    - => flow/congestion control
  - urg data pointer: urgent data 위치

- TCP seg, numbers, ACKS
  - Sequence numbers
    - segment data의 첫번째 byte number
    - 첫번째 패킷: 1, length 1024, MSS
    - 두번째 패킷: 1025
  - acknowledgements
    - cumulative ACK
      - 첫번째 패킷 SEQ 1, MSS 1024 => ACK 1025
      - 다음 SEQ 1025를 기대한다.
![IMAGE](/images/kucse-computer-network-2/transport-tcp-seqack.png)
- TCP round trip time, timeout
  - timeout이 RTT보다는 커야지
    - 그런데 RTT는 유동적
    - too short: 불필요한 재전송
    - too long: slow reaction
  - RTT를 어떻게 재지?
    - sampleRTT: 정상적인 패킷 RTT 계산. 최근 몇개 평균
    $$ EstimatedRTT = (1 - a) * EstimatedRTT + a * SampleRTT $$s
      - 보통 a = 0.125
    - timeout interval: EstimatedRTT + "safety margin"
      - EstimatedRTT 변동이 컸다면 -> safety margin도 크게
    $$ DevRTT = (1-b) * DevRTT + b * (SampleRTT - EstimatedRTT) $$
      - 보통 b = 0.25
    $$ TimeoutInterval = EstimatedRTT + 4*DevRTT $$
     
- Reliable Data Transfer
  - RDT services
    - pipelined segments
    - culmulative acks
    - signle transmission timer
  - retransmission triggered
    - timeout events
    - duplicate acks
  
  - TCP sender events
    - app에서 데이터 전송하려 할 때
      - segment를 seq로 만든다.
      - seq -> byte-stream. 첫번째 byte의 byte number
      - timer -> oldest unacked segment 기준
      - expiration interval: `TimeoutInterval`
  - timeout
    - Retransmission
    - Go-Back-N 이니까 oldest부터 다시 보낸다.
  - ack received
    - window sliding
    - timer restart
  - TCP sender
  - TCP retransmission scenarios
![IMAGE](/images/kucse-computer-network-2/transport-tcp-retransmission.png)
![IMAGE](/images/kucse-computer-network-2/transport-tcp-retransmission-2.png)
  - TCP fast transmit
    - time-out period가 상대적으로 길다
    - duplicate ACK가 쌓이면 transmit
    - "triple duplicate ACK" => resend unacked segment with smallest seq #
![IMAGE](/images/kucse-computer-network-2/transport-tcp-fastretransmit.png)
- TCP Flow control
![IMAGE](/images/kucse-computer-network-2/transport-tcp-flowcontrol.png)
  - (receiver control)
  - sender의 buffer가 넘어가지 않도록 속도를 조절해서 보내는 것.
![IMAGE](/images/kucse-computer-network-2/transport-tcp-flowcontrol-buffer.png)
  - rwnd: receive window, TCP header 값. 패킷 보낼때 마다 써서 보낸다.
  - rwnd에 따라 조절해서 전송
- connection management
![IMAGE](/images/kucse-computer-network-2/transport-tcp-connection-management.png)
  - sender와 receiver의 handshake
  - connection을 맺으면서 seq, rcvBuffer
  - 2-way handshake
![IMAGE](/images/kucse-computer-network-2/transport-tcp-2way.png)
    - 항상 잘 작동할까?
      - variable delay
      - message loss
      - failure scenarios
![IMAGE](/images/kucse-computer-network-2/transport-tcp-2way-failure.png)
      - 2way는 모자라다.
  - 3-way handshake
![IMAGE](/images/kucse-computer-network-2/transport-tcp-3way.png)
![IMAGE](/images/kucse-computer-network-2/transport-tcp-3way-fsm.png)
  - TCP: closing connection
    - 4-way handshake
      - FIN bit가 1인 Segment를 보낸다.
      - 받은 FIN에 ACK 응답
![IMAGE](/images/kucse-computer-network-2/transport-tcp-4way.png)

## 6. principles of congestion control
패킷이 너무 많이 생겼을 때  
RTT가 너무 많아졌을 때, 덜 만들자

- Congestion
  - "source가 데이터를 너무 많이, 빨리 보내서 네트워크가 이를 처리하기 어려운 상태"
  - flow control과는 다르다
    - flow control: 상대방 버퍼 문제
    - congestion control: 전체 데이터 문제
  - manifestation
    - lost packet (router-buffer overflow)
    - long delay (router-buffer queueing)
  - top-10 problem!
    - delay가 커지고, loss가 커지면 congestion
  - Scenario 1
    - one router, infinite buffer
![IMAGE](/images/kucse-computer-network-2/transport-tcp-congestion-scenario1.png)
  - Scenario 2
    - one router, finite buffer
![IMAGE](/images/kucse-computer-network-2/transport-tcp-congestion-scenario2.png)
    - Idealization: packet knowledge
      - sender가 router가 available한 상태일 때만 전송한다.
![IMAGE](/images/kucse-computer-network-2/transport-tcp-congestion-scenario2-ideal.png)
    - Idealization: known loss
      - sender가 패킷이 손실된 걸 알고 다시 보낸다.
![IMAGE](/images/kucse-computer-network-2/transport-tcp-congestion-scenario2-ideal2.png)
    - Realistic: duplication
      - 패킷은 송실될 수 있고
      - congestion
          - 다시보낸 것까지 감안한 성능이 "good put" 
![IMAGE](/images/kucse-computer-network-2/transport-tcp-congestion-scenario2-real.png)
  - Scenario 3
    - sender 4개
    - multihop paths
    - timeout / retransmit
![IMAGE](/images/kucse-computer-network-2/transport-tcp-congestion-scenario3.png)
![IMAGE](/images/kucse-computer-network-2/transport-tcp-congestion-scenario3-2.png)


## 7. TCP congestion control
- **additive increase, multiplicate decrease**
  - 증가할 땐 합 적용, 감소할땐 곱적용
    - cwnd 1씩 증가: 1 -> 2 -> 3 -. ... -> 99 -. 100
    - cwnd 절반으로 감소: 100 -> 50 -> 25
![IMAGE](/images/kucse-computer-network-2/transport-congestion-control.png)
![IMAGE](/images/kucse-computer-network-2/transport-congestion-control-cwnd.png)
  - cwnd deafult: 4096 -> 2048 -> 1024 ....
  - TCP sending rate
    $$ rate ~~= cwnd/RTT (bytes / sec) $$
- TCP slow start
![IMAGE](/images/kucse-computer-network-2/transport-congestion-control-slotstart.png)
  - initially cwnd = 1 MSS
  - double cwnd every RTT
  - 처음 rate는 slow, 점점 지수적 증가
- TCP: detecting, reacting to loss
  - timeout이 걸린다:
    - cwnd를 1로 셋팅 (반으로 줄이는 게 아니라)
    - slot start
  - TCP RENO: 똑같은 ACK가 3개 (중간에 몇개 빼먹었다)
    - cwnd를 반으로
  - TCP Tahoe: 항상 cwnd를 1로
    - (timeout 이건, 중복 3개 ACK건)
  - TCP: switching from slow start to CA (Congestion Avoidance)
  ![IMAGE](/images/kucse-computer-network-2/transport-congestion-control-switching.png)
    - variable ssthresth
      - sshthresh 까지는 지수적으로 증가
      - sshthresh 넘어가면 linear하게 
      - 문제가 발생하면 (timeout, duplicate ack 3)
        - Tahoe에선 1
        - RENO에선 반으로.
        - sshthresh는 직전 cwnd의 절반
- TCP throughput
  - W: window size
  $$ (avg.TCP throughput) = 3/4*W/RTT (bytes/s) $$

# Chapter 4: Network Layer: The Data plane
## 1. Overview of Network layer
- Network layer
  - End-to-End
  - 모든 router에 ip 가 필요 (switch, 기지국에는 필요없다)
  - router는 header를 조사해서 next를 찾는다
  - router별 network layer: routing table
- 기능
  - Forwarding
    - 다음번 hop을 찾아서 보내는 것
  - Routing
    - 길 찾기
    - 최단 경로 찾기: routing alogrithm 
- Data plane
  - forwarding 역할
    - 다음 번 hop이 어디인가
- Control plane
  - 전체 routing algorithm 돌려서 경로설정
  - Network-wide
  - traditional routing algorithm: 라우터에 구현
  - SDN: server에서 구현
- Per-router control plane
  ![IMAGE](/images/kucse-computer-network-2/network-control-router.png)
  - 라우터마다 control plane
  - 자기 라우팅은 자기가 계산
  - 주기적으로 control plane 돌려서 routing table 갱신
  - 기존 router에선 control plane, data plane이 분리되어 있지 않음
- Locally centralized control plane
  ![IMAGE](/images/kucse-computer-network-2/network-control-centralized.png)
  - 서버에서 라우팅 관리
- Network service model
  - Sender와 receiver 사이의 channel 역할
  - individual datagram services
    - guaranteed delivery
    - guaranteed delivery (< 40 ms delay)
  - flow of datagram services
    - in-order delivery
    - guaranteed minimum bandwidth
    - packet inter spacing: 어느정도 띄워서 보낸다
  - 이런거 다 안된다! 오늘날 인터넷은 그저 best effort. guarantee는 없다.
  ![IMAGE](/images/kucse-computer-network-2/network-control-models.png)

## 2. What's inside a router
![IMAGE](/images/kucse-computer-network-2/network-router-architecture.png)
- Input port functions
![IMAGE](/images/kucse-computer-network-2/network-router-input.png)
  - line termination: physical layer
  - link layer protocol: datalink layer
  - lookup, forwarding: deccentralized switching
    - ip header를 보고 "matching", next hop을 찾고
    - 목표: "line speed" processing
    - queueing
    - destination-based forwarding: destination 주소만 본다 (traditional)
    - generalized forwarding
- Destination-basesd forwarding
![IMAGE](/images/kucse-computer-network-2/network-router-forwarding.png)
  - longest prefix matching
![IMAGE](/images/kucse-computer-network-2/network-router-forwarding-prefix.png)
    - TCAM (ternary content addressable memories)
      - matching 되는 컨탠츠로 찾겠다.
      - ns 단위로 처리 가능
      - content addressable
  - Switching fabrics
![IMAGE](/images/kucse-computer-network-2/network-router-fabrics.png)
  - switching rate: rate at which packets can be transfer from inputs to outputs
- Switching via memory
  - 1 세대.
  - copy 때문에 느리다.
- Switching via a bus
  - bus contention. 버스 성능에 따라 속도가 제한
  - Cisco 5600: 32 Gbps bus. enterprise router로 충분
- Switching via interconnection network
  - bus가 여러개: bus bandwidth limitation 극복
  - banyan network (중요한 곳만 연결), crossbar, other interconnection nets
  - Cisco 12000: 60Gbps. 망 연결할 때 많이 쓴다.
    - KT 망 - KONKUK 망 - SK 망
- Input port queueing
![IMAGE](/images/kucse-computer-network-2/network-router-queuing.png)
  - HOL (Head of the line) Blocking: 큐의 처음 부분이 다른 애들이 나아가는걸 막는다.
  - 보통 queueing을 한다고 하면 output port에만
- Output port
![IMAGE](/images/kucse-computer-network-2/network-router-output.png)
  - HOL 문제 해결
  - 주로 router/swtich.
  - 보통 output port에 버퍼링한다.
![IMAGE](/images/kucse-computer-network-2/network-router-output-queuing.png)
  - HOL 문제가 없어서 queuing delay가 훨씬 줄어든다.
- buffering을 얼마나 할 것인가?
  - N: flow 포트 수. C: capacity
  $$ RTT * C / \sqrt(N) $$
  - ex) RTT: 250ms, C = 10Gbps, N: 16 포트
  - 계산결과: 2.5G bits ..?
- Scheduling mechanisms
![IMAGE](/images/kucse-computer-network-2/network-router-scheduling.png)
  - scheduling. 큐가 여러개라면 어떤 패킷을 처리할 것인가
  - FIFO scheduling
    - 먼저 들어오는 패킷을 먼저 처리
    - discard policy: 큐가 꽉 찼다면
      - tail drop: 나중에 들어온거
      - priority: 우선순위 낮은거
      - random: 임의
  - Scheduling policies: priority
![IMAGE](/images/kucse-computer-network-2/network-router-scheduling-priority.png)
    - multiple classes: 여러 priority. header에 marking 필요
    - 비율을 잘 맞춰야 한다. high N개 할 때마다 low 1개씩.
  - Round Robin (RR) scheduling:
![IMAGE](/images/kucse-computer-network-2/network-router-scheduling-rr.png)
    - 돌아가면서
  - Weighted Fair Queueing (WFQ)
![IMAGE](/images/kucse-computer-network-2/network-router-scheduling-wfq.png)
    - Round Robin 일반화.
      - highest: 3개씩
      - middle: 2개씩
      - lowest: 1개씩 

## 3. IP: Internet Protocol
![IMAGE](/images/kucse-computer-network-2/network-ip.png)
best effort.
- IP datagram format
![IMAGE](/images/kucse-computer-network-2/network-ip-datagram.png)
  - length: $2^{16}$ = 64KB 최대. 
  - 16-bit identifier, fragment offset: MTU로 짜르고, 모은다
  - upper layer: TCP, UDP
  - TCP 20 bytes, IP 20 bytes, App layer overhead
- IP fragmentation, reassembly
  - 패킷을 MTU 단위로 쪼개는 걸 fragmentation, 모으는 걸 reassemble
  - identification, offset으로 모은다
  - example
![IMAGE](/images/kucse-computer-network-2/network-ip-fragmentation.png)
    - offset은 octet단위
- IP addressing
![IMAGE](/images/kucse-computer-network-2/network-ip-address.png)
  - IP: 32-bit identifier
  - interface: connection between host/router and physical link
  - IP주소는 매 인터페이스마다 할당
  - **subnet mask**
- Subnets
![IMAGE](/images/kucse-computer-network-2/network-ip-subnets.png)
  - IP address
    - subnet part
    - host part
  - Subnet
    - 라우터 거치지 않고 서로 접근 가능한 영역
![IMAGE](/images/kucse-computer-network-2/network-ip-subnets-count.png)
    - 얘는 서브넷이 6개다.
    - CIDR이 /30 이면 호스트가 2개 (00: network address, 11: broadcast address)
  - CIDR: Classless InterDomaion Routing
![IMAGE](/images/kucse-computer-network-2/network-ip-cidr.png)
    - CIDR로 라우팅테이블 크기를 줄일 수 있다.
- IP addresses: how to get one?
  - hard-coded by system admin in a file. 수동으로 파일에 박아 넣는다.
    - Windows: control-panel ...
    - UNIX: /etc/rc.config
  - DHCP 
    - dynamic 할당 / 반납
    - renew 가능
    - address 재사용 허용
    - 더 많은 모바일 유저
  - DHCP Overview
    - host가 서브넷에 들어오면 "DHCP discover" broadcast
    - DHCP server responds with "DHCP offer"
    - host request "DHCP request"
    - server sends address "DHCP ack"
    - Well-known port: 68
  - DHCP: more than IP address
    - DHCP 는 IP address 할당 그 이상의 일을 한다
    - router의 IP를 같이 준다
    - DNS Server의 이름과 주소
    - network mask
  - DHCP: example
![IMAGE](/images/kucse-computer-network-2/network-ip-dhcp-example.png)
    - laptop에서 필요한거: 자기 주소, DNS, router
    - DHCP request: UDP, IP, Ehternet으로 encapsulate
    - Ehternet frame은 LAN에서 broadcast
  - 그러면 실제로 IP를 어디서 갖고오지?
    - 한국: KRNIC.NET -> KT, SKT
    - 전세계: ICANN
  - How does an ISP get block of addresses?
    - ICANN: Internet Corporation for Assigned
    - allocates addresses
    - manages DNS
    - assgin domain names, resolve disputes
- NAT: network address translation
![IMAGE](/images/kucse-computer-network-2/network-ip-nat.png)
  - single NAT IP address, different port -> different local addresses
  - motivaltion
    - 주소 1개를 여러 디바이스가 공유 (6만개)
    - 로컬 네트워크에서 주소 변경
      - 외부엔 알리지 않아도 된다
    - ISP에 비해 편리하다
    - 보안요소
      - 디바이스가 바깥세상에서 Visible 하지 않으므로
  - Implementation: NAT router
    - outgoing datagrams replace 
      - (source IP, port) -> (NAT IP, new port)
    - remember
    - incoming datagrams replace ("NAT table")
![IMAGE](/images/kucse-computer-network-2/network-ip-nat-table.png)
    - 16-bit port-number field
    - 단점
      - router는 layer 3 까지만 처리해야한다.
      - address 부족은 IPv6에서 해결되어야 한다.
      - end-to-end 위반
        - NAT는 중개자가 필요하다
      - NAT traversal
        - Server에 직접 연결하고 싶을 떈? 방법이 없다.
- IPv6
  - motivation
    - initial motivation: 32-bit address space가 모자르다
    - additional motiviation (header)
      - processing / forwarding speed up
      - QoS
  - IPv6 datagram format
    - fixed-length 40 byte header
    - 꼭 필요한 것만 있다
      - option이 없다.
      - fragmentation을 허용하지 않는다.
![IMAGE](/images/kucse-computer-network-2/network-ipv6-datagram.png)
      - flow label: QoS support
        - flow 가 무엇인가. src ~ dest로 연속해서 만드는 패킷들의 집합 (stream). 아직 잘 정의되진 않음
      - payload len: data length
      - next hdr: UDP냐 TCP 등의 추가 header. Next hdr에서.
      - hop limit: TTL reply 횟수
      - IPv4옵션 (여기 routing 해라) 가 없다.
      - fragmentaation, ID가 없다. flow label이 생김
      - 단순화했음에도 헤더의 길이가 늘어났다. address가 32bits -> 128bits가 됐으므로
  - Other changes from IPv4
    - Checksum. 단순화 시키기 위해 삭제. 통신장비의 성능이 좋아져서 error도 적다. processing time을 줄이기 위해
    - options: 허요, 하지만 Next header field에서.
    - ICMPv6: new version of ICMP
      - additional message type: "Packet Too Big"
      - multicast group management functions
   - Transition from IPv4 to IPv6.
      - 사람들이 잘 안바꾼다.
      - 동시에 모든 router를 upgrade할 수는 없다.
        - no "flag days"
        - IPv4와 IPv6를 같이
      - Tunneling
![IMAGE](/images/kucse-computer-network-2/network-ipv6-tunneling.png)
        - IPv6 datagram을 IPv4 datagram payload로 전송
![IMAGE](/images/kucse-computer-network-2/network-ipv6-tunneling-packet.png)
    - IPv6: adoption
      - 중국이 가장 많이 쓴다
      - Google: 8%의 client
      - NIST: 정부의 1/3 도메인
      - 20년 정도 걸릴듯
      - 20년간의 app 변경, WWW, Facebook, ...
## 4. Genralized Forward and SDN
SDN: Network 가상화 기술. (Software Defined Network)  
각 라우터의 control 모듈을 중앙서버에 모아두었다.
![IMAGE](/images/kucse-computer-network-2/network-sdn.png)
각 라우터 별 flow table이 있고, flow table을 주고 받는 프롵콜 OpenFlow가 있다.
- OpenFlow data plane abstraction
  - flow: 헤더 필드로 파악
  - rules
    - Pattern: 매칭, 목적지, 주소가 어디라면
    - Actions: drop, forward, midify, 경우에 따라선 controller에게 보낸다
    - Priority: 우선순위 높은 놈붙터 처리
    - Counters: 통계. 매칭이 일어났을 때마다 +1
![IMAGE](/images/kucse-computer-network-2/network-sdn-abstraction.png)
- OpenFlow: Flow table entries
![IMAGE](/images/kucse-computer-network-2/network-sdn-openflow.png)
![IMAGE](/images/kucse-computer-network-2/network-sdn-openflow-ex1.png)
![IMAGE](/images/kucse-computer-network-2/network-sdn-openflow-ex2.png)
  - OpenFLow abstraction
    - match + action: match와 action으로 서비스 정의
    - Router
      - match: longest prefix
      - action: forward
    - Switch
      - match: destination MAC
      - action: forward or flood(broadcast)
    - Firewall
      - match: IP
      - action: permit or deny
    - NAT
      - match: IP and port
      - action: rewrite
![IMAGE](/images/kucse-computer-network-2/network-sdn-openflow-example.png)
    
  