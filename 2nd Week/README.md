# 2024 동계 모각소 2주차 활동

## 소프트웨어학과 202020169 윤동현

## 운영체제과목 학습을 위한 `PintOS` 과제 수행하기

### What's `PintOS`?

> * `PintOS`는 스탠포드 대학교 컴퓨터공학과 수업을 위해 개발된 교육용 운영체제이다.
> * 총 4개의 큰 Project가 존재하고, 각 Project마다 수행 과제들이 존재한다.
> * 본인은 KAIST에서 제공하는 `git clone https://github.com/casys-kaist/pintos-kaist` URL을 통해 PintOS 과제를 진행하였다.
> * `PintOS`는 실행 파일 실행 시 일반 컴퓨터처럼 `booting`이 되고 각종 `kernel`의 기본 요소들이 초기화 되고, 프로그램들을 추가로 `load`하는 방식으로 운영이 된다.
> * 이 기본 로직에서 성능적으로 개선할 만한 부분들을 개선하는 것을 과제로 삼아 프로젝트를 진행하게 된다.

### Project 1. Threads - Alarm Clock

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
 
