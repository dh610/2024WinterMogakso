# 2024 동계 모각소 1주차 활동

## 소프트웨어학과 202020169 윤동현

## 실전코딩2 강의 수강 내용 정리

### 1번째 강의

1. 아두이노를 통한 interrupt 동작 확인하기

> * 아두이노는 기본적으로 `setup()`을 통해 기본 설정을 진행하고, `loop()`을 무한 루프로 실행하면서 동작한다.
> * `loop()`내의 동작에 따라 핀 번호를 통해 버튼, led 등의 디바이스 제어가 가능하다.
> * 버튼 입력에 따라 led를 제어하는 과제를 수행하였으며, `loop()` 내에서 버튼 입력이 들어오는 것을 지속적으로 listen 할 경우 busy하다는 것을 알게 되었다.
> * 버튼 입력을 `interrupt` 발생 조건으로 추가한다면, 버튼의 상태를 반복마다 check할 필요가 없어지므로, busy하지 않게 된다.

### 2번째 강의

1. Computer Hardware System

> * 대부분 컴퓨터들은 폰 노이만의 구조를 따르며, `CPU`와 `Memory`를 중심으로 다른 `Device`들이 붙는 형태를 가진다.
> * `CPU`가 직접 `Device`에 붙지는 않고, `Memory`를 통해 각종 `Device`들을 제어하게 된다.
> * 각 `Device`와 `Memory`의 중간다리 역할을 하는 `Buffer`가 존재한다.
> * `Buffer`는 붙어있는 `Device`의 종류에 따라 `Input Buffer`, `Output Buffer` 두 가지로 나뉜다.
> * 추가로 `Disk`는 storage device로 `Device`의 일종이기 때문에 `CPU`에서 직접 접근할 수 없고 `Memory`를 통해 접근해야 한다.

2. LINUX Software

> * LINUX는 네 가지 껍질로 하드웨어를 둘러싸고 있다. 각각 `Kernel`, `System Call`, `System Utilities`, `Linux Shell`이다.
> * `Kernel`은 하드웨어 바로 상위 layer로, 각종 low-level operation을 수행한다.
> * `System Call`은 `Kernel`과의 interface 역할을 하는 layer이고 `Kernel`을 감싸고 있는 구조이다.
> * `System Utilities`는 `System Call`을 감싸고 있는 상위 layer이고, `cat`, `ls`, `date`, `ps`, `kill`, `who` 등의 흔히 알고 있는 LINUX 명령어들이 이 layer에 속한다.
> * `Linux Shell`은 `System Utilities`를 감싸고 있는 상위 layer이고, `bash` 같은 것들이 이 layer에 속한다.

3. LINUX Kernel

> * Linux Kernel의 Device Driver는 크게 세 가지로 나눈다. 그 세 가지는 `Character device drivers`, `Network device drivers`, `Block device drivers`이다.
> * `Character device drivers`와 `Block device drivers`는 한 번에 처리하는 데이터의 단위를 기준으로 구분되며, `Character device drivers`는 1byte character 단위, `Block device drivers`는 정해진 block 단위로 읽는다.
> * 각각 file attribute의 첫 번째 문자가 c와 b로 나타난다.

4. Redirection

