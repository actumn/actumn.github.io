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
```
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
  
# Concurrency
## Threads
### (1)
- Multi-threaded
  - Multi process와 유사한 성격
    - 자신만의 Program counter, 레지스터
    - 스레드마다 스택
  - 한 프로스세내 스레드들은 address space 공유
  - Context switch
    - TCP (Thraed Control Block), 리눅스에선 PCB, TCP 구분이 없다.
    - Address space에 해당하는 부분은 그대로 둔다. (CR3 레지스터가 바뀌지 않는다.)
![IMAGE](/images/kucse-operating-system/threads.png)
- Why use threads?
  - Parallelism, 병렬성
    - Multiple core CPU가 계속 등장하고 있다.
  - Avoiding blocking
    - Slow I/O
    - main thread만 있고, I/O block되면 다른 일을 못한다. 
    - 다른 thread가 있고 다른 종류의 일을 실행할 수 있다면 성능 향상
  - 많은 서버 application이 multi-threaded 다.
- Thread creation
  - pthread_create
  - pthread_join: watis for the threads to finish
  - Nondeterministic: 실행순서는 예측할 수 없음
### (2)
- Race condition
![IMAGE](/images/kucse-operating-system/threads-race.png)
- Critical section
  - 공유되는 자원, 공유되는 변수들
  - 반드시 1개의 thread 에서만 접근.
- Mutual exclusion
  - critical section에서 1개의 thread만 접근함을 보장
- Atomicity
  - critical section 구간 자체를 interrupt가 발생하지 않도록 만든다.
  - 어떤 instruction이 중간에 interrupt 발생시 실행을 안하게, 또는 발생했음에도 instruction이 끝날때 까지 interrupt를 처리하지 않도록
- How to support synchronization
  - 하드웨어 제공 (atomic instructions)
    - Atomic memory add: 대부분 CPU 제공.
    - 자료구조는? Atomic update of B-Tree? CPU가 제공하기 어렵다. 회로도 복잡하고, Atomic 오버헤드. 
  - OS가 Atomic instruction을 기반으로 좀 더 일반적인 동기화 primitives (system call) 를 제공
- Mutex
  - pthread_mutex_lock
  - pthread_mutex_unlock
  - pthread_mutex_trylock
  - pthread_mutex_timedlock
  - Initialization
    - Static
      ```
      pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
      ```
    - Dynamic
      ```
      int rc = pthread_mutex_init(&lock, NULL);
      ```
  - Destory
    ```
    pthread_mutex_destroy()
    ```
- Condition varibales
  - pthread_cond_wait
  - pthread_cond_signal
  - 어떤 상태가 다른 CPU에 의해 바뀌었음을 인지하고, 바뀌었을 때 나에게 signal을 보내서 꺠워줄 수 있는 역할.
  - Synchronizing two threads
    ```
    while (ready == 0) 
        ; // spin
    ```
    ```
    ready = 1;
    ```
    - spin lock, busy wait: CPU 낭비. 어떤 상태가 되기까지 기다리겠다.
    - CPU cycle 낭비. 1 core CPU라면 thread 1 점유. 상태가 바뀌지도 않을텐데.
    - 서로 다른 core라면
      - 다른 CPU cache에 ready 값이 들어간다. 값 update를 알아차리기 위해 CPU 종류에 따른 cache coherence 알고리즘 의존적
      - compiler optimization은?
    - Condtition variable 활용
    ```
    pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
    pthread_cond_t cond = PTHREAD_COND_INITIALIZER;
    pthread_mutex_lock(&lock);
    while (ready == 0)
    pthread_cond_wait(&cond, &lock);
    pthread_mutex_unlock(&lock);
    ```
    ```
    Pthread_mutex_lock(&lock);
    ready = 1;
    Pthread_cond_signal(&cond);
    Pthread_mutex_unlock(&lock);
    ````
    - Compile
    ```bash
    gcc -o main main.c -Wall -pthread
    ```
## Locks
- Pthread Locks
```
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
...
pthread_mutex_lock(&lock);
counter = counter + 1; // critical section
pthread_mutex_unlock(&lock);
```
- lock을 어떻게 구현하지?
  - 하드웨어 support는?
  - OS의 역할은?
- Evaluation Locks
  - Mutual exclusion (제일 중요)
    - critical section 내에 1개의 thread만 진입
  - Fairness
    - lock 취득에 있어 fair한 순서보장
  - Performance
    - lock을 함에 따라 성능 하락이 있진 않은지
    - thread 수, CPU 수에 따라 성능의 impact는?
- Controlling Interrupts
```
void lock() {
  DisableInterrupts();
}
void unlock() {
  EnableInterrupts();
}
```
  - 프로세스 스케쥴링에 있어 가장 중요한건 timer interrupt.
  - Timer interrupt를 disable하면 timer interrupt가 발생하지 않음
    - scheduling X, context switch X, race condition X
  - 간단하지만 많은 단점
    - privileged operation
    - multiprocessor라면 통하지 않는다. (호출한 CPU만 Interrupt disable)
    - interrupt 발생 가능성을 잃을 수 있다. 더 처리해야하는데, 덜 처리할 수 있다. 성능 저하 가능성
    - 결정적으로 이런 시스템콜은 없다.
