---
title: "Operating System"
date: 2020-06-23T01:22:15+09:00
categories:
- Computer Science
tags:
- lecture
- computer science
- computer network
keywords:
- tech
math: true
draft: true
#thumbnailImage: //example.com/image.jpg
---

건국대학교 운영체제 강의노트  
http://pages.cs.wisc.edu/~remzi/OSTEP/#book-chapters
<!--more-->

# CPU virtualization
## Process
### (1)
- Program vs Process
  - Program
    - Executable 파일
    - instructios, static data
  - Process
    - 실행되고 있는 프로그램
    - Machine stae
      - Memory: instruction and data
      - Register: PC, stack pointer, ...
      - Others: 프로세스가 open한 파일 리스트. (우리가 file을 close하지 않아도, 프로세스가 종료할 때 운영체제가 알아서 close 해준다.)
- API
  - create, destory, wait, miscellaneous control
  - fork()
    - parent, child
    - 실행되는 순서가 다를 수 있다: OS process schedulling policy
    - 따라서 순서를 예측할 수 없다.
  - wait()
    - child가 하나라도 종료되는 걸 기다림.
  - waitpid()
    - 특정 child 프로세스가 종료되는 걸 기다림.
  - exec()
    - 기존에 있던 프로세스를 덮어쓴다.
    - 일반적으로 fork()하고 child에서 exec
- 프로세스가 여러개 -> 컨텍스트 스위칭
  - 동시자원 접근
  - timer, I/O request등 인터럽트가 일어나면 컨텍스트 스위칭 (그리고 OS에서 어떤 작업을 수행한다.)
- 코어는 한정적. 많은 프로세스를 어떻게 효율적으로 처리할 것인가? 
  - Ideal: 동시에 처리
  - Reality: 프로세스가 CPU 선점, 자원 독차지
  - 운영체제는 application 개발자에게 어떻게 Ideal하게 제공할 수 있을지 고민한다.
- Time sharing
  - 많은 가상 CPU가 존재한다고 생각. (실제 CPU는 몇몇개 정도로 한정적)
  - 원하는 만큼 동시에 (councurrent) 프로세스 실행
  - 비용
    - CPU가 공유된다.
    - 이는 효율적인가? (Context switching overhead)
  - 컨텍스트 스위치
    ![IMAGE](/images/kucse-operating-system/process-timesharing.png)
### (2)
- Process State
    ![IMAGE](/images/kucse-operating-system/process-states.png)
  - Running: 프로세서에서 돌고 있다.
  - Ready: 준비되어 있다. CPU가 아직 안꺼내갔다.
  - Blocked (Waiting): not ready.
    - Disk I/O, Network I/O 는 CPU보다 굉장히 느리다.
    - 여기서 데이터가 올때까지 기다린다 -> Process Block
  - example
    ![IMAGE](/images/kucse-operating-system/process-states-ex.png)
  - data strutures: `/include/linux/sched.h`
```c
struct task_struct {
#ifdef CONFIG_THREAD_INFO_IN_TASK
	/*
	 * For reasons of header soup (see current_thread_info()), this
	 * must be the first element of task_struct.
	 */
	struct thread_info		thread_info;
#endif
	/* -1 unrunnable, 0 runnable, >0 stopped: */
	volatile long			state;

	/*
	 * This begins the randomizable portion of task_struct. Only
	 * scheduling-critical items should be added above here.
	 */
	randomized_struct_fields_start

	void				*stack;
#ifdef CONFIG_THREAD_INFO_IN_TASK
	/* Current CPU: */
	unsigned int			cpu;
#endif
	struct mm_struct		*mm;
	struct mm_struct		*active_mm;

	/*
	 * Pointers to the (original) parent process, youngest child, younger sibling,
	 * older sibling, respectively.  (p->father can be replaced with
	 * p->real_parent->pid)
	 */

	/* Real parent process: */
	struct task_struct __rcu	*real_parent;

	/* Recipient of SIGCHLD, wait4() reports: */
	struct task_struct __rcu	*parent;

	/*
	 * Children/sibling form the list of natural children:
	 */
	struct list_head		children;

	/* Filesystem information: */
	struct fs_struct		*fs;

	/* Open file information: */
	struct files_struct		*files;
}
```
- Scheduling queue
  - Run queue
  - Ready queue
  - Waiting queue
  - 운영체제 입장에서 어떤 process가 어떤 state에 있는지 빨리 찾아야할 필요가 있다.
    - ex) 한 프로세스가 너무 길게 run -> (run <--> Ready)
  
