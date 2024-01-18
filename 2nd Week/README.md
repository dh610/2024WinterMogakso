# 2024 동계 모각소 2주차 활동

## 소프트웨어학과 202020169 윤동현

## 운영체제과목 학습을 위한 `PintOS` 과제 수행하기

### What's `PintOS`?

> * `PintOS`는 스탠포드 대학교 컴퓨터공학과 수업을 위해 개발된 교육용 운영체제이다.
> * 총 4개의 큰 Project가 존재하고, 각 Project마다 수행 과제들이 존재한다.
> * 본인은 KAIST에서 제공하는 `git clone https://github.com/casys-kaist/pintos-kaist` URL을 통해 PintOS 과제를 진행하였다.
> * `PintOS`는 실행 파일 실행 시 일반 컴퓨터처럼 `booting`이 되고 각종 `kernel`의 기본 요소들이 초기화 되고, 프로그램들을 추가로 `load`하는 방식으로 운영이 된다.
> * 이 기본 로직에서 성능적으로 개선할 만한 부분들을 개선하는 것을 과제로 삼아 프로젝트를 진행하게 된다.
> * 이번 동계 모각소를 진행하면서, Project 1에 대해 진행하고자 한다.

### Project 1. Threads - Alarm Clock

#### 과제 목표

 이번 모각소 활동 2주차에는 `PintOS`과제의 첫 번째 프로젝트의 첫 번째 과제인 `Alarm Clock` 과제를 수행하였다. 기존에 `PintOS`에서 구현돼있는 `timer_sleep()` 함수를 개선하는 것이 목표인 과제이다. 기존 코드는 아래와 같다.
```
void
timer_sleep (int64_t ticks) {
	int64_t start = timer_ticks ();

  while (timer_elapsed (start) < ticks)
		thread_yield ();
}
```
 위 코드는 기본적으로 이 함수를 요청한 스레드가 `ticks`시간만큼 아무 동작도 수행하지 않게 하는 함수이다. 하지만, 시간이 다 되지 않았을 경우 요청 스레드가 본인 차례로 scheduling 될 때마다 `thread_yield()` 함수를 호출하여 다음 scheduling thread로 넘기는 현상이 관찰되고, 이 `busy-waiting`을 해결하는 것이 첫 번째 과제의 핵심이다.
 
#### 시스템 흐름

 `PintOS` 컴퓨터는 기본적으로 `TIMER_FREQ`라는 상수를 통해 초당 몇 `tick`씩 동작하는지를 정의해뒀다. 한 `tick`을 기본 시간단위로 사용하고, 매 `tick`마다 `device` 디렉토리의 `timer.c`에 있는 `timer_interrupt()` 함수를 호출한다. 이 함수는 tick을 한개씩 늘리고 그 값을 `ticks` 전역변수에 저장하고, `thread_tick()`함수를 호출하여 컴퓨터의 총 실행 시간동안 idle 상태, kernel thread 상태, user 상태가 몇 `tick`씩 진행되었는지를 세도록 한다.

 스케줄링 방식은, 기본적으로 `ready_list`라는 doubly linked list에 저장된 스레드 중에 맨 앞에 있는 스레드를 꺼내고, 기존에 실행중이던 스레드를 맨 뒤로 보내는 `FIFO` 방식을 사용하였고, 진행중인 스레드가 종료되거나, `thread_yield()`함수를 호출했을 때, 다음 스레드로 `context switch`를 진행한다.

#### 문제 해결 방안

 `timer_sleep()`이 호출됐을 때, 일반적인 스레드들처럼 스케줄링을 하는 것이 아니고, 별도의 리스트를 만들어 원하는 시간이 될 때까지 그 리스트에 넣어두는 방식을 생각했다. 왜냐하면, 매 tick마다 어차피 `timer_interrupt()`가 호출이 되어 tick을 최신화하기 때문에, 이 때마다 새 리스트에 깨워야하는 스레드가 있는지만 확인해주면 되기 때문이다. 그래서 `sleep_list`라는 새 리스트를 선언해 주었다.
 새 리스트를 선언했으므로, `timer_sleep()`이 호출됐을 때, while문을 통해 대기하는 것이 아니고, `thread_sleep()`이라는 새 함수를 통해 현재 실행중인 스레드를 `sleep_list`에 추가한다.
 매 tick마다 `timer_interrupt()`에서 `sleep_list`에서 깨워야 할 스레드를 골라야 하므로, 깨울 스레드를 골라줄 새 함수인 `alarm_expire()`를 호출해준다. `alarm_expire()`을 구현함에 있어, 리스트에 들어있는 스레드들이 언제 깨어날지에 대한 정보가 필요하므로, `struct thread` 구조체에 `int64_t alarm` 변수를 선언하여 스레드를 깨워야 하는 tick을 저장해둔다.