- Spin locks with Load/Stores
```
typedef struct __lock_t { int flag; } lock_t;
void init(lock_t *mutex) {
    // 0 -> lock is available, 1 -> held
    mutex->flag = 0;
}
void lock(lock_t *mutex) {
    while (mutex->flag == 1)
        ;
    mutex->flag = 1;
}
void unlock(lock_t *mutex) {
    mutex->flag = 0;
}
```
| Thread 1 | Thread 2 |
|---|---|
| Call lock() |  |
| while (flag == 1)? |  |
| Context switch |  |
|  | Call lock() |
|  | while (flag == 1)? |
|  | flag = 1; |
|  | Context switch |
| flag = 1; |  |
  - No mutual exclusion
  - Performance Problem
    - waste time waiting

### (2)
- Spin locks
  - Test-and-set atomic instruction
    - 슈도코드. 실제로는 instruction 임
    ```
    int TestAndSet(int *old_ptr, int new) {
        int old = *old_ptr; // fetch old value at old_ptr
        *old_ptr = new; // store ’new’ into old_ptr
        return old; // return the old value
    }
    ``` 
    - 어떠한 preemption이 없음을 하드웨어가 보장
- Spin locks with Test-and-Set
```
typedef struct __lock_t { int flag; } lock_t;
void init(lock_t *lock) {
    lock->flag = 0;
}
void lock(lock_t *lock) {
    while (TestAndSet(&lock->flag, 1) == 1);
}
void unlock(lock_t *mutex) {
    mutex->flag = 0;
}
```
  - TestAndSet(0, 1) -> while 문을 빠져나감. lock 취득
  - TestAndSet(1, 1) -> 계속 while.
  - 문제점
    - Not fair
    - 성능. 
      - single core -> thread가 많아지면 더 심각해짐. 
      - unlcok 되기전에 ready queue에 들어가면?
- Compare-and-swap atomic instruction
```
int CompareAndSwap(int *ptr, int expected, int new) {
    int actual = *ptr;
    if (actual == expected)
        *ptr = new;
    return actual;
}
```
```
void lock(lock_t *lock) {
    while (CompareAndSwap(&lock->flag, 0, 1) == 1);
}
```
  - CompareAndSwap(0, 0, 1) -> while문 빠져나감. lock 취득
  - CompareAndSwap(1, 0, 1) -> 계속 while 대기
  - 달라진 건 없다. 
    - Mutex O
    - Fair X
    - Performance X
- Ticket locks: 티켓을 나눠주고 돌려받는 대로.
  - fetch-and-add atomic
    ```
    int FetchAndAdd(int *ptr) {
        int old = *ptr;
        *ptr = old + 1;
        return old;
    }
    ```
    ```
    typedef struct __lock_t {
        int ticket;
        int turn;
    } lock_t;
    void lock_init(lock_t *lock) {
        lock->ticket = 0;
        lock->turn = 0;
    }
    void lock(lock_t *lock) {
        int myturn = FetchAndAdd(&lock->ticket);
        while (lock->turn != myturn);
    }
    void unlock(lock_t *lock) {
        lock->turn = lock->turn + 1;
    }
    ```
    - Fair O
- Hardware support
  - Too much spinning
    - 성능관점 문제가 남아있다.
  - A simple approach
    - yield()
      - CPU 자원을 포기하겠다. 다른 thread를 실행시켜라.
      - 다만 scheduler는 다시 실행시킬 수 있다. vruntime등의 이유로
    - 성능 관점에서 여전히 좋지 않다.
    ```
    void lock(lock_t *lock) {
        while (TestAndSet(&lock->flag, 1) == 1)
        yield();
    }
    ```
      - 많은 thread가 round-robin 기반으로 schedule 된다면? vruntime 계산해봤더니 결국 그대로.
      - spinning은 해결되지 않는다. 결국 같은 thread들이 반복해서 yield. => OS가 중요해진다.
### (3)
- OS Support
  - Sleeping instead of spinning
    - Solaris
      - park(): 호출 thread를 재운다.
      - unpark(threadID): 해당 thread를 꺠운다.
    - Linux
      - futex: fast user-level mutex
      - futex_wait(address, expected)
      - futex_wake(address): 잚든 thread를 꺠운다.
      - address: lock variable
      - queue 기반으로 재우고 꺠운다.
- Lock with queue
  - queue: lock을 기다리는 queue
