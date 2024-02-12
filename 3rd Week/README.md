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

과제 파일의 `include/threads/thread.h`를 확인하면 `struct thread`라는 구조체를 확인할 수 있다. 이 구조체를 스케줄링 대상의 단위로 삼아 프로그램을 진행한다.

구조체에 `int priority`가 선언돼있는 것을 확인할 수 있고, 이 값을 기반으로 `list.h`에 들어있는 함수들로 정렬을 진행하여 스케줄링 관리를 할 수 있음을 알 수 있다.

`PintOS`의 커널은 `thread_yield()` 함수를 통해 `ready_list` 리스트에서 다음 실행할 스레드를 불러온다. 현재 진행 중인 과제는 우선순위가 가장 높은 스레드를 우선으로 선택하는 알고리즘을 구현하는 것이고, 이는 두 가지 방법으로 구현이 가능하다.

1. `thread_yield()`를 호출할 때마다 `ready_list`에서 가장 높은 우선순위를 가진 스레드를 고르도록 하는 것
2. 새 스레드가 리스트에 들어갈 때 마다 우선순위 순위에 맞는 위치에 스레드를 넣는 것

1번 방법과 2번 방법의 시간복잡도의 차이는 없지만, `list.h` 파일에 `list_insert_oredered()`라는 함수가 구현돼있으므로 이 함수를 이용해 2번 방법을 활용한다. 이 함수는 대소비교 함수를 직접 제작해서 함수 argument로 넣어줘야 한다. 리스트 원소들의 우선순위 대소비교를 위한 함수는 다음과 같다.

```
bool compare_priority(const struct list_elem *a, const struct list_elem *b, void *aux) {
	return list_entry(a, struct thread, elem)->priority > list_entry(b, struct thread, elem)->priority;
}
```

> `list_entry()`는 `list.h`에 구현된 매크로로, `list_elem`구조체를 포함하는 어떤 구조체 전체를 나타내는 매크로이다. 위에서는 `struct thread`를 두 번째 인자로 넣었으므로, a 혹은 b `list_elem`을 포함하는 `struct thread`의 주소를 나타내게 된다.

비교함수도 구현했으므로, 리스트에 스레드를 넣는 부분을 전부 찾아서 다음으로 바꿔준다.

```
list_insert_ordered(&ready_list, &curr->elem, &compare_priority, NULL);
```

리스트에 새 스레드를 넣는 부분은 `thread_yield()`, `thread_unblock()` 두 함수에 있으므로 그 부분을 위의 함수로 대체하면 된다.

현재 과제는 `preemptive`한 스케줄러를 구현하는 것이 목표이므로, `preemption` 발생 조건을 만족하는 부분을 찾아 수정을 해줘야 한다.

`preemption`을 발생시키는 경우는 크게 두 가지가 있다.

1. 현재 `Running` 상태인 스레드의 우선순위보다 더 높은 우선순위를 가진 스레드가 `Ready queue`에 들어왔을 때.
2. `Running` 상태인 스레드의 우선순위가 새로 설정되어 `Ready queue`에 존재하는 스레드의 우선순위 중 최댓값보다 작게 되었을 때.

이 두가지 상황을 만족할 때마다 `thread_yield()` 함수를 실행하여, 새로 스케줄링을 해줘야 한다. 따라서, `thread_preemption()`라는 함수를 새로 만들어 이 함수를 위의 상황마다 넣어준다. 함수의 코드는 아래와 같다.

```
void thread_preemption(void) {
	if (!list_empty (&ready_list) && thread_current ()->priority < 
    list_entry (list_front (&ready_list), struct thread, elem)->priority)
        thread_yield ();
}
```

1번 상황의 경우 `thread_create()`를 호출하여 새 스레드를 생성할 때이고, 2번 상황은 `thread_set_priority()`함수를 통해 우선순위를 새로 설정할 때이므로 각각 함수의 마지막에 위 함수를 넣어준다.
