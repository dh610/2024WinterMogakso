# 2024 동계 모각소 6주차 활동

## 소프트웨어학과 202020169 윤동현

## PintOS Project 2-Argument Passing

### 개요

프로젝트 1에서는 커널 영역에서만 동작했었다. 하지만 이제 프로젝트 2 부터는 유저 영역의 코드를 실행할 수 있는 환경을 만들어야 한다.

유저 영역에서 요청하는 각 시스템 콜을 처리할 수 있어야 유저 영역의 코드를 실행했을 때, 컴퓨터가 잘 동작할 수 있으므로, 이 시스템 콜에 대해 구현하는 것이 프로젝트 2의 궁극적인 목표이다.

이번 주차에 진행할 과제는 프로젝트 2의 argument passing 부분이다.

`threads/init.c` 파일에 들어가면, main 함수가 위치한 것을 확인해 볼 수 있다. main 함수에서는 각종 커널의 요소들을 이용할 수 있도록 초기화 해주고, `thread_start()` 함수를 통해 메인 스레드와 `idle` 스레드를 열고, `run_actions()`->`run_task()`를 통해 다음 코드를 실행한다.

```
process_wait(process_create_initd (task));
```

두 함수 `process_wait()`, `process_create_initd()`는 모두 `userprog/process.c`에 정의돼있다. `process_wait()`는 시스템 콜 `wait()`와 관련한 코드이므로, 이번 주차에서 다루지는 않을 예정이다. `process_create_initd()`의 코드는 다음과 같다.

```
tid_t process_create_initd (const char *file_name) {
	char *fn_copy;
	tid_t tid;

	fn_copy = palloc_get_page (0);
	if (fn_copy == NULL)
		return TID_ERROR;
	strlcpy (fn_copy, file_name, PGSIZE);

	tid = thread_create (file_name, PRI_DEFAULT, initd, fn_copy);
	if (tid == TID_ERROR)
		palloc_free_page (fn_copy);
	return tid;
}
```

이 코드의 동작을 간략히 요약하면, `initd()`함수를 fn_copy라는 argument를 사용해 새 스레드에서 실행하는 것이다. 또한 새로 만든 스레드의 id 값을 반환한다. 즉, 앞에서 `process_wait()` 함수에 argument로 들어가는 값은 여기서 새로 생긴 스레드의 id 값이고, `process_wait()`는 새 스레드가 종료될 때까지 기다려야 한다.

`initd()`의 동작을 분석하기 위해 코드를 확인해보자.

```
static void initd (void *f_name) {
#ifdef VM
	supplemental_page_table_init (&thread_current ()->spt);
#endif

	process_init ();

	if (process_exec (f_name) < 0)
		PANIC("Fail to launch initd\n");
	NOT_REACHED ();
}
```

현재 단계에서는 VM매크로를 정의하지 않으므로 ifdef~endif 부분은 무시하도록 하겠다.

프로세스를 초기화하고 `process_exec()`함수를 실행하는 것을 확인할 수 있다. `process_exec()` 함수는 다음과 같다.

```
int
process_exec (void *f_name) {
	char *file_name = f_name;
	bool success;

	struct intr_frame _if;
	_if.ds = _if.es = _if.ss = SEL_UDSEG;
	_if.cs = SEL_UCSEG;
	_if.eflags = FLAG_IF | FLAG_MBS;

	process_cleanup ();

	success = load (file_name, &_if);

	palloc_free_page (file_name);
	if (!success)
		return -1;

	do_iret (&_if);
	NOT_REACHED ();
}
```

인터럽트 프레임을 초기화 해주고, `process_cleanup()`을 통해 현재 실행중인 프로세스를 전부 비운다. exec계열 시스템 콜은 현재 실행중인 프로세스를 새로 실행할 프로세스로 대체하는 것이기 때문에 그렇다.

프로세스를 비우고 나서는 `load()`함수를 호출하고, 함수 결과가 잘못됐을 경우 -1을 반환하고 그것이 아니라면 새 프로세스의 정보를 담은 `_if` 인터럽트 프레임에 대해 `do_iret()`함수를 호출하여 스레드를 시작한다.

여기서 사용되는 `load()`함수를 채우는 것이 이번 과제에서의 목표이다. 현재 `load()`함수는 페이지 테이블을 생성 후, argument로 받은 파일을 그대로 읽어 파일의 내용을 메모리에 올리는 동작을 실행한다.

이번 과제의 첫 번째 목표는 argument로 받은 파일 이름을 parsing해서 파일 뒤의 argument들을 무시하여 혼동이 생기지 않도록 하는 것이다.

두 번째 목표는, parsing argument들을 제공한 양식에 맞추어 메모리의 스택 영역에 저장하는 것이다.

### System call parameter passing

시스템 콜 사용시 argument를 전달하는 방법은 여러 가지가 있다.

1. argument들을 레지스터에 저장하는 방법.
2. 스택 영역에 저장하는 방법.

첫 번째 방법은 argument가 많아질 경우 레지스터의 갯수가 모자라서 좋지 않은 방법일 수도 있다. 그 대신 간단하기 때문에 PintOS 체제에서는 대부분에 system call에 대해 첫 번째 방법을 사용한다.