```
typedef struct __lock_t {
    int flag; // lock
    int guard; // spin-lock around the flag and
                   // queue manipulations
    queue_t *q;
} lock_t;
void lock_init(lock_t *m) {
    m->flag = 0;
    m->guard = 0;
    queue_init(m->q);
}
```
```
void lock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1);
    if (m->flag == 0) {
        m->flag = 1; // lock is acquired
        m->guard = 0;
    } else {
        queue_add(m->q, gettid());
        m->guard = 0;
        park(); // ** wakeup/waiting race **
    }
}
void unlock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1);
    if (queue_empty(m->q))
        m->flag = 0;
    else
        unpark(queue_remove(m->q));
    m->guard = 0;
}
```
  - TestAndSet ~ m->guard = 0 까지 atomic block. 
  - unpark() => park() 하면?
  - lock 호출 주체가 park()하기 전에, lock 을 가지고 있던 놈이 unpark()
- 개선안.
```
void lock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1);
    if (m->flag == 0) {
        m->flag = 1; // lock is acquired
        m->guard = 0;
    } else {
        queue_add(m->q, gettid());
        setpark(); // another thread calls unpark before
        m->guard = 0; // park is actually called, the
        park(); // subsequent park returns immediately
    }
}
void unlock(lock_t *m) {
    while (TestAndSet(&m->guard, 1) == 1);
    if (queue_empty(m->q))
        m->flag = 0;
    else
        unpark(queue_remove(m->q));
    m->guard = 0;
}
```
  - setpark() : unpark 호출이력 확인. 있는 경우 park()는 재우지 않음.

## Lock-based Concurrnet Data Structures
여러 개의 쓰레드가 접근하는 공유 자료구조  
race condition을 해결할 방법들
### (1)
- Correctness
  - race condition을 발생시키지 않도록 lock
- Concurrency
  - lock을 걸게 되면 performance 하락 (병렬성 저하)
  - 최대한 효율적으로 쓸 방법은?

- Concurrent Counters
```
typedef struct __counter_t {
    int value;
    pthread_mutex_t lock;
} counter_t;
void init(counter_t *c) {
    c->value = 0;
    pthread_mutex_init(&c->lock, NULL);
}
void increment(counter_t *c) {
    pthread_mutex_lock(&c->lock);
    c->value++;
    pthread_mutex_unlock(&c->lock);
}
void decrement(counter_t *c) {
    pthread_mutex_lock(&c->lock);
    c->value--;
    pthread_mutex_unlock(&c->lock);
}
int get(counter_t *c) {
    pthread_mutex_lock(&c->lock);
    int rc = c->value;
    pthread_mutex_unlock(&c->lock);
    return rc;
}
```
- Sloppy counter
  - Logical counter
    - CPU core 별 logical counter
    - Global counter
    - Locks (각 local counter 별 하나씩, global counter 하나)
  - Basic idea
    - 각 CPU가 local counter를 갖고 있고, local counter에 있어선 중간 operation을 다른 core와 경쟁없이 반영.
    - 주기적으로 global counter에 반영
      - 자주 하면 sloppy counter의 장점이 퇴색, 드문드문하면 정확성 하락
    - 임계 값을 정하는 것이 중요
  - Example
![IMAGE](/images/kucse-operating-system/lockbased-sloppy-counter.png)
```
typedef struct __counter_t {
    int global;
    pthread_mutex_t glock;
    int local[NUMCPUS];
    pthread_mutex_t llock[NUMCPUS];
    int threshold; // ** update frequency **
} counter_t;
void init(counter_t *c, int threshold) {
    c->threshold = threshold;
    c->global = 0;
    pthread_mutex_init(&c->glock, NULL);
    int i;
    for (i = 0; i < NUMCPUS; i++) {
        c->local[i] = 0;
        pthread_mutex_init(&c->llock[i], NULL);
    }
}
```
```
void update(counter_t *c, int threadID, int amt) {
    int cpu = threadID % NUMCPUS;
    pthread_mutex_lock(&c->llock[cpu]); // local lock
    c->local[cpu] += amt; // assumes amt > 0
    if (c->local[cpu] >= c->threshold) {
        pthread_mutex_lock(&c->glock);
        c->global += c->local[cpu];
        pthread_mutex_unlock(&c->glock);
        c->local[cpu] = 0;
    }
    pthread_mutex_unlock(&c->llock[cpu]);
}

int get(counter_t *c) {
    pthread_mutex_lock(&c->glock);
    int val = c->global;
    pthread_mutex_unlock(&c->glock);
    return val; // only approximate!
}
```
get의 정확성이 떨어진다. local counter update가 그때 그때 반영이 안되서. 성능은 좋아졌다.

