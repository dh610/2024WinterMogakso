# 2024 동계 모각소 5주차 활동

## 소프트웨어학과 202020169 윤동현

## PintOS Project 1-Priority Scheduling

### Priority Inversion

우선순위 스케줄링을 진행할 때, `Priority Inversion`(우선순위 역전)이라는 현상이 발생할 수 있다.

우선순위가 32, 31, 30인 세 프로세스 H, M, L이 있다고 하자. L이 먼저 스케줄링 되어 실행되고, A라는 리소스에 lock을 걸었다고 가정하자.

이 때, H가 스케줄링 되어 실행될 경우, 현 PintOS 체제는 Preemptive 한 스케줄링 방식을 사용하기 때문에, H가 running 상태로 바뀌게 된다. 이 경우 H가 A 리소스에 접근하려고 하면 lock이 걸려있기 때문에 block 상태가 되어 CPU를 다른 프로세스에 넘겨주게 된다.

이런 상황에서, L보다 우선순위가 높은 M이 스케줄링 되면, L은 M이 끝날때 까지 기다릴 수밖에 없고, H는 L이 lock을 해제할 때까지 기다려야 하므로, 우선순위가 가장 높은 H가 상대적으로 우선순위가 가장 낮은것 처럼 동작하게 되며 우선순위 역전이 발생한다.

우선순위 역전 문제를 해결하는 방법으로는, `Priority Inheritance Protocol`(PIP), `Priority Ceiling Protocol`(PCP) 두 가지가 있다.

### Priority Ceiling Protocol

프로세스에서 잠금을 사용하기 시작하면, 그 순간부터 우선순위를 높여주는 방식이다. 어느 정도로 올려줄지는 OS를 디자인하는 사람이 정하기 나름이며, 보통 `Ready Queue`에 존재하는 프로세스 중 가장 우선순위가 높은 프로세스의 우선순위만큼 올려주는 방식을 사용한다고 한다.

구현 방식이 매우 간단하지만, 우선순위를 어느 정도로 올려줄지를 정하는 데에 대한 문제가 있다.

### Priority Inheritance Protocol

어떤 리소스를 기준으로 우선순위가 더 낮은 프로세스가 리소스를 잠금을 통해 점유 중일때, 우선순위가 더 높은 프로세스가 그 리소스에 접근 시도를 할 때, 이미 점유중인 우선순위가 낮은 프로세스의 우선순위를 현재 접근 시도를 하는 우선순위가 더 높은 프로세스의 우선순위만큼 상승시키는 방식이다.

현 과제에서는 높은 우선순위의 프로세스가 낮은 우선순위의 프로세스에게 우선순위를 donation한다는 용어로 설명하고 있다.

다양한 변수를 고려해야 해서 구현이 좀 더 까다로운 편이다. `Multiple Donation`, `Nested Donation` 두 가지 문제 발생을 고려하며 구현을 해야한다.

* `Multiple Donation`
  여러 번의 donation이 일어난 상황이다. 앞의 H, M, L로 예시를 들자면, L이 리소스 A와 B를 둘 다 점유하고 있는 상황을 보자.

  M은 리소스 A에 접근할 예정이고, H은 리소스 B에 접근할 예정이다. M이 스케줄링되어 리소스 A에 접근한다고 했을 때, M은 본인의 우선순위인 31을 L에 기부하게 되어 L의 우선순위가 31이 되게 된다.

  그 후 H가 리소스 B에 접근하면, H가 우선순위를 L에게 기부하여 L의 우선순위는 32가 된다.

  L이 리소스 B에 대한 잠금을 해제하면, H로부터 받은 우선순위를 해제하고, M으로부터 받은 우선순위인 31이 L의 우선순위가 된다.

  리소스 A에 대한 잠금을 해제하고 나서야 본인의 원래 우선순위로 돌아가게 된다.

  이와 같이 한 프로세스가 다른 여러 프로세스로부터 우선순위 기부를 받는 상황을 `Multiple Donation`이라고 한다.