#### 변경한 코드

> * `struct thread` 구조체
```
struct thread {
	/* Owned by thread.c. */
	tid_t tid;                          /* Thread identifier. */
	enum thread_status status;          /* Thread state. */
	char name[16];                      /* Name (for debugging purposes). */
	int priority;                       /* Priority. */
	int64_t alarm;
	int prio_orig;
	struct list lock_resource;
	struct lock *lock_wait;
	struct semaphore *sema_wait;

	/* Shared between thread.c and synch.c. */
	struct list_elem elem;              /* List element. */

#ifdef USERPROG
	/* Owned by userprog/process.c. */
	uint64_t *pml4;                     /* Page map level 4 */
#endif
#ifdef VM
	/* Table for whole virtual memory owned by thread. */
	struct supplemental_page_table spt;
#endif

	/* Owned by thread.c. */
	struct intr_frame tf;               /* Information for switching */
	unsigned magic;                     /* Detects stack overflow. */
};
```
>> 잠들 경우 깨어나야 하는 tick 값을 저장한 alarm 변수를 추가했다.
>> 
> * `timer_sleep()` 함수
```
void timer_sleep (int64_t ticks) {
	int64_t start = timer_ticks ();
	thread_sleep(ticks + start);
}
```
>> 기존의 while문을 `thread_sleep()`함수를 호출하는 부분으로 변경하여 `busy-waiting` 문제를 해결하였다.

> * `timer_interrupt()` 함수
```
static void timer_interrupt (struct intr_frame *args UNUSED) {
	ticks++;
	thread_tick ();
	alarm_expire(ticks);
}
```
>> 깨워야 할 스레드가 있을 경우 깨워주는 `alarm_expire()` 함수를 추가했다.

> * `thread_init()` 함수
```
void
thread_init (void) {
	ASSERT (intr_get_level () == INTR_OFF);

	/* Reload the temporal gdt for the kernel
	 * This gdt does not include the user context.
	 * The kernel will rebuild the gdt with user context, in gdt_init (). */
	struct desc_ptr gdt_ds = {
		.size = sizeof (gdt) - 1,
		.address = (uint64_t) gdt
	};
	lgdt (&gdt_ds);

	/* Init the globla thread context */
	lock_init (&tid_lock);
	list_init (&ready_list);
	list_init (&sleep_list);
	list_init (&destruction_req);

	/* Set up a thread structure for the running thread. */
	initial_thread = running_thread ();
	init_thread (initial_thread, "main", PRI_DEFAULT);
	initial_thread->status = THREAD_RUNNING;
	initial_thread->tid = allocate_tid ();
}
```
>> `sleep_list`가 추가되었으므로, 이를 초기화하는 커맨드를 추가했다.

#### 새로 추가한 코드

> * `thread_sleep()` 함수
```
void thread_sleep(int64_t ticks) {
	enum intr_level old_level = intr_disable();
	struct thread *curr = thread_current();

	curr->alarm = ticks;

	list_push_back(&sleep_list, &curr->elem);

	thread_block();

	intr_set_level(old_level);
}
```
>> `thread_block()`함수는 현재 실행중인 스레드를 blocking 상태로 만들고 남은 `ready_list`에서 CPU를 점유할 스레드를 고르도록 하는 함수이다. `context switching`을 유발하므로, 호출 전 interrupt를 꺼주고 호출해야 한다. 그 외 부분은 앞에서 설명한 방식과 같다.

> * `alarm_expire()` 함수
```
void alarm_expire(int64_t ticks) {
	if(!list_empty(&sleep_list))
		for (struct list_elem *e = list_begin(&sleep_list); e !=list_end(&sleep_list);) {
			struct thread *tmp = list_entry(e, struct thread, elem);

			if (tmp->alarm <= ticks) {
				e = list_remove(e);
				thread_unblock(tmp);
			} else e = list_next(e);
		}
}
```
>> `thread_unblock()`함수는 인수로 받은 스레드를 ready 상태로 만들고 `ready_list`에 넣어 새로 스케줄링을 진행하도록 하는 함수이다. 그 외 부분은 앞에서 설명한 대로이다.

#### 결과 비교

> * 결과 전체 길이가 너무 길어서 마지막 실행 시간 명세 부분만 발췌해 왔다.
```
Timer: 586 ticks
Thread: 550 idle ticks, 36 kernel ticks, 0 user ticks
```
> * 수정 전에는 idle ticks가 0으로 나왔는데, `busy-waiting`개선 이후에는 idle상태가 상당히 길게 나오는 것을 확인할 수 있다.