### (2)
- Concurrent Linked Lists
```
typedef struct __node_t {
    int key;
    struct __node_t *next;
} node_t;

typedef struct __list_t {
    node_t *head;
    pthread_mutex_t lock;
} list_t;

void List_Init(list_t *L) {
    L->head = NULL;
    pthread_mutex_init(&L->lock, NULL);
}

int List_Insert(list_t *L, int key) {
    pthread_mutex_lock(&L->lock);
    node_t *new = malloc(sizeof(node_t));
    if (new == NULL) {
        perror("malloc");
        pthread_mutex_unlock(&L->lock);
        return -1; // fail
    }
    new->key = key;
    new->next = L->head;
    L->head = new;
    pthread_mutex_unlock(&L->lock);
    return 0; // success
}

int List_Lookup(list_t *L, int key) {
    pthread_mutex_lock(&L->lock);
    node_t *curr = L->head;
    while (curr) {
        if (curr->key == key) {
            pthread_mutex_unlock(&L->lock);
            return 0; // success
        }
        curr = curr->next;
    }
    pthread_mutex_unlock(&L->lock);
    return -1; // failure
}
```
Critical section이 너무 크다.
- Scaling LinkedList
  - 공유자원 접근하는 부분만 lock 하자.
```
void List_Init(list_t *L) {
    L->head = NULL;
    pthread_mutex_init(&L->lock, NULL);
}
void List_Insert(list_t *L, int key) {
    // synchronization not needed
    node_t *new = malloc(sizeof(node_t));
    if (new == NULL) {
        perror("malloc");
        return;
    }
    new->key = key;
    // just lock critical section
    pthread_mutex_lock(&L->lock);
    new->next = L->head;
    L->head = new;
    pthread_mutex_unlock(&L->lock);
}

int List_Lookup(list_t *L, int key) {
    int rv = -1;
    pthread_mutex_lock(&L->lock);
    node_t *curr = L->head;
    while (curr) {
        if (curr->key == key) {
            rv = 0;
            break;
        }
        curr = curr->next;
    }
    pthread_mutex_unlock(&L->lock);
    return rv; // bug pruning
}
```

- Concurrent Queues
```
typedef struct __node_t {
    int value;
    struct __node_t *next;
} node_t;

typedef struct __queue_t {
    node_t *head;
    node_t *tail;
    pthread_mutex_t headLock;
    pthread_mutex_t tailLock;
} queue_t;

void Queue_Init(queue_t *q) {
    node_t *tmp = malloc(sizeof(node_t)); // dummy node
    tmp->next = NULL;
    q->head = q->tail = tmp;
    pthread_mutex_init(&q->headLock, NULL);
    pthread_mutex_init(&q->tailLock, NULL);
}

void Queue_Enqueue(queue_t *q, int value) {
    node_t *tmp = malloc(sizeof(node_t));
    assert(tmp != NULL);
    tmp->value = value;
    tmp->next = NULL;

    pthread_mutex_lock(&q->tailLock);
    q->tail->next = tmp;
    q->tail = tmp;
    pthread_mutex_unlock(&q->tailLock);
}

int Queue_Dequeue(queue_t *q, int *value) {
    pthread_mutex_lock(&q->headLock);
    node_t *tmp = q->head;
    node_t *newHead = tmp->next;
    if (newHead == NULL) {
        pthread_mutex_unlock(&q->headLock);
        return -1; // queue was empty
    }
    *value = newHead->value;
    q->head = newHead;
    pthread_mutex_unlock(&q->headLock);

    free(tmp);
    return 0;
}
```
- Concurrent Hash Table
```
#define BUCKETS (101)

typedef struct __hash_t {
    list_t lists[BUCKETS];
} hash_t;
void Hash_Init(hash_t *H) {
    int i;
    for (i = 0; i < BUCKETS; i++)
        List_Init(&H->lists[i]);
}
int Hash_Insert(hash_t *H, int key) {
    int bucket = key % BUCKETS;
    return List_Insert(&H->lists[bucket], key);
}
int Hash_Lookup(hash_t *H, int key) {
    int bucket = key % BUCKETS;
    return List_Lookup(&H->lists[bucket], key);
}
```

## Condition Variables
Condition을 기다리는 방법. 
- How to wait for a condition?
  - Spinning은 CPU 낭비.
  ```
  volatile int done = 0;
  void *child(void *arg) {
      printf("child\n");
      done = 1;
      return NULL;
  }
  int main(int argc, char *argv[]) {
      pthread_t c;
      printf("parent: begin\n");
      pthread_create(&c, NULL, child, NULL); // create child
      while (done == 0); // spin
      printf("parent: end\n");
      return 0;
  }
  ```