> * terminal에 따라오는 세 개의 device 는 `stdin`, `stdout`, `stderr`이고, 각각의 번호는 0, 1, 2번이다.
> * Redirection 명령어로는 `<`계열 `>`계열이 존재하고, pipe 명령어인 `|`도 있다.
> * `<` 계열: 입력 스트림을 변경하는 명령어다. variation으로 `<<`, `<<<`가 있다. 사용할 때는 `"실행파일" < "텍스트파일"`과 유사한 형태로 사용한다. 추가로 `0<`의 형태로도 사용이 가능한데 0번이 `stdin`이므로 같은 명령어이다.
> * `>` 계열: 출력 스트림을 변경하는 명령어다. variation으로 `>>`, `&>`, `&>>`가 있다. `<` 계열의 사용과 마찬가지로 `"실행파일" > "텍스트파일"`과 유사한 형태로 사용한다. 마찬가지로 1번이 `stdout`이므로 `1>`과 같은 동작을 수행한다고 볼 수 있다. `2>`도 있는데, 이는 에러 메시지만 전달하는 동작을 하게 된다.
> * 모든 프로세스는 기본적으로 키보드 입력 `stdin`, 화면 출력 `stdout`, `stderr`가 원칙이다. 하지만 Redirection을 이용한다면, 화면에 출력돼야할 결과가 다른 파일로 이동한다던가, 프로세스를 실행하면 입력을 받기위해 blocking을 실행해야 하는데 다른 파일로부터 입력을 받아버려서 blocking을 하지 않는다던가 등의 결과가 나타난다.
> * `<<`: `"명령어" <<"문자열"`형태로 사용하며, "문자열"과 같은 문자열이 입력될 때 까지 입력을 받는다. 입력이 끝나면 모든 입력을 "명령어"의 입력 스트림으로 전달한다.
> * `>>`: `>`의 경우 file의 시작 부분부터 새로 write 하므로, 기존의 내용이 보존되지 않는다. 하지만 `>>`를 사용하면, file의 마지막 부분부터 write하여 기존의 내용을 보존하면서 내용을 추가하는 것이 가능해진다.
> * `>&`: 출력에 관한 모든 스트림 `stdout`, `stderr`를 모두 같은 목적지로 Redirection하는 명령어이다.
> * `|`: 한 프로세스의 출력 스트림을 다음 프로세스의 입력 스트림으로 전달하는 명령어이다.
> * Redirection은 명령어의 순서를 마구잡이로 작성해도 정상적으로 동작한다. 예를 들어 `echo hello > /tmp/out` 명령어와, `> /tmp/out echo hello`, `echo >/tmp/out hello` 전부 정상적으로 동작한다.
> * 다만 Redirection을 여러 개 사용할 경우 순서를 마구잡이로 작성했을 때, 에러는 나지 않지만 원하는 결과가 나오지 않을 경우가 있으므로 유의해서 사용해야 한다.

5. Buffer

> * 모든 스트림은 buffer라는 중간 다리가 있다. buffer로 데이터가 전송되고, 특정 조건을 만족할 경우 buffer가 비워지면서 데이터가 목적지로 전송된다.
> * 통상적인 `stdin`, `stdout`의 경우 개행문자 `'/n'`을 buffer를 flush하는 기준으로 삼는다.
> * buffer가 조건이 맞춰지지 않았을 때에도 `fflush()` System Call을 이용해 버퍼를 비워줄 수 있다.

6. printf, scanf

> * `printf()`, `scanf()`는 각각 `stdout`, `stdin`을 목적지로 데이터를 전송하는(받는) 함수이다.
> * variation으로 `fprintf()`, `fscanf()`가 있는데, 이들은 맨 앞의 매개변수에 출력 혹은 입력받을 file pointer를 작성한다. 이때 이 매개변수로 각각 `stdout`, `stdin`을 사용할 경우 `printf()`, `scanf()`와 동일하게 작동하게 된다.

### 3번째 강의

1. Buffer Control

> * Buffer 설정에는 세 가지 방식이 있다. `unbuffered`, `line buffered`, `fully buffered` 세 가지 방법이다.
> * `unbuffered`: `buffering`을 사용하지 않는 방법. 1byte마다 바로바로 스트림을 통해 데이터를 전달한다.
> * `line buffered`: 개행문자 `'/n'`을 기준으로 `buffer`를 `flush`하는 방법이다. 일반적으로 표준입출력에서 많이 사용하는 방법이다.
> * `fully buffered`: 설정한 `byte size`를 기준으로 size만큼 `buffer`가 가득 차면 `flush`하는 방법이다.
> * `fully buffered`는 `setvbuf()` 함수를 통해 특정 스트림으로의 `buffer` size를 설정해주는 방식으로 code level에서 적용이 가능하다.
> * `Buffering`을 하는 이유: 컴퓨터 시스템 상으로 입출력 장치가 가장 느리다. 그 장치의 연산을 위해 지속적으로 관여하는 것 자체가 비효율 적이기에, 매 byte마다 전송하는 것이 아닌 `buffer` 단위로 전송하여 입출력 장치에 관여하는 횟수를 줄여 성능 개선을 도모한다.
> * `buffer`의 크기가 너무 작을 경우는 성능이 나빠지고, 크기가 너무 클 경우는 입출력 장치의 작업 수행 효율이 나빠진다.
> * `buffer`를 바꾸기 전에 반드시 `buffer`를 `flush`해줘야 한다. 하지 않고 바꾸면 `buffer`에 들어있던 값이 버려지기 때문.

2. Pipe

