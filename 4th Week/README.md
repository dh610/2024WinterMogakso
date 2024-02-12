# 2024 동계 모각소 4주차 활동

## 소프트웨어학과 202020169 윤동현

## PintOS Project 1-Priority Scheduling 

### Synchronization

`Multithread` 환경에서는 여러 스레드가 공유 리소스를 가지고 동작하기 때문에 여러 문제가 발생할 수 있다. 그 중 하나가 동기화(synchronization) 문제이다.

스레드 a, b가 10의 값을 가지고 있는 변수 x의 값을 1 증가시켜 저장하는 코드를 각각 실행한다고 가정해보자.

멀티스레드 환경에서는 `concurrent`하게 처리되기 때문에, 우리가 기대하는 것은 한 스레드가 변수 x에서 10을 가져오고 1을 더한 11을 다시 저장하고, 다음 스레드가 11을 가져와서 1을 더하고 12를 저장하여 최종적으로 x의 값이 12가 되는 결과를 예상할 수 있다.

하지만, 결과가 11이 될 수 있는데, 이는 스레드 a, b가 10이라는 값을 각각 불러오면, 1을 더해도, 둘 다 11이라는 값을 x에 저장할 수밖에 없다.

따라서, 한 스레드가 공유 리소스에 접근할 때 다른 리소스가 접근하지 못하도록 해 주어야 하고, 이를 `Synchronization`(동기화)라고 한다.

동기화 문제를 해결하는 법으로는 mutex lock, semaphore 등이 있다.

### Semaphore

`PintOS`에서 `struct semaphore`로 세마포어를 정의했으므로 이 구조체에 따라 설명하겠다. 구조체의 선언은 다음과 같다.

```
struct semaphore {
  int value;
  struct list waiters;
};
```

세마포어를 관리하는 두 함수 `sema_up()`과 `sema_down()`의 정의는 다음과 같다.

```
void sema_down (struct semaphore *sema) 
{
  enum intr_level old_level;

  ASSERT (sema != NULL);
  ASSERT (!intr_context ());

  old_level = intr_disable ();
  while (sema->value == 0) 
    {
      list_push_back (&sema->waiters, &thread_current ()->elem);
      thread_block ();
    }
  sema->value--;
  intr_set_level (old_level);
}

void sema_up (struct semaphore *sema) 
{
  enum intr_level old_level;

  ASSERT (sema != NULL);

  old_level = intr_disable ();
  if (!list_empty (&sema->waiters))
  {
    thread_unblock (list_entry (list_pop_front (&sema->waiters),
                                struct thread, elem));
  }
  sema->value++;
  intr_set_level (old_level);
}
```

위의 흐름을 보면 알 수 있듯이, 스레드가 접근할 때마다 `.value`의 값을 1씩 내린다. `.value` 값이 0이 될 경우 대기를 하게 되고 스레드 구조체는 `.waiters` 리스트의 요소로 들어간다. 리소스 이용이 끝난 스레드가 세마포어의 값을 늘리면 그때서야 `.waiters`에서 대기중인 스레드가 깨어나게 된다. 세마포어는 이런식으로 작동하는 `Synchronization` 방식이다.

여기서 대기하는 스레드들 또한 우선순위 스케줄링에 맞게 스케줄링 되도록 수정해주어야 하는데, 이미 앞선 프로젝트에서 `compare_priority()`비교 함수를 통해 `list_insert_ordered()` 함수를 사용해봤으므로, 그에 맞게 수정만 해주면 된다.

`sema_down()` 함수를 보면 `list_push_back()` 함수를 사용하는 부분이 있으므로, 이 부분만 우선순위 순서에 맞게 들어가도록 `list_insert_ordered()` 함수로 교체해주면 된다.

추가로, 우선순위가 변할 때, `ready_list`에 대해서만 재정렬 하기 때문에, `.waiters` 리스트에 대해서도 재정렬 해줄 필요가 있다. 우선순위가 변하는 상황을 모두 찾아 코드를 추가하는 것은 번거로우므로, 위의 `compare_priority()` 비교 함수를 인자로 받는 `list.h`의 함수 `list_sort()` 함수를 활용하여, `sema_up()`을 호출할 때마다 정렬하도록 한다.

또한 `thread_unblock()`을 통해 `ready_list`에 새 원소가 추가됐으므로, `thread_preemption()`을 통해 새로 스케줄링을 진행한다.

수정 결과는 다음과 같다.

```
void sema_down (struct semaphore *sema) {
	enum intr_level old_level;

	ASSERT (sema != NULL);
	ASSERT (!intr_context ());

	old_level = intr_disable ();
	
	while (sema->value == 0) {
		list_insert_ordered(&sema->waiters, &thread_current()->elem, compare_priority, NULL);
		thread_block ();
	}

	sema->value--;
	intr_set_level (old_level);
}

void sema_up (struct semaphore *sema) {
	enum intr_level old_level;

	ASSERT (sema != NULL);

	old_level = intr_disable ();
	if (!list_empty (&sema->waiters)) {
		list_sort(&sema->waiters, compare_priority, 0);
		thread_unblock (list_entry (list_pop_front (&sema->waiters),
					struct thread, elem));
	}
	sema->value++;

	thread_preemption();

	intr_set_level (old_level);
}
```

### Mutex lock

특정 리소스에 대해 잠금을 실행하여, 다른 어느 프로세스도 접근할 수 없도록 하는 것이다. 잠금이 걸린 상태에서 다른 프로세스가 접근하려고 할 때, 다른 프로세스들은 전부 대기상태가 된다.

PintOS에서는 구조체 `struct lock`과 그에 관련된 함수들로 구현했다. 세마포어가 구현된 상태이기 때문에, 값이 1인 세마포어를 설정하여 구현했다.

```
struct lock {
	struct thread *holder;
	struct semaphore semaphore;
};
```

> .holder는 리소스를 점유 중인 스레드를 가리키는 포인터이고, .semaphore는 값이 1인 세마포어이다.

```
void lock_acquire (struct lock *lock) {
	ASSERT (lock != NULL);
	ASSERT (!intr_context ());
	ASSERT (!lock_held_by_current_thread (lock));

	sema_down (&lock->semaphore);
	lock->holder = thread_current ();
}

void lock_release (struct lock *lock) {
	ASSERT (lock != NULL);
	ASSERT (lock_held_by_current_thread (lock));

	lock->holder = NULL;
	sema_up (&lock->semaphore);
}
```

> 잠금을 요청하고 해제하는 함수들이다. 값이 1인 세마포어를 설정하였기에, 잠금은 하나의 스레드만 점유할 수 있고, 세마포어 값을 0으로 만들어 잠금을 만들 수 있으므로 잠금 요청 시는 `sema_down()` 함수를 사용하고, 반대로 잠금 해제 시에는 `sema_up()` 함수를 사용한다.

잠금에 대한 코드 수정은 `Priority Inversion`문제를 다루고 구현해야하므로 다음 주차에 다루도록 하겠다.