## Limited Direct Execution
프로세스에게 CPU 자원을 주고, "하고 싶은 대로 다 해봐라"

| OS                          | Program  |
|---------------------------|---|
| Create entry for process list |   |
| Allocate memory for program |   |
| Load program into memory |   |
| Set up stack with argc/argv |   |
| Clear registers |   |
| Execute call main() |   |
|  | Run main()  |
|  | Execute return from main()  |
| Free memory of process |   |
| Remove from process list |   |

프로그램이 돌아가는 중에 OS는 컨트롤 할 여지가 없다?  
OS도 결국은 실행코드 집합. CPU 자원이 있어야한다.  
- 프로그램이 CPU를 갖고 있으므로 
  - 감시도 못하고
  - Time sharing / Process policy / Context switch 하기 어렵다.
### Problem 1: Restricted Operations (Previleged opeartions)
- How to perform restricted operations?
  - Restricted operations (previleged operations)
    - Issuing an **I/O request** to a disk
    - **Gaining access** to more system resources
    - CPU, memory 등 시스템 자원은 항상 모자란 자원. 유저 프로세스가 요청한 대로 다 제공 불가.
    - 요청에 대해 얼만큼 허락할 것인지.
  - Application은 제한된 operation을.
    - 다만 프로세스에게 완전한 제어권은 X
- Processor modes: CPU가 제공하는 기능. Intel은 4가지 모드 제공, 운영체제는 User, Kernel 쓴다.
  - User mode: Restricted operation 사용 불가.
    - 유저 모드 코드는 할 수 있는게 제한된다.
    - 제한된 operation은 프로세서 exception을 일으킨다.
  - Kernel model
    - 코드가 뭐든지 할 수 있다.
    - OS는 커널모드로 동작.
- System call
  - Restricted opeartion은 유저모드에선 불가. 하지만 필요하다 -> 시스템 콜.
  - Trap instruction
    - 커널 진입 (시스템 콜 호출시)
    - previleged level을 kernel 모드로.
    - previleged operation 수행
  - Return-from-trap instruction
    - 시스템 콜이 끝나고 호출되는 instruction
    - 커널 모드 -> 유저 모드
    - previleged level을 kernel 모드에서 다시 user mode로
  - 유저 프로세스는 Retricted opeartion을 쓰기 위해 시스템 콜을 호출해야 한다.
  - Trap -> (instruction) -> Return-from-trap
- Restricted Operation을 맘대로 못쓰게 하기 위해 프로세서 모드. 
  - 우리는 그냥 시스템 콜을 부르면 쉽게 커널 모드 진입. 허락되지 않은 일도 맘대로 할 수 있지 않을까.
  - => 시스템 콜을 호출한 프로세스는 jump할 address를 지정할 수 없다. 
    - 정해져 있는 시스템 콜을 호출하게 되어 있다.
    - 제공되는 Restricted operation이 대단히 제한적이다.
  - Trap Table
    - Trap instruction이 호출되었을 때 실행될 코드들, 즉 trap handler가 저장된 table
    - 시스템 콜 number
    - Trap instruction을 해도 특정 address 접근이 아니라, 미리 정해져 있는 number만 요청
    - 운영체제가 미리 정의해 놓은 hadnler만 호출 => 운영체제 검사 => 통과, OS 코드 실행 
      - 여러 권한 검사
      - 이 프로세스가 이 코드를 실행해도 되는가
      - 이 trap number로 특정 호출을 해도 되는지.
    ![IMAGE](/images/kucse-operating-system/exec-trap-table.png)
    