> * Pipe는 bash 상에서 `|`명령어를 통해 구현할 수 있다. 예를 들어. `ls | wc`를 하면 ls의 실행 결과가 터미널에 출력되지 않고 wc의 입력으로 들어가 그 결과가 터미널 상으로 출력되는 방식이다.
> * Pipe를 통해 여러 명령어를 작성할 경우, background에서 명령어들이 병렬적으로 처리되고, Input을 받아야 할 때는 blocking을 진행하여 다른 프로세스가 input을 줄 때까지 대기한다.
> * bash 명령어로도 &&, || 등의 연산자를 사용할 수 있으며, 이 때 &&를 할 경우 각 프로세스들이 순차적으로 처리된다. 실제로 `sleep 1 && sleep 1 && sleep 1`은 3초 후에 끝나는 반면, `sleep 1 | sleep 1 | sleep 1`은 병렬적으로 처리되어 1초면 끝난다.
> * Named pipe는 mkfifo 명령어로 생성할 수 있다. 생성했을 경우 새 파일이 생성되며, 그 파일의 file attribution을 확인했을 때 맨 앞글자가 p인 것을 확인할 수 있다.
> * Named pipe는 redirection을 통해 어떤 명령어의 표준출력을 pipe에 받고, 받은 내용들을 다른 명령어의 표준입력으로 보내는 역할을 할 수 있다.
> * `popen()`함수를 이용해 파이프를 열 수 있다.
> ```
> int main() {
>     char buf[256];
>     int len;
>     FILE *pin = popen("ls", "r");
>     FILE *pout = popen("wc", "w");
> 
>     len = fread(buf, sizeof(char), sizeof(buf), pin);
>     fwrite(buf, sizeof(char), len, pout);
> 
>     pclose(pin);
>     pclose(pout);
> 
>     return 0;
> }
> ```
> * 위 코드는 수업시간 실습에 사용한 코드를 에러처리만 빼고 간략히 쓴 것이다. 이를 통해 `popen()`의 사용법을 익힐 수 있다.
> * ls명령어의 결과를 `pin` pipe에 받아 `buf`에 저장 후 그 값을 `pout` pipe로 보내 wc는 ls의 결과를 토대로 연산을 진행, 즉 `ls | wc`와 같은 결과를 출력하게 된다.

3. foreground와 background
> * foreground: 우리가 현재 보고 있는 bash 상태를 말한다.
> * background: 우리 눈에 보이지 않게 뒤에서 프로세스들이 동작하게 하는 영역을 말한다.
> * 명령어 마지막에 띄어쓰기 후 `&`를 작성해 주면, 명령어가 background에서 동작한다.
> * 이 때, foreground에서 `ps`명령어를 입력하면, background 명령어 사용 전에 `ps`를 입력했을 때 보이지 않던 새 프로세스가 돌고 있는 것을 확인할 수 있다.
> * `Ctrl+Z`명령어로 동작을 일시정지 시킬 수 있고, `Ctrl+C`로 동작을 종료시킬 수 있다.
> * fg 명령어로 background에서 작동하는 프로세스를 foreground로 가져올 수 있고, 거꾸로 bg 명령어를 통해 foreground에서 작동하는 프로세스를 bg로 옮길 수 있다. 어떤 프로세스를 이동시킬 것인지는 `ps`명령어를 통해 확인할 수 있는 프로세스 아이디 번호를 통해 지정할 수 있다.
> * kill 명령어를 이용해서 지정된 프로세스를 종료시킬 수 있다.

### 4번째 강의

1.  C compiler