두 번째 방법은 이번 과제에서 구현할 방법으로, 스택 영역에 argument들을 저장하는 방법이다.

`/bin/ls -l foo bar` 라는 parameter가 들어왔다고 가정했을 때, 스택 영역의 결과는 다음과 같다.(user stack의 시작주소는 0x47480000이다.)

|Address|Name|Data|Type|
|:---:|:---:|:---:|:---:|
|0x4747fffc|argv[3][...]|'bar\0'|char[4]|
|0x4747fff8|argv[2][...]|'foo\0'|char[4]|
|0x4747fff5|argv[1][...]|'-l\0'|char[3]|
|0x4747ffed|argv[0][...]|'/bin/ls\0'|char[8]|
|0x4747ffe8|word-align|0|uint8_t[]|
|0x4747ffe0|argv[4]|0|char *|
|0x4747ffd8|argv[3]|0x4747fffc|char *|
|0x4747ffd0|argv[2]|0x4747fff8|char *|
|0x4747ffc8|argv[1]|0x4747fff5|char *|
|0x4747ffc0|argv[0]|0x4747ffed|char *|
|0x4747ffb8|return address|0|void (*) ()|

위와 같이 각 문자열들을 stack 영역에 저장하고, align을 맞추기 위해 8로 나누어떨어지는 값까지 null로 채운다.

그 후, 각 문자열들의 스택 상의 주소를 저장해주는데 구분을 위해 마지막 주소는 null로 채워준다.

스택 포인터는 인터럽트 프레임 구조체의 rsp 레지스터를 활용하여 조작할 수 있다.

또한, argument의 위치들과 갯수를 저장하기 위해, rsi 레지스터에 스택 상에서 argument의 시작 주소, 그리고 argument의 개수를 rid 레지스터에 저장한다.

마지막에 가짜 반환주소 0을 저장한다.

위의 예시에서는 rsi 레지스터에 0x4747ffc0을 저장하고 rdi 레지스터에 4를 저장한다.

### 코드 구현(tokenize)

우선 `load()` 함수는 file_name이라는 문자열 변수와 if_라는 인터럽트 프레임 포인터를 인수로 받는다.

이 file_name 문자열을 tokenize 하는 것이 첫 번째로 할 일인데, 과제 설명에서 제공돼있듯이 `strtok_r()` 함수를 사용하면 간단하게 parsing이 가능하다. 구현 코드는 다음과 같다.

```
	char *argv[64] = { 0 };
	char *save_ptr;
	int argc = 0;

	for (argv[argc] = strtok_r(file_name, " \t\r\f\r\n\v", &save_ptr);
		argv[argc] != NULL; argv[argc] = strtok_r(NULL, " \t\r\f\r\n\v", &save_ptr))
			argc++;
```

> 위 코드를 `load()`함수의 변수 선언 직후에 추가해주면 된다.
> 굳이 argv 배열을 0으로 초기화한 이유는 후술할 코드 구현에 설명돼있다.
 
아래로 계속 내려가면 `setup_stack()`함수를 호출한 부분이 나오는데, 이 때, 스택의 세팅이 끝나기 때문에 이 이후에 코드를 구현한다.

### 코드 구현(load to stack)

이제 스택 영역에 argument 정보들을 저장해야 한다.

위의 parameter passing에서 설명했듯이, argument들을 순서대로 스택 영역에 저장하면 된다. 스택 포인터는 `if_->rsp`가 나타낸다. `<string.h>`에 선언된 `memcpy()` 함수를 이용하여 옮겼다. 코드는 다음과 같다.

```
	for (int i = argc - 1; i >= 0; i--) {
		int size = (int)strlen(argv[i]) + 1;
		if_->rsp -= size;
		memcpy(if_->rsp, argv[i], size);
		argv[i] = (char *)if_->rsp;
	}
```
> 구분자인 null값까지 옮기기 위해 size를 배열의 길이보다 1 길게 설정했다.
> 이후에 주소를 옮기기 편하도록 argv값을 새 주소로 설정해주었다.

이제 align을 위해 8의 배수에 맞춰 값을 전부 0으로 세팅해야 한다. 그에 대한 코드는 다음과 같다.

```
	int align = if_->rsp % 8;
	if_->rsp -= align;
	memset(if_->rsp, 0, align);
```

이제 다음 스택 영역에 주소들을 저장해야 한다. 다음과 같이 구현했다.

```
	int size = 8 * (argc + 1);
	if_->rsp -= size;
	memcpy(if_->rsp, argv, size);
```
> 본인의 다음 원소와 연속하게 존재하는 배열의 특성을 이용하여 반복문을 이용하지 않고 한번에 옮겼다. 이 때 NULL 포인터까지 한번에 옮기기 위해 이전에 argv 배열을 0으로 초기화했던 것이다.

이제 스택 포인터의 위치와 argument 갯수를 저장해야 한다. 또, 가짜 반환 주소를 스택에 저장한다.

```
	if_->R.rsi = if_->rsp;
	if_->R.rdi = argc;

	if_->rsp -= 8;
	memset(if_->rsp, 0, 8);
```

위의 코드들을 추가하면 argument passing이 끝난다.