- Condition Variable
  - Thread가 어떤 상태를 기다리는 데, 상태가 만족되지 않는 시간은 sleep (특정 큐에 들어간다.)
  - 다른 Thread가 꺠울 수 있다.
  - ptrhead_cond_wait(queue, mutex)
    - 어떤 조건이 만족되지 않을때 만족되기까지 기다린다.
    - 함수내에서 unlock, sleep
  - pthread_cond_signal()
    - 이때 다시 lock, wait에선 return
  - Example
  ```
  int done = 0;
  pthread_mutex_t m = PTHREAD_MUTEX_INITIALIZER;
  pthread_cond_t c = PTHREAD_COND_INITIALIZER;
  void *child(void *arg) {
      printf("child\n");
      thr_exit();
      return NULL;
  }
  int main(int argc, char *argv[]) {
      pthread_t p;
      printf("parent: begin\n");
      pthread_create(&p, NULL, child, NULL);
      thr_join();
      printf("parent: end\n");
      return 0;
  }
  ```
  ```
  void thr_exit() {
      pthread_mutex_lock(&m);
      done = 1;
      pthread_cond_signal(&c);
      pthread_mutex_unlock(&m);
  }
  void thr_join() {
      pthread_mutex_lock(&m);
      while (done == 0)
      pthread_cond_wait(&c, &m);
      pthread_mutex_unlock(&m);
  }
  ```
  - State varibale이 없다면
  ```
  void thr_exit() {
      pthread_mutex_lock(&m);
      pthread_cond_signal(&c);
      pthread_mutex_unlock(&m);
  }
  void thr_join() {
      pthread_mutex_lock(&m);
      pthread_cond_wait(&c, &m);
      pthread_mutex_unlock(&m);
  }
  ```
    - Wait 하기전에 Signal. 
    - main threada를 깨울 애가 없어진다.
    - 따라서 state variable이 필요하다.
  - Lock이 없다면
  ```
  void thr_exit() {
      done = 1;
      pthread_cond_signal(&c);
  }
  void thr_join() {
      while (done == 0)
          pthread_cond_wait(&c);
  }
  ```
    - 마찬가지로 Wait 하기전에 signal.
### (2)
- Producer / Consumer Problem
  - Producer: 데이터 생산 thread
  - Consumer: 데이터 소비 thread
  - Example
    - Pipe
    - Web server
  - bounded buffer (큐의 길이가 한정적) 가 shared resouce 니까, 동기화가 필요하다. Condition variable 쓰자.
- Example
```
int buffer; // single buffer
int count = 0; // initially, empty
void put(int value) {
    assert(count == 0);
    count = 1;
    buffer = value;
}
int get() {
    assert(count == 1);
    count = 0;
    return buffer;
}
```
- Example - if
  - Producer
    ```
    cond_t cond;
    mutex_t mutex;
    void *producer(void *arg) {
        int i;
        for (i = 0; i < loops; i++) {
            pthread_mutex_lock(&mutex);
            if (count == 1)
                pthread_cond_wait(&cond, &mutex);
            put(i);
            pthread_cond_signal(&cond);
            pthread_mutex_unlock(&mutex);
        }
    }
    ```
  - Consumer
    ```
    void *consumer(void *arg) {
        int i;
        for (i = 0; i < loops; i++) {
            pthread_mutex_lock(&mutex);
            if (count == 0)
                pthread_cond_wait(&cond, &mutex);
            int tmp = get();
            pthread_cond_signal(&cond);
            pthread_mutex_unlock(&mutex);
            printf("%d\n", tmp);
        }
    }
    ```
![IMAGE](/images/kucse-operating-system/cond-if.png)
T(c1)이 wait, T(p)가 produce 했는데 T(c2)가 consume.  
T(c1)이 일어나보니 데이터가 없다 -> error

- Example - while
  - Producer
    ```
    cond_t cond;
    mutex_t mutex;
    void *producer(void *arg) {
        int i;
        for (i = 0; i < loops; i++) {
            pthread_mutex_lock(&mutex);
            while (count == 1)
                pthread_cond_wait(&cond, &mutex);
            put(i);
            pthread_cond_signal(&cond);
            pthread_mutex_unlock(&mutex);
        }
    }
    ```
  - Consumer
    ```
    void *consumer(void *arg) {
        int i;
        for (i = 0; i < loops; i++) {
            pthread_mutex_lock(&mutex);
            while (count == 0)
                pthread_cond_wait(&cond, &mutex);
            int tmp = get();
            pthread_cond_signal(&cond);
            pthread_mutex_unlock(&mutex);
            printf("%d\n", tmp);
        }
    }
    ```
![IMAGE](/images/kucse-operating-system/cond-while.png)
T(c1)이 sleep, T(c2)가 sleep, T(p)가 produce, signal, sleep  
T(c1)이 comsume 하고 sleep. producer를 꺠우려 했는데 consumer가 깨어났다.  
T(c2). 일어나보니 data가 없다.  
같은 condition variable이라는 게 문제.

- Example - while & Two CVs
  - Producer
    ```
    cond_t empty, fill;
    mutex_t mutex;
    void *producer(void *arg) {
        int i;
        for (i = 0; i < loops; i++) {
            pthread_mutex_lock(&mutex);
            while (count == 1)
                pthread_cond_wait(&empty, &mutex);
            put(i);
            pthread_cond_signal(&fill);
            pthread_mutex_unlock(&mutex);
        }
    }
    ```
  - Consumer
    ```
    void *consumer(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 0)
            pthread_cond_wait(&fill, &mutex);
        int tmp = get();
        pthread_cond_signal(&empty);
        pthread_mutex_unlock(&mutex);
        printf("%d\n", tmp);
        }
    }
    ```