> * 컴파일 순서
>> 1. `c파일`을 `cpp`를 통해 컴파일 할 수 있게 변형한다.(`preprocessing`과정)
>> 2. `cpp`가 완료된 파일을 컴파일 과정을 통해 `어셈블리 파일`로 바꾼다
>> 3. `어셈블리 파일`을 어셈블러를 통해 `오브젝트 파일`로 만든다.
>> 4. 여러 `오브젝트 파일`을 링커가 연결하여 `executable file`을 만든다.
>> 5. `Loader`가 `executable file`을 메모리에 load 함으로써 프로그램을 실행하게 된다.
> * 컴파일 옵션 (`cc`명령어로 컴파일하는 기준)
>> * `-E`: `cpp` 명령어와 같은 기능을 수행하며, preprocessing을 진행할 경우 결과 코드가 어떻게 되는지 표준 출력으로 출력해준다.
>> * `-S`: 컴파일 과정을 거치며, `.s`확장자를 가지는 `어셈블리 파일`을 만든다.
>> * `-o`: `.o`를 가지는 `오브젝트 파일`을 만든다.
>> * 옵션을 지정하지 않을 경우 위의 과정을 모두 실행하여 `executable file`만 만들어준다.
> * `linking`
>> * `cc` 명령어 사용 시 여러 파일을 지정할 경우, `linking` 과정을 추가하여 하나의 `executable file`로 만든다.
>> * `linking` 진행 시 `extern` 선언에 대한 처리를 진행하게 된다.
>> * 여러 프로그램을 `linking`할 때 사소한 팁
>>> - `extern` 선언은 `.h`, 즉 헤더파일에 선언해두는 것이 좋다. 그리고 함수의 `main body`는 해당 헤더파일을 `include`하는 `.c`파일에 작성하는 것이 좋다.
>>> - 한 프로그램을 여러 파일로 만들어 둘 때, `오브젝트 파일`로 만들어 두는 것이 좋다. 수정하지 않은 파일까지 `Compile->Assemble`하지 않고, 수정한 파일만 그 과정을 거쳐 `오브젝트 파일`을 만든 후 `Linking`만 하면 되기 때문이다.
> * 그 외
>> * CISC: 명령어 종류가 많은 시스템 (intel) <-> RISC (ARM)
>> * 컴파일할때 `cc -Wall`을 해야 모든 경고 메시지를 출력해볼 수 있다.
>> * warning이어도 컴파일은 되지만, 그렇게 하지 않는게 좋다.
>> * [bash] echo $PATH 명령어를 입력하면, 명령어 입력시 찾는 경로의 순서를 볼 수 있다.
>> * `-pg` 옵션을 사용해서 프로그램의 성능을 확인해볼 수 있다.

2. C preprocessor(CPP)