| OS | Hardware | Program |
|-|-|-|
| Initialize trap table |  |  | 
|  | Remember address of syscall handler |  |
| Create entry for process list |  |  |
| Allocate memory for program |  |  |
| Load program into memory |  |  |
| Set up user stack with argc/argv |  |  |
| Fill kernel stack with regs/PC |  |  |
| **Return-from-trap** |  |  |
|  | Restore regs from kernel stack |  |
|  | **Move to user mode** |  |
|  | Jump to main |  |
|  |  | Run main() |
|  |  | ... |
|  |  | Call system call |
|  |  | **Trap** into OS |
|  | Save regs/PC to kernel stack |  |
|  | Move to kernel mode |  |
|  | Jump to trap handler |  |
| Handle trap |  |  |
| **Return-from-trap** |  |  |
|  | Restore regs from kernel stack |  |
|  | **Move to user mode** |  |
|  | Jump to PC after trap |  |
|  |  | Return from main() |
|  |  | **Trap** (via exit()) |
| Free memory of process |  |  |
| Remove from process list |  |  |


### Problem 2: Switching Between Processes
- How to regain control of the CPU?
  - CPU에서 프로세스가 돌고 있다면, OS는 돌고 있지 않다.
  - OS가 돌고 있는게 아니라면, 이걸 어떻게 하지? (System call, Context Switching)
- Cooperative Approach: Wait for system calls
  - 오래 실행되는 프로세스는 "주기적으로 CPU를 포기할 것"으로 추정
    - 대부분의 프로세스는 CPU 제어권을 OS에게 꽤나 빈번히 넘긴다.
    - 시스템 콜 내부에 스케쥴링 코드 => time sharing
  - illegal operation 할 때도 제어권을 넘긴다
    - Dividing by zero
    - Segmentation fault
    - 응용 프로그램에서 시스템콜을 부르지 않는다면?
  - 운영체제 개발자는 Application 개발자를 믿지 않는다.
- Non-Cooperative Approach: The OS takes control
  - Timer interrupt
    - CPU에서 소프트웨어적으로 프로그래밍 가능한 timer를 제공. 
    - timer device는 주기적으로(1ms?) interrupt를 발생시킨다.
    - 인터럽트 발생시 현재 프로세스는 머추고 사전에 설정된 인터럽트 핸들러가 동작한다.
- Context Switch
  - Saving and restoring context
    - 몇몇 레지스터 값 저장
    - 곧 실행될 프로세스 값 restore (kernel stack에서 꺼내서, kernel stack => 실행 프로세스)
    - Return-from-trap instruction이 실해되면 system은 또 다른 프로세스 실행 재개.

### (3)
| OS | Hardware | Program |
|-|-|-|
| Initialize trap table |  |  |
|  | Remember address of syscall handler |  |
|  | Remember address of timer handler |  |
| **Start interrupt timer** |  |  |
|  | Start timer(interrupt CPU in X ms ) |  |
|  |  | Process A |
|  | **Timer interrupt** |  |
|  | Save regs(A) to k-stack(A) |  |
|  | Move to kernel mode |  |
|  | Jump to trap handler |  |
| Handle the trap |  |  |
| Call switch() routine |  |  |
|   save regs(A) to k-stack(A)* |  |  |
|   restore regs(B) from k-stack(B)* |  |  |
|   switch to k-stack(B) |  |  |
| Return-from-trap (into B) |  |  |
|  | Restore regs(B) from k-stack(B) |  |
|  | Move to user mode |  |
|  | Jump to B’s PC |  |
|  |  | Process B |
|  |  |  |

## CPU Scheduling
### (1)
- Workload Assumptions (Workload 가정)
  - Each jobs (thread/process) runs for the same amount of time
  - All job, arrive(실행할 수 있는 상태) at the smae time (ready queue 진입)
  - 일단 시작하면, 각 job은 자원을 양보하지 않는다.
  - 모든 job은 CPU만 사용. (I/O 가 없다. CPU를 양보하지 앟는다.)
  - 각 job의 run-tim을 알고 있다.
  - 다 비현실적 가정.. 이러한 가정에서 출발해서 조금씩 스케쥴링을 개선한다.
- Scheduling Metrics
$$ T_{turnaround} = T_{completion} - T_{arrival} $$
  - 생선된 시간부터 종료된 시간까지 얼마나 걸렸는가.
  - CPU 스케쥴링 평가 지표
- FIFO (First In, First Out): First come, First served (FCFS)
![IMAGE](/images/kucse-operating-system/scheduling-fifo.png)
  - 간단하고 쉽다.
  - Average turnaround time: $$(10+20+30)/3 = 20$4
  - 실행시간이 같다는 가정을 지워보자.