### (3)
![IMAGE](/images/kucse-operating-system/cond-more.png)
- More concurrency
```
int buffer[MAX];
int fill_ptr = 0;
int use_ptr = 0;
int count = 0;

void put(int value) {
    buffer[fill_ptr] = value;
    fill_ptr = (fill_ptr + 1) % MAX;
    count++;
}
int get() {
    int tmp = buffer[use_ptr];
    use_ptr = (use_ptr + 1) % MAX;
    count--;
    return tmp;
}
```
```
cond_t empty, fill;
mutex_t mutex;
void *producer(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        pthread_mutex_lock(&mutex);
        while (count == MAX)
            pthread_cond_wait(&empty, &mutex);
        put(i);
        pthread_cond_signal(&fill);
        pthread_mutex_unlock(&mutex);
    }
}
```
```
void *consumer(void *arg) {
    int i, tmp;
    for (i = 0; i < loops; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 0)
            pthread_cond_wait(&fill, &mutex);
        tmp = get();
        pthread_cond_signal(&empty);
        pthread_mutex_unlock(&mutex);
        printf("%d\n", tmp);
    }
}
```
- Covering conditions
  - 모든 thread를 꺠워야 할 수도
  - pthread_cond_broadcast()
  - example
    - Multi-threaded memory allocation library

## Semaphores
### (1)
lock과 condition variable 두 목적으로 쓸 수 있다.
- POSIX
  - `sem_init(sem_t *s, int pshared, unsigned int value)`
    - semaphore init
    - 초기값 value 존재
    - pshared - 0이면 thread에서 공유, 0이 아니면 process끼리 공유. "shared memory"
  - sem_wait(sem_t *s)
    - s 값을 1씩 줄인다.
    - s가 음수면 프로세스를 큐에 재운다.
  - sem_post(sem_t *s)
    - s 값을 1 증가
    - sleep 된 프로세스가 있다면 꺠운다.
- Binary semaphore
```
sem_t m;
sem_init(&m, 0, 1);
sem_wait(&m);
// critical section here
sem_post(&m);
```
  - condition lock 대체 가능
![IMAGE](/images/kucse-operating-system/semaphore-binary.png)
- Semaphores for ordering
```
sem_t s;
void * child(void *arg) {
    printf("child\n");
    sem_post(&s);
    return NULL;
}
int main(int argc, char *argv[]) {
    pthread_t c;
    sem_init(&s, 0, X); // what should X be?
    printf("parent: begin\n");
    pthread_create(&c, NULL, child, NULL);
    sem_wait(&s);
    printf("parent: end\n");
    return 0;
}
```
  - 초기값을 뭐로 하지? lock 대신이면 1.. 0을 줄수도 있다.
![IMAGE](/images/kucse-operating-system/semaphore-ordering.png)
### (2)
- Producer/Consumer Problem
```
int buffer[MAX]; // bounded buffer
int fill = 0;
int use = 0;
void put(int value) {
    buffer[fill] = value;
    fill = (fill + 1) % MAX;
}
int get() {
    int tmp = buffer[use];
    use = (use + 1) % MAX;
    return tmp;
}
```
```
sem_t empty, sem_t full;
void *producer(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        sem_wait(&empty);
        put(i);
        sem_post(&full);
    }
}

void *consumer(void *arg) {
    int i, tmp = 0;
    while (tmp != -1) {
        sem_wait(&full);
        tmp = get();
        sem_post(&empty);
        printf("%d\n", tmp);
    }
}
```
```
int main(int argc, char *argv[]) {
    // ...
    sem_init(&empty, 0, MAX); // MAX are empty
    sem_init(&full, 0, 0); // 0 are full
    // ...
}
```

- Race condition
  - Single thread producer / cocnsumer 라면 동작
  - multi thread의 경우 put(), get() 에서 race condition
  - producer, consumer를 mutex 로 감싼다.
    ```
    void *producer(void *arg) {
        int i;
        for (i = 0; i < loops; i++) {
            sem_wait(&mutex);
            sem_wait(&empty);
            put(i);
            sem_post(&full);
            sem_post(&mutex);
        }
    }

    void *consumer(void *arg) {
        int i;
        for (i = 0; i < loops; i++) {
            sem_wait(&mutex);
            sem_wait(&full);
            int tmp = get();
            sem_post(&empty);
            sem_post(&mutex);
        }
    }
    ```
- Deadlock
  - 2개 이상 thread에서
  - 테스트케이스보단 production 경험.
  - 동기화는 설계부터 고려되어야 한다.