> * cpp의 역할
>> 1. Include Header Files
>> 2. Define Macro
>> 3. Conditional Compilation
>> 4. Line Control
> * Include Header Files
>> * 컴파일러에서 기본 지정된 경로로 헤더파일을 찾으러 간다.
>> * 경로를 바꾸고 싶다면 `-I`옵션을 통해 변경 가능.#
>> * 또한 `:`를 활용해 여러 경로를 지정할 수 있다.
>> * 두 번 include하는 것을 방지해야한다. `#pragma once`라고 하면 한번만 include 된다.
>>> ```
>>> #ifndef _MATH_
>>> #include <math.h>
>>> #endif
>>> ```
>>> * 위처럼 작성해도 두 번 include되는 것을 방지할 수 있다.
>>> * 다만 대부분의 헤더파일은 여러번 include되는것을 방지하기 위해 자체적으로 `#ifndef`를 활용하여 예방한다.
>> * 헤더파일을 수정했을 경우, 헤더파일을 include하는 c파일들을 전부 다시 컴파일 해주어야 한다. h파일은 컴파일 필요x
> * Define Macro
>> * macro는 소스코드를 바꿔끼는 것이다. integer값 같은 것들을 바꾸는 것이 아니라 문자열 그대로 바꿔치기만 한다.
>>> ```
>>> #define PI 3.14159
>>> 
>>> PIPI
>>> PI PI (PI) PI-PI PI,PI PI_PI
>>> ```
>> * `PIPI`및 `PI_PI`는 전환이 안되지만, 그 아랫줄들은 전부 전환이 된다. token으로 분할되어있기 때문.
>>> ```
>>> #define f(a) a*a
>>> b = f(20+13)
>>> ```
>> * 위의 코드는 매크로를 치환하면 `b = 20+13*20+13`이 되므로 원하는 결과가 나오지 않는다.
>> * `cc -D_PCC_VERSION=1 main.c func.c`를 통해 매크로의 값을 바꿀 수 있다. 다만 코드 실행 중 `#define _PCC_VERSION` 부분을 만나면 선언 값으로 변경된다.(여기서 `_PCC_VERSION`은 매크로 이름의 예시이다.)
>> * `##`: 두 문자열을 합쳐주는 역할을 한다. 매크로 변환 시 최우선으로 변환한다.
>>> ```
>>> #define HE HI
>>> #define LLO _THERE
>>> #define HELLO "HI THERE"
>>> #define CAT(a,b) a##b
>>> #define XCAT(a,b) CAT(a,b)
>>> CAT(HE,LLO)
>>> XCAT(HE,LLO)
>>> ```
>>> * `CAT(HE,LLO)`를 했을 때는, CAT매크로가 `##`을 사용중이므로 우선순위가 높다. 따라서 `HELLO`로 변환되고, `HELLO`의 매크로인 `HI THERE`가 출력된다.
>>> * `XCAT(HE,LLO)`를 했을 때는, 세 매크로가 전부 `##`을 사용중이지 않으므로, `CAT(HI,_THERE)`로 치환된다. 결과적으로 `HI _THERE`가 출력된다.
>>> * 다만, 이렇게 쓰는 것은 혼란을 야기할 수 있어 좋은 습관은 아니라고 할 수 있다.
>> * `#`은 매개변수를 문자열로 바꾸어준다. `#define STR(a) #a`라고 선언할 경우 `a`로 들어오는 문자열로 바꿔준다고 보면 된다. 만약 `#define STR(a) a`라고 선언할 경우 매개변수와 관계없이 `a`가 된다.
>> * 매크로 식 선언시 매개변수마다 괄호를 쳐주는 것이 좋다.
>> '#define DEBUGPRINT(_fmt, ...)  fprintf(stderr, "[file %s, line %d]: " _fmt, __FILE__, __LINE__, __VA_ARGS__)'
>> * 위 매크로를 사용하면 어떤 파일의 몇 번째 줄에서 디버깅 메시지를 출력하는지 확인해 볼 수 있다.
>> * 디버깅 메시지를 on/off를 편하게 하기 위해서 `DEBUGPRINT`전후로 `ifdef`, `endif`를 넣어준다.
>> * 그 후 `ifdef`에서 확인하는 매크로를 `cc -D`를 활용해서 선언할 경우 디버깅 메시지를 출력할 수 있고, 해당 옵션을 사용하지 않을 경우 디버깅 메시지를 실행하지 않게 된다.
> * Conditional Compilation
>>> ```
>>> #define _PCC_VERSION 1
>>> 
>>> #if _PCC_VERSION == 1
>>> int fmul (int a, int b) {
>>>     return a * b;
>>> }
>>> #else
>>> int fmul (int aaa, int bbb) {
>>>     return aaa + bbb + 1;
>>> }
>>> #endif
>>> ```
>> * 위의 코드는 `_PCC_VERSION`의 값에 따라 fmul이 어떤 함수인지가 달라진다. 이를 `Conditional Compilation`이라고 하고, cpp를 통해 어느 `fmul`함수가 컴파일 되는지 확인해 볼 수 있다.
>> * 추가로 위의 코드에서 `#if`를 열었으면 반드시 `#endif`를 통해서 `#if`를 닫아줘야 한다.
>> * `#warning`을 통해 컴파일 과정에서 경고 메시지를 발생시킬 수 있다.

### 5번째 강의

#### 1. Floating Point vs Fixed Point