![IMAGE](/images/kucse-operating-system/scheduling-fifo-problem.png)
    - Average turnaround time: $$(100+110+120)/3 = 110$$
    - 오래걸리는 process가 앞에 있을 때 비횽ㄹ적.
    - 전체 성능이 안좋아진다: Convoy effect
- SJF (Shortest Job First)
![IMAGE](/images/kucse-operating-system/scheduling-sjf.png)
  - 실행시간이 빠른 애들부터 먼저 하자.
  - Average turnaround time: $$(10 + 20 + 120) / 3 = 50$$
  - 모든 job이 동시에 올 때 최적
  - 동시에 도착한다는 가정을 지워보자.
![IMAGE](/images/kucse-operating-system/scheduling-sjf-problem.png)
    - Average turnaround time: $$(100 + (110 -10) + (120 - 10) / 3 = 103.33$$
    - SJF에서도 Convoy effect 발생
### (2)
- Preemptive Scheduler
  - Non-preemptive scheduler
    - 일단 프로세스가 시작하면 다음 스케쥴링은 프로세스가 끝난 뒤 도착
    - 현재보다는 과거에 했었던 방법. 계산 위주의 job.
      - 한번 실행이 되면 runtime state를 모른다.
      - 실행이 끝날 때 까지 wait => batched job
  - Preemptive scheduler
    - 어떤 프로세스가 선점하던 자원을 뺏어서 다른 프로세스에게 줄 수 있다.
    - 실행하던 프로세스를 멈추고, CPU 자원을 다른 프로세스에게 줘서 다른 프로세스가 실행하도록 자원을 선점하게 하는 scheduler
    - Context switch
    - 현재 운영체제 대부분 preemptive scheduler
- STCF (Shortest Time-to-Completion First): Preemptive Shortest Job First (PSJF)
![IMAGE](/images/kucse-operating-system/scheduling-stcf.png)
  - Average turnaround time: $$ 120 + (20-10) + (30-10) / 3 = 50 $$
- Scheduling Metrics
  - Turnaround time
  - Response time: job이 도착한 시점부터 running state까지 간 시간. $$ T_{response} = T_{firstrun} - T_{arrival} $$
  - PSJF에서 average response time은 3.33
![IMAGE](/images/kucse-operating-system/scheduling-stcf-problem.png)
    - turnaround time은 괜찮은데, response time과 interactivity가 별로다.

- RR (Round-Robin)
![IMAGE](/images/kucse-operating-system/scheduling-rr.png)
  - = Time slicing
  - 스케쥴링이 Time slice가 끝날때마다. (STCF는 프로세스가 새로 도착 / 혹은 종료될 때 뿐)
  - Time slice: 2라 했을때
    - Response time = $ (0 + 2 + 4) / 3 = 2$, turnaround time이 조금 구리다.
  - 실행시간이 아무리 길어져도 response time은 동일. 
  - rseponse time vs turnaround time trade-off. slice가 짧으면 컨텍스트 스위치 overhead
  - 어떤 종류의 application을 돌리지에 따라 다르다. 
  - 현재 스케쥴러는 모두 RR기반. 
  - 보통 사용자 device와 interactive 케이스가 많다. 일반적으로 RR이 성능이 좋다.
  - 운영체제마다 time slice가 상수이기도, 동적으로 바꾸기도 한다.
- 프로세스가 CPU만 사용한다는 가정을 지워보자.
![IMAGE](/images/kucse-operating-system/scheduling-IO.png)
  - 프로스세스가 blocked.
  - 디스크는 CPU에 비해 굉장히 느리다.
  - 인터럽트 - 시스템 콜.
  - 놀 수 있는 CPU 자원을 다른 프로세스에게 양보.
- 마지막으로 실제 실행시간을 알고 있다는 가정을 지워보자.
  - SJFT, STCF와 같은 접근은 불가능하다.
  - 남는건 FIFO, RR
  - 선택지는 RR밖에 남지 않는다.
    - 현실적으로 타협가능한 알고리즘.

## Multi-level Feedback Queue
### (1)
- Crux of the Problem
  - 아래 두개를 만족하는 스케쥴러는 어떻게 설계할까
    - interactive job에 최소한의 response time
    - turnaround time 최소화
  - 실행시간을 우리는 모른다.
- Multi-level Feedback Queue (MLFQ): Multiple queues
  - 각자 다른 priority level
  - 한 순간에 한 job은 해당하는 한 priority level queue에 들어있다.
![IMAGE](/images/kucse-operating-system/mlfq-example.png)
  - Basic rules
    - Rule 1: Priority(A) > Priority(B): A runs
    - Rule 2: Priority(A) = Priority(B): A & B run in RR
  - 위 그림에선 A, B가 다 끝나거나, I/O Blocked가 되야 C, D가 실행될 수 있겠다.
- 어떻게 priority를 주지?
  - Fixed priority to each job (사용자 명시)
    - 자기꺼가 가장 높은 우선순위 하겠지.
    - 모든 프로세스가 가장 높은 우선순위 큐에만???
    - OS는 application 개발자를 믿지 않는다.
  - 동적으로 바꾼다 : 최근에 어떤 행동을 했는지 보고 이후 행동을 예측
    - job이 CPU를 양보 (I/O blocked) -> 높은 우선순위, CPU를 오래 쓰지 않으니까.
    - job이 CPU를 오랜시간 점유 => MLFQ는 우선순위를 내린다.
- How to Change Priority
  - Workload
    - I/O-bound jobs
      - Interactive, and short-running: 사용자 입력, I/O data send/recv
    - CPU-bound jobs
      - 계산 집중적.

  - Priority adjustment algorithm
    - Rule3
      - job이 시스템에 들어오면, 높은 priority
    - Rule4
      - a: time slice를 줄 때 마다 CPU를 소진하면. 낮은 priority
      - b: time slice가 끝나기 전에 CPU 자원을 양보한다 => priority 유지
  
### (2)
- Problems
  - Starvation: I/O bound job이 많으면? Interactive job이 많으면?
    - 모든 time slice를 I/O bound job이 갖는다. CPU bound job은 얻지 못한다.
  - Gaming the scheduler: timer slice가 끝나기 전에 I/O 요청하면?
  - Changing behavior: CPU bound -> I/O bound
- Priority Boost
  - 주기적으로 시스템의 모든 job의 priority를 높은 쪽으로
    - Rule5: S만큼 시간이 지난 후 모든 job을 다시 가장 높은 queue로
    - starvation과 changing-behavior를 해결 할 수 있다.
- Starvation 문제
![IMAGE](/images/kucse-operating-system/mlfq-starvation.png)
  - Priority boost로 해결
![IMAGE](/images/kucse-operating-system/mlfq-starvation-solve.png)
- Gaming the scheduler 문제
![IMAGE](/images/kucse-operating-system/mlfq-gaming.png)
  - Rule 4를 재정의하자.
    - 누적된 CPU 사용량이 time slice만큼 되는가?
    - 누적된 사용량이 time slice만큼 된다면 priority를 낮추자
![IMAGE](/images/kucse-operating-system/mlfq-gaming-solve.png)
- Changing bevaior 문제
  - Priority boost
    - 우선순위가 높은 큐에 머무를 기회가 많아진다.

## Multiprocessor
### (1)
![IMAGE](/images/kucse-operating-system/multiprocessor.png)
- SMP (Sementric Multi processing)
  - System Bus, Memory Bus에 트래픽이 많아진다.
  - 코어가 많아지고, 프로세스와 쓰레드가 많아지므로
- NUMA
  - CPU 패키지마다 다른 메모리 액세스
    - laptop, desktop은 보통 CPU 패키지 1개임 ..
  - 다른 CPU 패키지에서 쓰는 메모리를 접근할 수 있다.
    - malloc 같은 api는 운영체제가 어느 cpu에 할지 결정.
  - cache는 원본의 복사본.
    - consistency 문제가 있을 수 있으니 해당 프로토콜이 존재한다. (MESI)
- Single-Queue scheduling: Single-queue multiprocessor scheduling (SQMS)
![IMAGE](/images/kucse-operating-system/multiprocessor-sqms.png)
  - CPU 가 여러개여도 Ready Queue는 1개.
    - 지금까지의 CPU 스케쥴링은 그대로 쓸 수 있다.
  - n개의 job을 꺼내올 수 있다. (n = CPUs)
  - A가 0->1->2->3 ... Cache affinity(캐시 친화도)가 구리다.
  - (CPU마다 클락이 다르다 등의 이유로) time slice가 다를 수 있다. (끝나는 시점이 다를 수 있다.)
    - queue에 접근할 때 동기화 필요.
    - critical section이 생기므로 공유되는 메모리 영역, sequential 할 수 밖에 없다.
    - scalability가 구려진다.
  - Queue 하나가 간단할 수 있지만 여러 단점들이 존재한다.
- Multi-Queue Scheduling: Multi-queue multiprocessor scheduling (MQMS)
![IMAGE](/images/kucse-operating-system/multiprocessor-mqms.png)
  - CPU마다 Ready queue를 따로 두자.
  - job이 시스템에 arrive 할때, 하나의 queue에만 존재하자.
    - 어떤 큐에 둘것인가? random, shorter queue, ...
  - 독립적으로 Ready queue에 접근, 스케쥴링
  - 하나의 큐 - 하나의 CPU. 따라서 동기화가 필요 없다.
    - 1 job은 계속 같은 CPU에서 돈다. locality도 좋다.
  - Load imbalance 문제
![IMAGE](/images/kucse-operating-system/multiprocessor-mqms-problem.png)
    - 어떻게 하지?
      - Migration : job 을 다른 큐로 옮기자. 누가? 언제?
      - Work stealing
        - CPU가 한가해지면 다른 큐에서 job을 가져온다. 밸런스를 맞춰보자.
![IMAGE](/images/kucse-operating-system/multiprocessor-mqms-steal.png)
        - 일반적으로 메모리 bound가 적은 놈을 가져온다.
      
### (2)
- Linux CPU schedulers
  - Completely Fair Scheduler (CFS)
![IMAGE](/images/kucse-operating-system/multiprocessor-cfs.png)
    - Red-Black tree
    - default (SCHED_NORMAL)
    - weighted fair scheduling
    - 프로세스가 얼마나 CPU를 점유하고 이ㅣㅆ었는지,
    - 가장 적게 점유한 애가 최하단 왼쪽, 얘를 우선으로 하려고 노력한다.
  - Real-Time Scheduler
![IMAGE](/images/kucse-operating-system/multiprocessor-rts.png)
    - Multilevel Feedback Queue와 유사.
    - SCHED_FIFO, SCHED_RR
    - Priority-based scheduling
      - priority가 고정되어 있다. (동적으로 변하지 않음). 우선순위 조정 필요
    - sched_setattr
  - Deadline Scheduler
![IMAGE](/images/kucse-operating-system/multiprocessor-deadlinescheduler.png)
    - SCHED_DEADLINE
    - EOF(Earliest Deadline First)-like scheduler
    - Deadline이 제일 급한 애부터 실행시켜라
    - Period(주기)마다 실행하는데, 정해진 Execution time 만큼 실행한다. 
    - sched_setattr
- Completely Fair Scheduler (CFS)
  - vruntime
    - red-black tree에 얼만큼 job이 실행됐는지 저장된다. virtual runtime 기준.
    - 각각의 job 마다 nice value가 있어서 runtime에 따라서 weight값을 부여
    - nice: -20 ~ 19
      - nice가 높으면 CPU 자원을 적게 사용하도록, 낮으면 CPU 자원을 많이 사용하도록 스케쥴러가 동작.
      $$ vruntime = vruntime + DeltaExec * Weight_0 / Weight_p $$
    - DeltaExec: 최근 실행시간
- `/proc/<pid>/sched`
  - priority
    - prio = nice + 120
      - CFS: 100 ~ 139
      - 0 ~ 99 는 real-time scheduler 를 위해 reserved.
  - renice
    - 막바꾸면 안된다.
    - -1~-20: root 만 rksmd
- Multiprocessor Scheduling in CFs
  - Load metric
    - Load of thread: 각 쓰레드가 단위 시간당 CPU를 사용한 시간 평균. priority, weighted 고려
    - Load of core: loads of the threads 합
  - 코어 별 로드를 균일하게 맞추고자 한다. 4ms마다.
  - Imbalance라고 판단되면 고려할 것
    - Cache.. 코어마다 다른 캐시
    - NUMA... CPU 패키지별로 메모리, 버퍼 할당.
      - CPU 패키지별로 load diff가 25% 보다 작으면 migration, load balancing 하지 않음.
  