* `Nested Donation`
  여러 겹의 donation이 일어난 상황이다. 앞의 H, M, L로 예시를 들자면, L이 리소스 A를 점유하고, M이 리소스 B를 점유하고 있는 상황을 보자.

  M이 리소스 A에 접근시도를 하면 M은 L에게 우선순위를 기부하여 L의 우선순위가 31이 된다.

  이 상황에서, H가 리소스 B에 접근시도를 하여 M의 우선순위를 32로 올려봤자, M은 리소스 A에 대해 대기중인 상황이기 때문에 우선순위를 기부한 의미가 없어지게 된다.

  따라서, H는 M과 L 모두에게 우선순위를 기부하여 세 스레드가 전부 32의 우선순위를 가지게 된다.

  L이 리소스 A에 대한 점유를 내려놓으면, 본인이 기부받은 32와 31의 우선순위는 전부 리소스 A와 관련된 우선순위이므로, 본인의 원래 우선순위인 30으로 돌아간다.

  그 후 M이 리소스 B에 대한 점유를 내려놓으면, M의 우선순위도 원래대로 돌아간다.

  이와 같이 리소스를 점유하는 프로세스에만 기부를 하는 것이 아니라 대기의 원인이 되는 모든 프로세스에 우선순위를 기부하는 상황을 `Nested Donation`이라고 한다.

### 구현

우선 스레드 구조체를 변경해줄 필요가 있다. 각 스레드가 어떤 리소스를 요청하고 있는지 확인할 수 있어야 하므로 `struct lock`에 대한 원소를 추가한다.

또한, 원래 우선순위로 돌아가야하므로 그에 대한 원소도 추가한다. 마지막으로, 리스트를 만들어 현재 점유중인 리소스도 확인할 수 있도록 한다.

따라서 `struct thread`에 아래의 코드를 추가한다.

```
int prio_orig;
struct lock *lock_wait;
struct list lock_resource;
```

또한 잠금에 대해 리스트를 만들었으므로, `struct lock`에 아래의 코드를 추가한다.

```
struct list_elem elem;
```

`struct thread`에 새 원소들이 생겼으므로 `init_thread()` 함수도 다음의 초기화 코드를 추가해준다.

```
t->prio_orig = priority;
list_init(&lock_resource);
```

`lock_aquire()` 함수를 다음과 같이 수정한다.

```
void lock_acquire (struct lock *lock) {
	ASSERT (lock != NULL);
	ASSERT (!intr_context ());
	ASSERT (!lock_held_by_current_thread (lock));

	if (lock_try_acquire(lock)) return;

	thread_current()->lock_wait = lock;

	for (struct thread *tmp1 = thread_current(), *tmp2; tmp1->lock_wait != NULL; tmp1 = tmp2) {
		tmp2 = tmp1->lock_wait->holder;
		if (thread_get_priority() <= tmp2->priority) break;
		tmp2->priority = thread_get_priority();
		if (tmp2->lock_wait != NULL) {
			list_remove(&tmp2->elem);
			list_insert_ordered(&tmp2->lock_wait->semaphore.waiters,
				&tmp2->elem, compare_priority, NULL);

		}
	}

	sema_down (&lock->semaphore);
	list_push_back(&thread_current()->lock_resource, &lock->elem);
	thread_current()->lock_wait = NULL;
	lock->holder = thread_current();
}
```

> `lock_try_aquire()` 함수는 잠금을 시도하여 성공할 경우 리소스를 점유한 상태로 true를 반환하고, 실패할 경우는 그냥 false를 반환하는 함수이다. 이 함수를 통해 이후 donation 관련 코드를 실행할지 말지를 정한다.
> 이후 코드는 리소스 점유에 실패한 상황. 즉, 대기하는 상황이므로, 현재 스레드의 lock_wait에 대기할 lock을 추가한다.
> 반복문을 돌면서 본인의 lock과 관련하여 대기중인 모든 스레드의 우선순위를 변경해야 할 경우 변경한다.
> 그 이후 세마포어 함수를 통해 스레드를 대기 상태로 만든다.
> 세마포어 함수가 끝났다는 것은 대기가 끝나고 리소스를 점유하는데 성공했다는 뜻이므로, lock_resource 리스트에 획득한 리소스를 추가하고, lock_wait를 NULL로 설정하여 lock에 대한 대기를 하고 있지 않음을 나타낸다.
> 현재 스레드가 리소스를 점유하게 됐으므로 lock의 holder를 현재 스레드로 설정한다.

