---
layout: posts
title:  "The benefits and costs of writing a POSIX kernel in a high-level language"
date:   2018-11-09
categories: paper-review
---
- [repo](https://github.com/mit-pdos/biscuit)
- [paper](https://www.usenix.org/system/files/osdi18-cutler.pdf)
  
## 도입
Go로 만든 Kernenl(Biscuit)의 성능을 시험하며 C 대신 HLL(High Level Language)로 커널을 개발하는 것이 합리적인가를 이야기하는 논문  
Biscuit은 POSIX를 충분히 구현하고 있으며 Go의 HLL features(클로저, 채널, map, interface, gc 등)를 충분히 사용하고 있고,  
NGINX, Redis 등의 밴치마크에서 gc, 스레드 스택 확장 등의 HLL 기능에 커널 CPU 타임이 최대 13%,  
NGINX 밴치마크에서 가장 길었던 single GC pause 시간이 115 microseconds,  
1 NGINX client request를 완료하는데 가장 긴 GC 딜레이 합이 600 microseconds 라고 함다.  
똑같은 시스템 콜, 페이지 폴트, 컨텍스트 스위치의 Go와 C 코드 성능차이 비교에선 Go가 5%~15% 느렸다고. (거의 동일한 소스코드)  

2017년에 Linux 커널에서 버퍼 오버플로우나 use-after-free 버그 가능성이 있는 취약점이 최소 50개 발견되었다는건 놀랍다.
