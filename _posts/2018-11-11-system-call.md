---
layout: posts
title:  "System Call"
date:   2018-11-11
categories: cs
---
biscuit 논문을 보면서 든 궁금점.  
POSIX-subset 시스템 콜 인터페이스를 만족하는 Biscuit은 nginx, redis를 소스 수정 없이 실행시킬 수 있어서 이 부분이 궁금했다.  
lib.c 의 accept등의 시스템콜 함수를 똑같이 수행할 수 있었다는 것인데  

- http://duksoo.tistory.com/m/entry/System-call-%EB%93%B1%EB%A1%9D-%EC%88%9C%EC%84%9C  

시스템 콜의 처리는 eax에 시스템콜 고유 번호를 넣고 0x80 소프트웨어 인터럽트를 발생시킨다는 것.  
input, return이 동일하도록 이 시스템콜 인터페이스를 만족시키면 다른 OS에서도 호환이 가능하겠다.


[리눅스 시스템 콜 테이블](https://thevivekpandey.github.io/posts/2017-09-25-linux-system-calls.html)  
이는 biscuit에도 _sysbounds로 구현되어 있었다.  