`lock_release()` 함수를 다음과 같이 수정한다.

```
void
lock_release (struct lock *lock) {
	ASSERT (lock != NULL);
	ASSERT (lock_held_by_current_thread (lock));

	struct thread *curr = thread_current();

	list_remove(&lock->elem);
	curr->priority = curr->prio_orig;

	if (list_empty(&curr->lock_resource)) goto exit;

	for (struct list_elem *e = list_begin(&curr->lock_resource); e != list_end(&curr->lock_resource); e = list_next(e)) {
		struct lock *tmp = list_entry(e, struct lock, elem);
		if (!list_empty(&tmp->semaphore.waiters))
			curr->priority = list_entry(list_begin(&tmp->semaphore.waiters), struct thread, elem)->priority >
			curr->priority ? list_entry(list_begin(&tmp->semaphore.waiters), struct thread, elem)->priority : curr->priority;
	}

exit:

	lock->holder = NULL;
	sema_up (&lock->semaphore);
}
```

> 잠금을 해제하므로 `list_remove()`함수를 통해 현재 스레드의 lock_resource 리스트에 현재 리소스가 없도록 한다.
> 이후의 코드는 우선순위를 원래대로 돌리거나, 혹은 다른 스레드의 우선순위를 받거나를 정하는 코드이다.
> 우선 스레드의 우선순위를 prio_orig 값으로 변경하여 최초의 상태로 만든다.
> 그 후 lock_resource 리스트가 비었다면, donation을 받지 않아도 되므로 exit로 이동한다.
> 그렇지 않다면, lock_resource 리스트를 탐색하면서 lock의 holder중에 가장 우선순위가 높은 스레드의 우선순위를 받는다.
> 그 이후 exit 루틴을 수행한다.
> 리소스를 점유하는 스레드가 없게 되므로 lock의 holder를 NULL로 설정해주고, 세마포어 함수를 통해 현재 잠금을 점유할 수 있는 상태로 만들고 스레드는 `ready_list`에 들어가 스케줄링 후보가 된다.

우선순위에 대한 정책이 바뀌었으므로, 우선순위를 바꾸는 함수인 `thread_set_priority()` 함수도 변경해줘야 한다. 변경 내용은 다음과 같다.

```
void thread_set_priority (int new_priority) {
	struct thread *curr = thread_current();
	curr->priority = new_priority;
	curr->prio_orig = new_priority;
	if (!list_empty(&curr->lock_resource))
		for (struct list_elem *e = list_begin(&curr->lock_resource); e != list_end(&curr->lock_resource); e = list_next(e)) {
			struct lock *tmp = list_entry(e, struct lock, elem);
			if (!list_empty(&tmp->semaphore.waiters))
				curr->priority = list_entry(list_begin(&tmp->semaphore.waiters), struct thread, elem)->priority >
				curr->priority ? list_entry(list_begin(&tmp->semaphore.waiters), struct thread, elem)->priority : curr->priority;
		}
	
	thread_preemption();
}
```

> 우선순위를 바꾸는 함수기에 요청값대로 우선순위를 변경한다. 우선순위가 donation에 의해 변경된 것이 아니므로 prio_orig도 새 우선순위 값으로 설정해준다.
> 그 후는 위의 `lock_release()` 함수에서 처리한 것처럼 반복문을 통해 lock_resource 리스트를 탐색하여 우선순위를 변경할지 말지 정하고, 변경해야 할 경우 변경한다.
> 우선순위가 변경됐기 때문에 `thread_preemption()` 함수 호출을 통해 새로 스케줄링을 할 수 있도록 한다.
