---
layout: posts
title:  "pip install mecab-python3 삽질"
date:   2018-11-28
categories: explore
---
맥북 환경에서 mecab-python3 설치를 하던 도중 발견한 에러
```
pip install mecab-python3


 #include <stdexcept>:
gcc: cannot find stdexcept
```
python 라이브러리를 설치하는데 왠 gcc 에러.  
[본 repo](https://github.com/SamuraiT/mecab-python3)를 클론받아서 이거저거 해보았다.  
아래의 커맨드로 이슈를 재현해 볼 수 있었다.
```
python setup.py bdist_wheel

run bdist_wheel...
run build...
run build_py...
run build_ext...

 #include <stdexcept>:
gcc: cannot find stdexcept
```
build_ext 커맨드에서 해당 이슈가 발생했다.  
요점은 C++로 구현된 mecab을 gcc로 컴파일하고, 파이썬으로 랩핑해야 하는데  
gcc 컴파일이 c모드로 실행되었던 것.  
아래의 커맨드로 이슈를 해결할 수 있었다.
```
CFLAGS='-std=libc++` pip install mecab-python3 
```