```
void *producer(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        sem_wait(&empty);
        sem_wait(&mutex);
        put(i);
        sem_post(&mutex);
        sem_post(&full);
    }
}

void *consumer(void *arg) {
    int i;
    for (i = 0; i < loops; i++) {
        sem_wait(&full);
        sem_wait(&mutex);
        int tmp = get();
        sem_post(&mutex);
        sem_post(&empty);
    }
}
```

### (3)
- Reader-Writer Locks
  - write 보다 read가 훨씬 많더라.
  - Reader
    - rwlock_acquire_readlock()
    - rwlock_release_readlock()
  - Writer
    - rwlock_acquire_writelock()
    - rwlock_release_writelock()
```
typedef struct _rwlock_t {
    // binary semaphore (basic lock)
    sem_t lock;
    // used to allow ONE writer or MANY readers
    sem_t writelock;
    // count of readers reading in critical section
    int readers;
} rwlock_t;
void rwlock_init(rwlock_t *rw) {
    rw->readers = 0;
    sem_init(&rw->lock, 0, 1);
    sem_init(&rw->writelock, 0, 1);
}
```
```
void rwlock_acquire_writelock(rwlock_t *rw) {
    sem_wait(&rw->writelock);
}
void rwlock_release_writelock(rwlock_t *rw) {
    sem_post(&rw->writelock);
}
```
writer starvation 문제가 있을 수 있다.
```
void rwlock_acquire_readlock(rwlock_t *rw) {
    sem_wait(&rw->lock);
    rw->readers++;
    if (rw->readers == 1)
        // first reader acquires writelock
        sem_wait(&rw->writelock);
    sem_post(&rw->lock);
}
void rwlock_release_readlock(rwlock_t *rw) {
    sem_wait(&rw->lock);
    rw->readers--;
    if (rw->readers == 0)
        // last reader releases writelock
        sem_post(&rw->writelock);
    sem_post(&rw->lock);
}
```
- How to implement semaphores
```
typedef struct __Sem_t {
    int value;
    pthread_cond_t cond;
    pthread_mutex_t lock;
} Sem_t;

// only one thread can call this
void Sem_init(Sem_t *s, int value) {
    s->value = value;
    Cond_init(&s->cond);
    Mutex_init(&s->lock);
}
```
```
void Sem_wait(Sem_t *s) {
    Mutex_lock(&s->lock);
    while (s->value <= 0)
        Cond_wait(&s->cond, &s->lock);
    s->value--;
    Mutex_unlock(&s->lock);
}

void Sem_post(Sem_t *s) {
    Mutex_lock(&s->lock);
    s->value++;
    Cond_signal(&s->cond);
    Mutex_unlock(&s->lock);
}
```
  - 원래 표준: value를 줄이고 음수면 잠든다. (number: 자고 있는 스레드 수)
  - 리눅스 구현: value는 0보다 작아질 수 없다.
    - 더 자세히) Linux는 condition variable이 아닌 futex로 구현.

## Common Concurrency Problems
### (1)
- Concurrency Problems
  - Non-deadlock bus
    - Atomicity-violation bugs: critical section X
    - Order-violation bugs: 순서 문제 해결 X
  - Deadlock bugs
- Atomicity-Violation Bugs
  - race condition이 발생하지 않을 것이란 기대. 하지만 atomic이 보장되지 않았다.
  - Example (MySQL)
  ```
  Thread 1:
  if (thd->proc_info) {
      ...
      fputs(thd->proc_info, ...);
      ...
  }

  Thread 2:
  thd->proc_info = NULL;
    ```
  - Atomicity-Violation Fixed
    ```
    pthread_mutex_t proc_info_lock = PTHREAD_MUTEX_INITIALIZER;
    Thread 1:
    pthread_mutex_lock(&proc_info_lock);
    if (thd->proc_info) {
    ...
    fputs(thd->proc_info, ...);
    ...
    }
    pthread_mutex_unlock(&proc_info_lock);
    Thread 2:
    pthread_mutex_lock(&proc_info_lock);
    thd->proc_info = NULL;
    pthread_mutex_unlock(&proc_info_lock);
    ```
