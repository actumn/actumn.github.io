---
layout: posts
title:  "탄막슈팅게임의 DQN"
date:   2018-11-12
categories: project
---
- [repo](https://github.com/actumn/deep-learning-bullet-hell-environment)  
  
![dl-bullet-hell-env](https://user-images.githubusercontent.com/12994698/48325149-9334ae00-e677-11e8-92f6-40aa290d591a.PNG)  
딥러닝 기초를 익히고 어쩌다보니 재밌는 레포를 발견  
[Sacred curry shooting](https://www.pygame.org/project/937)이라는 pygame으로 구현한 탄막슈팅게임을 바탕으로 환경을 구성해서 RL을 구현했다는 듯  
원본은 tensorflow - keras 코드인데, 맘에 안들어서 pytorch로 포팅(딥러닝 모델도 조금 변경했다)  
action은 2개. 0 - 왼쪽 이동, 1 - 오른쪽 이동  
좋은 사례가 될 것 같아서 해봤는데 ..  
분명히 피하긴 피하는데 학습이 잘돼서 피하는건지 운이 좋아서 피하는건지 모르겠다.  
어쩔땐 그냥 죽으러 가버린다  
  
학습 상태를 알기 위해선 지표 그래프등을 구성해서 봐야할 것 같은데 어떻게 해야할지 잘 모르겠다.  
학습 모델을 개선을 하기 위해선 딥러닝 논문을 찾아보고 여러 사례를 찾아봐야 할듯  
  
재밌었고 좋은 공부가 됐음  
Touhou MoF(동방풍신록)의 DQN을 구현할 땐 (위아래 이동 등)action을 늘리고 모델 구현을 이것저것 바꿔봐야겠다.  
