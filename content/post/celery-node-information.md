---
title: "celery.node 소개"
date: 2020-03-26T02:29:57+09:00
categories:
- celery.node
tags:
- node.js
- typescript
- opensource
keywords:
- tech
thumbnailImagePosition: left
thumbnailImage: /images/celery-node-logo.png
---
celery.node, 저의 첫 오픈소스 프로젝트입니다.  
repository: https://github.com/actumn/celery.node
<!--more-->
![IMAGE](/images/celery-node-logo-word.png)

이름에서 유츄해볼 수 있듯이 [celery 프로젝트](https://github.com/celery/celery)를 node.js로 구현한 프로젝트입니다.

사실 node.js로 구현한 사례는 이미 [node-celery](https://github.com/mher/node-celery)가 있습니다만  
celery client는 구현할 수 있는 반면 celery worker는 지원하지 않으며  
더 이상 관리가 안되고 있는 등의 문제가 있습니다.

이전에 프로젝트를 할 때 python과 node.js를 사용하는 마이크로서비스 아키텍쳐를 위해서  
celery를 쓰고 싶었는데 node.js celery worker를 지원하는 프로젝트가 없어서 아쉬웠습니다.  

그래서 제가 직접 celery 코드를 하나하나 열심히 뜯어보면서 만들었습니다.

처음에는 가치없는 프로젝트인게 아닌가 싶기도 하고  
한다고 누가 알아봐줄까 싶은 순간도 있었습니다만

그래도 [2019 공개 SW 개발자대회](https://www.oss.kr/dev_competition_notice/show/43272fe9-cdb3-40b6-b9c3-bd495850b122)에 출품해서 [수상](https://www.oss.kr/index.php/dev_competition_activities/show/18e47132-10ac-4e1c-96e4-9077dd0757bf)을 하기도 하고  
가끔씩 새로운 사람이 스타를 눌러주기도 하고  
기능을 제안해주기도 하면서  
"이 프로젝트가 분명히 가치가 있구나" 라는 믿음이 생기기 시작했습니다.

그래서 본격적으로 제대로 홍보도 하고 자랑도 해볼까 합니다.  

자!  
이제 celery를 이용해서 간단하게 python, go와 통합할 수 있어요.
![IMAGE](/images/celery.node-concept-image.png)


celery.node 소개 비디오 (Youtube)
[![Video Label](https://img.youtube.com/vi/Fz8drsdoiA0/0.jpg)](https://www.youtube.com/watch?v=Fz8drsdoiA0)


