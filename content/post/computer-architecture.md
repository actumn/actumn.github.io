---
title: "Computer Architecture"
date: 2020-06-20T13:59:16+09:00
categories:
- Computer Science
tags:
- lecture
- computer science
- computer network
keywords:
- tech
draft: true
math: true
#thumbnailImage: //example.com/image.jpg
---

건국대학교 컴퓨터 구조 강의노트  
Computer Organization and Design: The Hardware / Software Interface
<!--more-->

# Ch0. Introduction

# Ch1. Performance Evaluation
- Motivaltion
  - 왜 어떤 하드뒈어가 다른 것보다 성능이 좋지?
  - 어떤 시스템은 퍼포먼스 요소가 하드웨어와 연관되어 있나
  - Machine intstruction set이 어떻게 퍼포먼스에 영향을 주는가.
- Terminology
  - Response time: 1 task 처리하는 데 얼마나 걸리는지
  - Throughput: 단위 시간에 얼마나 많은 Task를 처리하는 가
  - Response time과 throughput에 영향을 주는 것은?
- Relaitvie Perfomance
  - $ Performance = 1 / ExeTime $
  - "X가 Y보다 n배 빠르다"
    $$ Px / Py = Ey / Ex = n $$
  - ex) task 10s on A, 15s on B -> E(B) / E(A) = 15 s / 10 s = 1.5
  - A가 B보다 1.5배 빠르다

- 실행시간 측정
  - Elapsed time (wall clock time, Response time)
    - 전체 response time (여러 측면을 포함해서): Processing, I/O, OS overhead, Idle time
    - 시스템 퍼포먼스를 결정한다.
  - CPU time
    - 순수히 task만 걸린 시간
    - 주어진 job을 처리하는 데 걸린 시간. I/O, scheduling, IDLE 포함 X
    - 유저 CPU time, 시스템 CPU time
    - 다른 프로그램이 CPU와 시스템 성능에 다르게 영향 받는다.
  - CPU Clocking
![IMAGE](/images/kucse-computer-architecture/performance-clocking.png)
    - flip-flop: 1 클락에 변화 계산
    - clock period: clock cycle 시간. 상태를 flip-flop에 저장한다.
      - ex) $250ps = 0.25ns = 250 * 10^{-12}s $
    - clock frequency: 1 클락에 1 digital operation
      - ex) $ 4.0GHz = 4000MHz = 4.0 * 10^9Hz $
    - Notes.
    $$ K = 10^3 $$
    $$ M = 10^6 $$
  - CPU time
  $$ CPUTime = CPUClockCycles * ClockCycleTime \\\ = CPUClockCycles / ClockRate $$
  - 성능 개선
    - 클락 사이클 감소
    - 클락 rate 증가
    - 하드웨어 디자이너는 clock rate vs cycle count의 trade-off 를 고려
  - CPU Time example
    - A computer 2GHz clock, 10 s CPU time
    - Designing Computer B
      - 6s CPU time이 목표
      - Clock을 빨리 할 수는 있지만, 클락 사이클이 1.2배가 된다.
      $$ ClockRateB = ClockCycleB / CPUTimeB \\\\ = 1.2 * ClockCycleA / 6s $$
      $$ ClockCyclesA = CPUTimeA * ClockRateA \\\\ = 10s * 2GHz = 20 * 10^9 $$
      $$ ClockRateB = 1.2 * 20 * 10^{9} / 6s = 24 * 10^ 9 / 6 = 4GHz $$
  - Instruction Count and CPI
    $$ ClockCycles = InstructionCount * CyclesPerInstruction $$
    $$ CPUTime = InstructionCount * CPI * ClockCycleTime \\\\ = InstructionCount * CPI / ClockRate $$
      - 프로그램당 Instruction Count
        - 프로그램, ISA, 컴파일러로 결정됨
      - 평균 CPI
        - CPU 하드웨어로 결정됨
        - Instruction에 따라 CPI가 달라질 수 있다.
        - 평균 CPI는 Instruction mix에 영향받음
      - CPI Example
        - ComputerA: Cycle Time = 250ps, CPI = 2.0
        - ComputerB: Cycle Time = 500ps, CPI = 1.2
        - Same ISA
        - Which is faster? How much?
        $$ CPUTime_A = InstructionCount * CPI_A * CycleTime_A \\\\ = I * 2.0 * 250ps = 1 * 500ps $$
        $$ CPUTime_B = InstructionCount * CPI_B * CycleTime_B \\\\
        = I * 1.2 * 500ps = I * 600ps $$
        $$ CPUTime_B / CPUTime_A = I * 600ps / I * 500ps = 1.2 $$
      - CPI in More Detail
        - Instruction 종류에 따라 Cycle 수가 달라진다.
        $$ \sum_1^n {CPI_i * InstructionCount_i} $$
        - Weihted average CPI
        $$ CPI = ClockCycles / InstructionCount = \sum_i^n {CPI_i * InstructionCount_i / InstructionCount} $$
          - $ InstructionCount_i / InstructionCount = Relative Frequency $, i 타입의 Instruction 비율