- Order-Violation Bugs
  - A가 항상 B 전에 실행될 것이란 기대
  - Example
    ```
    Thread 1:
    void init() {
      ...
      mThread = PR_CreateThread(mMain, ...);
      ...
    }
    Thread 2:
    void mMain(...) {
      ...
      mState = mThread->State;
      ...
    }
  - Order-Violation Fixed
    - Thread 1
    ```
    pthread_mutex_t mtLock = PTHREAD_MUTEX_INITIALIZER;
    pthread_cond_t mtCond = PTHREAD_COND_INITIALIZER;
    int mtInit = 0;
    
    void init() {
        ...
        mThread = PR_CreateThread(mMain, ...);
        // signal that the thread has been created...
        pthread_mutex_lock(&mtLock);
        mtInit = 1;
        pthread_cond_signal(&mtCond);
        pthread_mutex_unlock(&mtLock);
        ...
    }
    ```
    - Thread 2
    ```
    void mMain(...) {
        ...
        // wait for the thread to be initialized...
        pthread_mutex_lock(&mtLock);
        while (mtInit == 0)
            pthread_cond_wait(&mtCond, &mtLock);
        pthread_mutex_unlock(&mtLock);
        mState = mThread->State;
        ...
    }
    ```
### (2)
- Deadlock Bugs
![IMAGE](/images/kucse-operating-system/deadlock.png)
  - Circular dependencies. 순환 참조
  ```
  Thread 1:
  pthread_mutex_lock(L1);
  pthread_mutex_lock(L2);

  Thread 2:
  pthread_mutex_lock(L2);
  pthread_mutex_lock(L1);
  ```
- Why do deadlocks occur?
  - 코드가 크니 일일히 찾기 쉽지 않다.
  - Example (virtual memory system)
    - VMS -> FS
    - FS -> VMS
    - 자주 읽는 데이터는 Intermediate Buffer 안에. (Buffer cache, Page cache) 
    - Circular request
  - Nature of encapsulation
    - Example (Java vector class)
      ```
      Vector v1, v2;

      Thread 1:
      v1.AddAll(v2);
      Thread 2:
      v2.AddAll(v1);
      ``
      - vector 내에서 lock이 잘 되어 있어도 deadlock이 발생함을 알기 어렵다.
- Conditions for Deadlocks
  - Mutual exclusion
    - race condition을 없애기 위해 만든 lock이지만 그로 인해 deadlock이.
  - Hold-and-wait
    - lock을 가진채로 다른 lock을 얻으려 한다.
  - No preemption
    - 다른 thread가 갖는 lock을 강제로 뺏어 올 수가 없다.
  - Circular wait
    - lock을 기다리는 구조가 circular
  - 이 네가지가 모두 만족 되었다면 circular
    - 1가지만 피하면 deadlock을 피할 수 있다.
- Deadlock prevention
  - Circular wait
    - lock acquire 순서가 다르면서 circular
      ```
      Thread 1:
      lock(L1);
      lock(L2);
      
      Thread2:
      lock(L2);
      lock(L1);
      ```
    - 그렇다면 lock acquire 순서가 모두 같으면 circular wait이 발생하지 않는다.
    - partial ordering
  - Hold-and-wait
    - 모든 lock을 한꺼번에 acquire
      ```
      pthread_mutex_lock(prevention); // begin lock acquisition
      pthread_mutex_lock(L1);
      pthread_mutex_lock(L2);
      ...
      pthread_mutex_unlock(prevention); // end
      ```
    - critical section이 커진다.
    - lock을 모두 알고 있어야 한다.
    - -> 모든 상황에 적용은 어렵다.
  - No preemption
    - 현재 대부분의 운영체젱서 강제로 뺏어 올 수 있는 방법은 없다.
    - 현실적 solution: trylock
      ```
      top:
      pthread_mutex_lock(L1);
      if (pthread_mutex_trylock(L2) != 0) {
          pthread_mutex_unlock(L1);
          goto top;
      }
      ```
    - 내가 L1, L2 둥다 못가져 온다면 둘다 포기하고 다시 처음부터 acquire.
    - livelock
      - 여러 thread가 모든 lock을 acquire 하지 못한 채로 loop
      - solution: goto 하기 전에 random delay
      - 주의점: goto 하기 전에 unlock. 자원 획득하기 전에 release 필요.
  - mutual excclusion
    - Lock-free approaches. Lock-free 알고리즘을 쓰자.
      ```
      void insert(int value) {
        node_t *n = malloc(sizeof(node_t));
        assert(n != NULL);
        n->value = value;
        pthread_mutex_lock(listlock);
        n->next = head;
        head = n;
        pthread_mutex_unlock(listlock);
      }
      ```
    - CompareAndSwap과 같은 atomic instruction을 쓸 수 있겠다.
      ```
      void insert(int value) {
          node_t *n = malloc(sizeof(node_t));
          assert(n != NULL);
          n->value = value;
          do {
              n->next = head;
          } while (CompareAndSwap(&head, n->next, n) == 0);
      }
      ```
    - CPU 자원 낭비 가능성, thread가 많아지면 효율이 구려진다.
- Deadlock Avoidance
  - 쉽지 않다.
![IMAGE](/images/kucse-operating-system/deadlock-avoid.png)
  - 여기저기 쓸 수 있는 방법은 아니다.
- Detect and recover
  - deadlock이 발생했을때 detect and recover?
  - 어려운 기술. trace가 가능한가?
  - 무엇을 recove? checkpoint??
  - **Restart !!**


- QnA
  - 리눅스를 분석해보고 싶은데요.... 익숙한 System call의 커널 entry point를 시작으로 내부 구현 코드를 추적해보자
  - 운영체제를 구현해보고 싶은데요.... RTOS 구현 from scratch.
