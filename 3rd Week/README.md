# 2024 동계 모각소 3주차 활동

## 소프트웨어학과 202020169 윤동현

## PintOS Project 1-Priority Scheduling

아래는 과제 링크이다.
https://casys-kaist.github.io/pintos-kaist/project1/priority_scheduling.html

이번에 수행할 과제는 PintOS의 스케줄링 방식을 우선순위 스케줄링 방식으로 바꾸는 것이다.

기존의 PintOS는 `Round-Robin` 방식을 이용하여 먼저 들어온 스레드를 다 수행하고 그 다음 스레드를 진행하는 방식으로 스케줄링을 진행했다.

이번 과제는 기존에 `Round-Robin` 방식으로 구현돼있는 PintOS의 스케줄링 방식을 `Priority Scheduling` 방식으로 스케줄링 하도록 새로 구현하는 것이다.

### Process State

 * `New`: 프로세스가 생성되어 `Ready`상태가 된다.
 * `Ready`: `New` 단계를 거쳐 `Ready Queue`에 들어간 상태이다(`Ready Queue`는 스케줄링 대기중인 프로세스들의 모임이다).
 * `Running`: 프로세스가 CPU를 점유하고 있는 상태이다.
 * `Waiting`: `Ready`와는 다른 개념으로, 프로세스가 I/O 등 다른 처리를 기다리고 있는 상태이다.
 * `Terminated`: 종료 상태이다.

### Scheduling

CPU 한개는 한 가지의 프로세스만 처리가 가능하다. 하지만, CPU를 점유하는 프로세스가 `Waiting`상태가 될 경우 CPU가 아무 행동도 하지 않게 된다. 이 상황이 이전 주차에서 해결했던 `busy waiting` 문제이다. 이를 해결하고 항상 CPU가 돌게 만들도록 운영체제가 `Process scheduling`을 활용하여 프로세스가 `Waiting` 상태가 될 경우 `Ready` 상태인 프로세스 중 하나를 골라 `Running` 상태로 바꿔주는 `Context Switching`을 진행한다. 스케줄링에는 `FCFS`, `SJF` `Round-Robin`, `Priority Scheduling` 등 다양한 방식이 존재하는데 현재 과제에서는 `Priority Scheduling` 방식을 다루고 있으므로 후에는 이에 대한 설명만 진행한다.

### Context Switching

앞서 스케줄링을 진행할 때 `Context Switching`이라는 것을 진행한다고 했는데, 이에 대해 설명하겠다.

이를 설명하기 전에 PCB(Process Control Block)이라는 것을 설명하자면, 이는 각 프로세스마다 존재하는 고유 데이터같은 것이다. 메모리의 Kernel Space에 doubly linked list 자료구조 형태로 있으며, 모든 프로세스에 대해 1대1로 존재하는 데이터이다. 프로세스의 ID, 상태, 레지스터 정보, 메모리 정보, 입출력 상태 등의 정보가 저장되어 있다.

`Context Switching`이 발생하면, Kernel 영역에서 CPU가 실행되며, 현재 `Running` 상태인 프로세스가 현재의 레지스터 정보나 메모리 정보 등을 PCB에 새로 저장하고, PCB list에서 다음 스케줄링 대상인 프로세스의 PCB 정보를 가져온다. 그 정보들을 기반으로 다음 스케줄링 대상인 프로세스가 이전에 끊겼었던 상태 그대로 새로 프로세스를 진행할 수 있도록 해준다.

### Preemptive

* `Non-preemptive`: 스케줄링 진행 시, 스케줄러가 현재 `Running` 상태인 프로세스에 개입할 수 없는 스케줄링 방식을 의미한다.
* `Preemptive`: 스케줄링 진행 시, 스케줄러가 현재 `Running` 상태인 프로세스에 개입하여 새로 스케줄링을 진행할 수 있도록 하는 스케줄링 방식을 의미한다.

### Priority Scheduling

각 프로세스마다 주어진 우선순위를 기반으로 스케줄링을 진행한다. 현재 진행하는 `PintOS` 과제에서는 `Preemptive` 기반 `Priority Scheduling`을 구현하는 것이 목적이므로, 이에 대한 설명만 진행하겠다. `Priority Scheduling`은 `Non-preemptive`하게도 구현이 가능하다.

스케줄러는 `Ready queue`에 들어있는 프로세스들 중에 가장 우선순위가 높은 프로세스를 `Running` 상태로 만든다. 참고로 이 우선순위는 운영체제마다 `Policy`에 따라 다를 수 있으며, 이 과제에서는 우선순위가 알아서 주어진다는 가정 하에 과제를 진행하므로, 이에 대한 고려는 하지 않아도 된다.

이후에 `Ready queue`에 현재 `Running` 상태인 프로세스보다 우선순위가 높은 프로세스가 들어온다면, 현재 진행중인 프로세스를 중단시키고 `Context switching`을 진행하여 우선순위가 가장 높은 프로세스를 새로 선택한다. 

위의 일련의 과정을 계속 반복하면서 스케줄링을 진행한다.

### 코드 구현