> * 각 자료형이 표현가능한 수의 범위
>> * `signed`: 자료형의 비트가 n비트라고 했을 때 `-2^(n-1)~(2^(n-1))-1`의 범위의 수들을 표현하고, 이들은 `2^n`가지의 수이다.
>> * `unsigned`: 자료형의 비트가 n비트라고 했을 때 `0~(2^n)-1`의 범위의 수들을 표현하고, 이들은 `2^n`가지의 수이다.
>> * `signed` `unsigned`에 따라 전기 신호가 변하는 것이 아니고 각 비트를 처리하는 원칙이 바뀌는 것일 뿐임.
> * 2의 보수(2's complement)
>> * signed 자료형 기준으로 음수와 양수를 표현하기 위한 방법이다.
>> * MSB는 부호를 표현하는 bit로 사용되고, 0일때 양수, 1일때 음수를 나타낸다.
>> * 절댓값이 같은 어떤 자연수와 음의 정수의 합이 0인 것을 이용해서 변환한다.
>> * 원래 2진수 표현의 각 비트에 not을 취하고, 그 2진수에 1을 더하면 부호 변환이 완료된다.
> * floating point(부동소수점)
>> * sign(1bit) + exponent(8bit) + mantissa(23bit)
>> * 표현식: `(-2 * sign + 1) * (1.0 + mantissa * 2^(-23)) * 2^(exp - 127)`
> * 고정소수점
>> * 소수위치를 몇번째 비트와 그 다음비트 사이의 값으로 고정
>> * 예를 들어, 1byte 자료형 기준으로 오른쪽에서 4번째 비트와 5번째 비트 사이에 소수점이 존재한다고 했을 때, 오른쪽에서 4번째 비트는 2^(-1)을 곱하고, 5번째 비트는 2^0을 곱한다.
>> * q번째 비트 다음에 소수점이 위치한다고 가정했을 때 사칙연산은 아래와 같이 진행된다.
>>> * 덧셈/뺄셈: 위의 소수점 원칙을 따른다고 했을 때, `*(2^(-q))`를 추가해서 연산한다.
>>> * 곱셈: 덧셈/뺄셈과 똑같이 진행하되, `/(2^(-q))`를 추가해서 연산한다.
>>> * 나눗셈: 두 수를 나누고 `*2^q`를 추가해서 연산한다.
> * 추가사항
>> * 삼각함수나 행렬곱, 벡터 등 추가구현하면 추가점수 줌
>> * 상수 미리 정의
>> * column major `{0, 3, 6, 1, 4, 7, 2, 5, 8}`
>> * 32비트 컴퓨터는 4바이트단위로 다루기 때문에 배열을 4의 배수로 선언하는게 성능이 좋다.

#### 2. LINKING

> * `c파일`이 `executable file`이 되기까지 거치는 과정 중 가장 마지막 과정이 `Linking`이다.
> * `Linking`을 통해 `library`와 `object file`들을 하나의 `executable file`로 만들어준다.
> * `Linking`은 `Static Linking`과 `Dynamic Linking` 두 가지이다.
> * `Static Linking`: `executable file`을 만들 때, `library`를 `object file`에 붙인다. `cc -static` 명령어를 통해 가능
> * `Dynamic Linking`: 메모리에 load한 이후에 run time동안 `shared memory` 공간을 참조해서 참조 함수 또는 변수를 가져온다.

#### 3. Memory

> * `Memory`는 다섯 가지 층(`Instructions`, `Literals`, `Static Data`, `Dynamic Data(Heap)`, `Stack`)으로 나뉜다.
> * `Linux`는 모든 요소를 `file`처럼 관리한다. memory 영역도 마찬가지로, file permission에 의해 권한을 부여하고 관리함.
> * `Instructions`, `Literals`의 file permission은 `READ-ONLY`로 설정돼있고, 수정이 불가능하다.
> * `Static Data`, `Heap`, `Stack`의 file permission은 `WRITE-ONLY`로 설정돼있다.
> * `static`: `local variable` 앞에 붙을 경우 local 밖에서도 접근 가능한 변수로 바뀌고, `global variable` 앞에 붙을 경우 현재 소스코드 외부에서 extern으로 접근하는 것이 불가능한 변수로 바뀐다.
> * `Stack`: 메모리 맨 끝에서부터 역방향으로 자라나는 영역으로, 함수 실행에 따라 `local variable` 등등이 이 영역에 저장된다.
> * `Heap`: `malloc()`, `calloc()` 등으로 할당된 메모리 영역이다.
> * `function call overhead`: 같은 연산을 하더라도 function call로 구현하지 않은 것은 `instruction`영역에 이미 구현돼있기 때문에, 그냥 진행하면 되는 반면, function call로 구현된 것은 `Stack` 영역에 할당됐다가 해제됐다가를 반복하는 과정에서 overhead가 추가로 있는 것.
>> * 물론 여기서 `instruction`영역을 사용할 경우 overhead가 줄어들지만 저장공간은 function call을 활용할 때보다는 더 사용하게 된다.
> * function call 과정
>> 1. 함수가 호출되면, `Stack` 영역에 함수의 매개변수들이 올라가게 된다.
>> 2. 그 후 `return address`가 `Stack`영역에 올라가서 차후에 원래 routine으로 돌아갈 수 있도록 한다.
>> 3. `local variable`의 선언들도 `Stack`영역에 올라간다.
>> 4. 함수 실행이 끝나면, `Stack`영역에 올라간 부분들을 pop하면서 하나씩 빼고, 그 중 `return address`가 pop 됐을 때는 그 값을 토대로 원래 routine으로 돌아간다.
>> 5. 다만 `Stack`영역을 pop할 때, 영역의 값을 0으로 설정하고 pop하는 것이 아니기 때문에, 나중에 새로운 함수를 호출했을 때, 그 영역에 올라간 변수는 이전 함수의 쓰레기값을 받게 되는 것이다. 그래서 `local variable` 선언 시, 값을 초기화 해 주는 것이 안전하다고 할 수 있다.

#### 4. Type Basic

> * OS 버전마다 각 type의 크기가 다르다. 예를 들어 int가 반드시 32비트라는 것을 보장할 수 없다는 것이다.
> * `long long >= long >= int >= short >= char`이 원칙을 지키도록 설정만 됐을 뿐 각 type의 size를 정하지는 않았다는 것이다.

#### 5. Assignment

> * 고정소수점을 통해 곱셈을 세 가지 방식으로 구현하는 실습을 진행했다.
> * `fixed`라는 새로운 32비트 자료형을 선언하여 진행했고, 입력과 출력에 쓰이는 자료형을 `float`로 했다.
> * `FX_POINT`라는 매크로를 지정해서 8번째 비트를 기준으로 본인이 쓰는 정수부의 비트수와 소수부의 비트수를 분할했다.
> * `FX_QNUM` 매크로를 지정해서 `FX_POINT`에서 소수부의 비트수만 추출한 값을 저장하도록 했다.
> * 일반 `float` 소수 상태에서 `1 << FX_QNUM`을 해야 `fixed` 정수 비트와 소수 비트에 정수와 소수들이 위치할 수 있으므로 이를 이용해서 `float<->fixed`를 진행하도록 했다.
> * 곱셈은 세 가지 형태로 구현했다.
>> 1. `fx_mul_float`: 두 값을 `float`로 변환시켜 곱셈을 진행하고 다시 `fixed`로 변환하여 반환하는 방식.
>> 2. `fx_mul_long`: 두 값을 `long long`으로 변환시켜 곱셈을 진행하고, 이 때 `1 << FX_QNUM`이 두 번 곱해지는 꼴이 되므로 이 값으로 한 번 나눠서 반환하는 방식.
>> 3. `fx_mul_fast`: 위의 두 방식은 형 변환이 많이 들어가기 때문에, 이 overhead를 줄이고자 `FX_QNUM`을 절반 나눈 값으로 두 값을 `left shift` 진행해서 곱한 값을 반환하는 방식. overhead는 줄어들지만, 하위 비트들이 잘리기 때문에 정밀한 표현이 어려워 질 수 있다(오차가 커진다).
> * 보고서에 maxnum, minnum, resolution number 들어가야 함.

### 6번째 강의

#### 1. Pointers

> * Dynamic allocate
>> 1. `malloc()`: 힙에 메모리를 할당해서 반환해주는 함수
>> 2. `calloc()`: `malloc()`을 수행하면서 메모리 영역을 전부 0으로 초기화하는 함수
>> 3. `realloc()`: `malloc()`으로 할당된 메모리의 크기를 바꿔주는 함수
>> 4. `free()`: 힙에 할당된 메모리를 해제하는 함수
> * Noun-Adjective Form
>> * 상수 선언 시 `type const` 순서로 선언해준다.
>> 1. `int const`: `int`형 상수이다.
>> 2. `int const *`: `int const`형 포인터이다. 포인터가 가리키는 주소공간의 값을 변경하는 것은 불가능하고, 포인터의 주소를 변경하는 것은 가능하다.
>> 3. `int *const`: `int`형 포인터를 `const`로 선언한 것이다. 포인터가 가리키는 주소공간의 값을 변경하는 것은 가능하고, 포인터의 주소를 변경하는 것은 불가능하다.
>> - 이런식으로 해석하기 편해진다.
> * 포인터를 출력할 때는 `%p`를 사용하는 것이 좋다.
> * 실제로 `static` 선언한 `local variable`과 `global variable`의 주소를 출력했을 때, 둘이 메모리 상 연속해서 위치하는 것을 확인할 수 있다.
>> - `static` 선언한 `local variable`과 `global variable`은 모두 메모리의 `Static Data` 영역에 존재하기 때문이다.
> * 또한 한 함수 내에 선언한 두 `local variable`의 주소를 출력했을 때, 이때도 둘이 메모리 상 연속해서 위치하는 것을 확인할 수 있다.
>> - `loccal variable`은 메모리의 `Stack` 영역에 존재하기 때문이다.
> * `const`는 바꿀 수가 없지만, 여러 방법을 이용해서 바꿀 수 있는데, 그럼에도 사용하는 이유는 바꾸지 말자는 것을 명확히 명시하려는 이유에 있다.
> * 오전 실습
> ```
> void ma(char *buf, int size)
> {
> 	int i; 
> 	buf = (char *) malloc(size); 
> 	buf[0] = 'A';
> 	buf[1] = 0;
> 	printf("%p %p %s\n", buf, &buf, buf);
> }
> ```
> > - 이 함수를 실행했을 때 입력으로 들어온 `buf`와 `size`를 `Stack`영역에 새로 argument로 할당한다.
> > - 그 위치에 할당된 `buf`의 값을 `malloc()`을 통해 새 주소로 설정해도 이 함수를 call한 routine에서는 그 값(문자열의 주소)이 변하지 않기 때문에 > 이 `ma()`함수가 끝나도 원래 routine에서 printf를 해도 `"A"`가 출력되지 않는다.
> ```
> void mma(char **buf, int size)
> {
> 	int i; 
> 	*buf = (char *) malloc(size); 
> 	(*buf)[0] = 'A';
> 	(*buf)[1] = 0;
> 	printf("%p %p %s\n", buf, *buf, *buf);
> }
> ```
>> - 위 함수의 문제점을 개선한 것이 이 함수이다.
>> - `buf`를 이중 포인터로 선언했으므로, `buf`이 가리키는 값은 call routine에서의 `buf`의 값(문자열의 주소)일 것이다.
>> - 따라서 `*buf`를 수정할 경우 call routine에 영향을 줄 수 있고, `malloc()`이 반환한 값을 정상적으로 전달할 수 있게 된다.
> * `void *`
>> - 어느 주소던 가리킬 수 있는 장점이 있다.->형변환 해버리면 됨
>> - 하지만, 어느 타입으로 사용할지 알 수 없고, 한번에 몇 바이트를 이용하고 있는지 알 길이 없음
> * 함수포인터
>> * `int *f()`: `int *`를 반환하는 함수
>> * `int (*f)()`: `int`를 반환하는 함수의 포인터
>> * `int *(*f)()`: `int *`를 반환하는 함수의 포인터
함수명은 static에 선언
> * 포인터 관련된 문제
>> * `int *a, b`처럼 `*`을 변수명 앞에 붙여 가독성을 올린다. `a`는 포인터이고 `b`는 int형 변수임을 명확히 알 수 있다.
>> * `#define PINT int *`와 같은 매크로 선언은 위험하다. `PINT a, b`를 하면, `a`와 `b`가 모두 포인터가 되는 것이 아니고 위처럼 `a`는 포인터 `b`는 변수가 된다.
>> * `typedef int* PINT`로 선언하면 위의 매크로로 목표하고자 했던 목적을 달성할 수 있다. 이 경우 `PINT a, b`를 하면 두개 다 포인터로 선언된다.

#### 2. GDB

> * `gdb`를 사용하기 위해서는 반드시 `-g`옵션을 사용해야 한다. `-O`옵션은 꺼야한다.
> * `gdb "실행파일이름"`으로 실행 가능하다.
> * gdb 내에서 명령어를 입력하여 디버깅을 진행할 수 있다.
> * `list`: 소스코드를 확인할 수 있다. `list n1, n2`로 변형할 수 있고, 이는 `n1`줄부터 `n2`줄까지 소스코드를 보여준다.
> * `run`: 실행을 해볼 수 있다.
> * `where`: 에러가 난 줄 위치를 알 수 있다.
> * `break n`: n번째 줄에서 실행을 멈추도록 만들 수 있다.
> * `delete n`: n번째 break point를 삭제한다. n번째 줄이 아니고 break point의 발생 번호이다.
> * `print "변수명"`: 변수의 현재 값을 출력하도록 만들 수 있다.
> * `step`: `break`된 지점에서 다음 명령을 실행한다

#### 3. Optimization

> * 성능
>> - 속도: `CPU` > `Memory` > `Storage` > `I/O`
>> - `Register` > `Cache` > `Memory`
>> - `error`가 있으면 성능이 저하된다.
>> - `locality`를 살려서 코딩하면 성능을 올릴 수 있다. 유사한 데이터들을 가까운 메모리에 넣는 방식
>> - `Pipeline`을 통해 성능을 올린다. 한 instruction 수행을 여러 단계로 나누어서 각 단계마다 한개의 instruction을 잡고 수행하는 형태이다.
>>> 이렇게 될 경우, 거의 병렬에 가깝게 코드를 수행하는 것처럼 된다.
>>> 이 요소 때문에 성능을 위해서는 if문을 아끼는 것이 좋다.
> * `gcc optimization`
>> * `-O`명령어 family를 활용하여 최적화를 진행할 수 있다.
