
# Chapter 1 - 리눅스 소개와 전망

책의 학습흐름
커널 디버깅을 통해 커널을 사용 -> 소스코드 분석 -> 학습

## 유닉스 VS 리눅스 개념 정리
유닉스도 하나의 OS인데 비오픈소스이다. 리눅스는 유닉스에서 파생된 오픈소스로, 비슷한 구조, 형태라서 유닉스 계열이라 부른다. 

## 유닉스
멀티 태스킹, 멀티 유저를 지원하는 OS를 개발하고자 해서 생겼다.
근데 한 회사가 독점 -> 소송 당하고 패소 -> 유닉스를 팔게 됨 -> 유닉스 변종들의 탄생 -> 유닉스 표준을 만들어서 합치려고 함 (POSIX 규격 등장)

암튼 이런 상황에서 유닉스가 유료가 된 것에 불만을 가진 개발자들이 GNU 설립 -> 유닉스를 다시 구현 -> 완성도 UP -> 유닉스를 구동하는 핵심 코어 유닉스 커널위에 유닉스 유틸리티가 올라간 형태

GNU 뿐만 아니라 다른 기관도 유닉스 새로 개발 -> BSD 개발자도 유닉스를 다시 만들어서 Net/1 이라는 이름으로 무료 공개  -> 기존 유닉스 회사가 소송 -> 정체

## 리누스 토발즈 등장
핀란드 헬싱키 대학의 리누스 토발즈라는 학생이 GNU 시스템에 적합한 커널을 직접 개발 -> 당시 GNU 시스템을 위한 커널을 개발하던 다른 단체들이 리누스 토발즈의 시스템을 채택하는 결정 -> ==리눅스 탄생==

리눅스가 인기 많은 이유는 다양하지만 신기한 점 하나를 적어보면
각 기능마다 메일링 리스트를 통해 각 개발자에게 이슈를 보낸다. 이슈를 받은 개발자들이 서로 버그를 해결하는 패치를 제안하며 토론한다는 점이다. 

## 리눅스 개발 단체
1. 리눅스 커널 커뮤니티
   2주 간격으로 커뮤니티 소속 개발자에게 이슈나 패치 반영 사실을 메일로 보낸다. 
   책에서 알려주는 리눅스 커널의 최신 LTS 버전은 4.19인데 찾아보니 25.03.24 기준으로는 ==6.14==이다.
2. CPU 벤더 사
3. SoC 벤더 사
   System-On-Chip의 약자로, 하나의 컴퓨터 또는 전자 시스템들의 요소를 통합한 집적회로이다. 삼성전자, 인텔, 엔비디아 등이 있다. 
4. 보드 벤더 및 OEM
   라즈베리 파이 재단, 삼성 전자, LG 전자 등이 있다.
![[Pasted image 20250403172733.png]]


## 리눅스 개발을 위해서 알아야 할 것
1. 디바이스 드라이버
   인터럽트 핸들러 작성하는 법, 디바이스 파일에 open/read/write 함수 등록, 디바이스 트리 읽어서 디바이스 속성 저장하기 
2. 리눅스 커널
   커널에서 제공하는 시스템 콜을 알아야 시스템을 제어할 수 있다. 
3. CPU 아키텍처
   커널 코드는 어셈블리와 연관, 그러면 CPU 아키텍처를 알아야 한다. 
4. SoC
5. HAL 코드 구현 (HW Abstraction Layer) (뭔가 느낌이 OSI 7계층에서 개발하는 느낌)
6. 빌드 스크립트 구현
7. 테스트용 디바이스 드라이버 구현 
8. Git 같은 형상관리 툴

## 라즈베리파이와 리눅스 커널
### 라즈베리파이 spec
model - 라즈베리파이 3 모델 B
SoC - Broadcom BM2837 SoC
CPU - 1.2GHz ARM Cortex-A53 MP4
GPU - Broadcom VideoCore IV MP2 400 MHz
Memory - 1GB LPDDR2
SD card - Micro SD, push-pull type

책의 리눅스 커널 버전은 4.19
라즈비안 OS의 커널 소스는 리눅스 커널과 코드가 99% 같다. (디바이스 드라이버 제외)


# Chapter 2 - 라즈베리 파이 설정

라즈비안 굽고, 설치하는 건 생략

## 라즈베리 파이 커널 빌드
라즈비안에서 하단의 리눅스 커널의 코드를 수정하고 빌드해서 동작하는 것을 확인해볼 것이다.

터미널 명령어 모음집인 스크립트를 실행하는 방식으로,
리눅스 커널을 빌드해서 이미지 생성하고,
생성한 이미지를 설치하고 하는 법을 배운다. 


커널을 빌드하기 위해서 기본 유틸리티 프로그램을 설치해야 한다. 
컴파일러 시간에 배웠던 bison과 flex 가 있음을 확인할 수 있다. 

```shell
apt-get install git bc bison flex libssl-dev
```


라즈비안 최신 커널 소스를 github에서 git clone으로 다운 받는다. 
책에서는 당시 최신 버전인 rpi-4.19.y 버전의 branch를 사용한다. 나도 4.19 버전으로 진행할 예정이다. 

```shell
git clone --depth=1 https://github.com/raspberrypi/linux
```

이제 커널을 빌드할텐데 레퍼런스는 다음 사이트다. 
https://www.raspberrypi.org/documentation/linux/kernel/building.md

커널 빌드 작업은 시스템 명령어를 여러 개 입력해서 이루어진다. 
하나씩 입력하기 귀찮으니 쉘 스크립트 파일 하나 생성해두고 스크립트 파일을 실행하는 것으로 대체한다. 

커널 빌드를 위한 쉘 스크립트는 다음과 같다. 
chmod +x build_rpi_kernel.sh 로 실행권한을 준다.
주석으로 각 라인에 대한 설명을 추가했다. 
실행은 다음 명령어로 실행가능하다. 

```shell
./{스크립트 파일 이름}.sh
```

```shell
#!/bin/bash 

echo "configure build output path"

// 현재 작업 디렉터리를 KERNEL_TOP_PATH에 저장
KERNEL_TOP_PATH="$( cd "$(dirname "$0")" ; pwd -P )"
// 현재 디렉터리에 out 폴더 추가
OUTPUT="$KERNEL_TOP_PATH/out"
echo "$OUTPUT"

// 빌드 로그를 rpi_build_log.txt 파일에 저장하도록 지정한다. 
KERNEL=kernel7
BUILD_LOG="$KERNEL_TOP_PATH/rpi_build_log.txt"

echo "move kernel source"
cd linux

// 커널 config 파일 생성 코드
// config 파일을 참고해서 .config 파일을 생성한다. 
echo "make defconfig"
make O=$OUTPUT bcm2709_defconfig

// 리눅스 커널 빌드하는 명령어
echo "kernel build"
make O=$OUTPUT zImage modules dtbs -j4 2>&1 | tee $BUILD_LOG
```


빌드한 커널 이미지를 설치하는 쉘 스크립트는 다음과 같다. 

```shell
#!/bin/bash

KERNEL_TOP_PATH="$( cd "$(dirname "$0")" ; pwd -P )"
OUTPUT="$KERNEL_TOP_PATH/out"
echo "$OUTPUT"

cd linux

make O=$OUTPUT modules_install
cp $OUTPUT/arch/arm/boot/dts/*.dtb /boot/
cp $OUTPUT/arch/arm/boot/dts/overlays/*.dtb* /boot/overlays/
cp $OUTPUT/arch/arm/boot/zImage /boot/kernel7.img
```


커널을 빌드하면서 전처리 코드를 생성하는 방법을 배워보자. 
커널 소스코드를 보면 많은 매크로 함수들이 있는데, 이걸 풀어서 표현하는 것을 전처리 코드로 표현한다고 하는 것이다. 

1. 전체 전처리 파일을 추출
2. 특정 전처리 파일을 추출

전체 전처리 파일을 추출할 때는 다음과 같다. 
빌드할 때 make ~~ 라는 명령어를 사용하는데, 이때 사용되는 Makefile 이라는 이름을 파일을 수정하면 된다. 

![[Pasted image 20250507205324.png]]

9번째 줄에 보이는 것처럼 Makefile의 421번째 줄에 ==-save-temps=obj== 구문을 추가해주면 된다.
이걸 사용해서 빌드하면 out 폴더에 전처리 코드가 생성된다. 

전처리 코드의 파일명은 prefix가 ==.tmp_== 이고 suffix가 ==.i== 이다. 


특정 전처리 파일을 추출할 때는 다음과 같다. 
빌드 스크립트에 3줄만 추가해주면 된다. 

```shell 
PREPROCESS_FILE=$1
echo "build preprocessed file: $PREPROCESS_FILE"

make $PREPROCESS_FILE O=$OUTPUT zImage modules dtbs -j4 2>&1 | tee $BUILD_LOG
```

빌드 스크립트를 실행할 때, 명령어 뒤에 한 칸 띄우고 인자를 하나 넣어줄 수 있는데 그게 빌드 스크립트 내부에서 $1 키워드로 가져올 수 있다. 
해당 인자를 전처리를 할 파일명을 지정해주면 된다. 

```shell
./build_preprocess_rpi_kernel.sh linux/sched/core.i
```


## 리눅스 커널 소스의 디렉터리 구조와 의미

### arch 
아키텍처의 줄임말로, 아키텍처 별로 동작하는 커널 코드가 존재한다. 
- arm 
- arm64
- x86

### include 
커널 코드 빌드에 필요한 헤더 파일이 위치한다. 

### Documentation
커널 기술 문서가 존재. 개발자 대상이라서 이해하기 어렵다. 

### kernel
커널의 핵심 코드. 하위 디렉터리로 다음과 같이 있다. 
- irq : 인터럽트 코드
- sched : 스케줄링 코드
- power : 커널 파워 매니지먼트 코드
- locking : 커널 동기화 코드
- printk : 커널 콘솔 코드
- trace : ftrace 관련 코드

아까 arch랑 뭐가 다른가 싶다. 여기서는 아키텍처와는 무관한 커널 공통 코드를 의미한다. 

### mm
Memory Management의 약자로, 가상 메모리 & 페이징 관련 코드가 있다. 
이 또한 아키텍처와 무관하고, 아키텍처 관련 메모리 코드는 arch/`*`/mm/ 에 존재한다. 

### drivers
모든 시스템의 디바이스 드라이버 코드. 

### fs 
모든 파일 시스템 코드.

### lib 
커널에서 제공하는 라이브러리 코드.
똑같이 아키텍처 별 코드는 arch`*`/lib 에 존재한다. 


## objdump

objdump는 오브젝트 포맷의 파일을 조작할 수 있는 프로그램이다. 
우리는 리눅스 커널의 어셈블리와 섹션 정보를 보기 위해서 사용할 예정이다. 
대상 오브젝트 파일은 커널을 빌드했을 때 output 폴더에 생성되는 vmlinux 파일이다. 

```shell
objdump -x vmlinux | more
```

위 명령어는 vmlinux 파일의 헤더 정보를 확인하는 코드이다. 

```shell
objdump -d vmlinux
```

위 명령어는 vmlinux에서 어셈블리 코드를 출력하는 코드이다. 
근데 너무 많은 어셈블리 코드가 출력되어서 보기 어렵다. 

커널을 빌드하면 output 폴더에 System.map 이라는 파일이 같이 생성되는데,
이 파일은 어셈블리 코드 중에서 flag 같은 것으로, 특정 동작을 수행하는 어셈블리가 몇 번째에 위치하는지를 표시해둔 파일이다. 
열어보면 다음과 같은 형식이다. 
내용을 보면 schedule 관련한 어셈블리 코드의 주소가 0x807a0b6c ~ 0x807a0c14 임을 알 수 있다. 
![[Pasted image 20250507212044.png]]


```shell
objdump --start-address=[시작주소] --stop-address=[끝주소] -d vmlinux
```

위 코드는 시작 주소~끝 주소 사이의 어셈블리 코드를 출력해준다. 


# Chapter 3 - 커널 디버깅과 코드 학습


### 함수 호출 스택을 보는 법 

만약 커널 함수 function_A가 있을 때 이 함수가 언제 호출되는지 확인하고 싶으면 어떻게 할까? 

그냥 우리가 코딩하던 방식에서 생각해보면, 그냥 print 찍어 보면 된다. 
커널도 똑같다. 그냥 커널 소스 코드에서 print 로그 찍어서 언제 호출되는지 보면 된다. 

이렇게 호출 스택을 확인하는 것을 ==ftrace== 라고 한다. 

근데,
커널 코드가 엄청 많고 복잡한데 일일히 다 찾아서 소스코드 분석하면서 수정할 수 있나?

그게 싫어서 호출 스택을 확인할 수 있는 방법이 있다. 
ftrace를 설정하는 명령어를 스크립트로 작성해두고, 실행하면 된다.

예시는 다음과 같다. 
```shell
#!/bin/bash

echo 0 > /sys/kernel/debug/tracing/tracing_on
sleep 1
echo "tracing_off"

echo 0 > /sys/kernel/debug/tracing/events/enable
sleep 1
echo "events disabled"

echo  secondary_start_kernel  > /sys/kernel/debug/tracing/set_ftrace_filter
sleep 1
echo "set_ftrace_filter init"

echo function > /sys/kernel/debug/tracing/current_tracer
sleep 1
echo "function tracer enabled"

echo rpi_get_interrupt_info > /sys/kernel/debug/tracing/set_ftrace_filter
sleep 1
echo "set_ftrace_filter enabled"

echo 1 > /sys/kernel/debug/tracing/events/irq/irq_handler_entry/enable
echo 1 > /sys/kernel/debug/tracing/events/irq/irq_handler_exit/enable
echo "event enabled"

echo 1 > /sys/kernel/debug/tracing/options/func_stack_trace
echo "function stack trace enabled"

echo 1 > /sys/kernel/debug/tracing/options/func_stack_trace
echo 1 > /sys/kernel/debug/tracing/options/sym-offset
echo "function stack trace enabled"

echo 1 > /sys/kernel/debug/tracing/tracing_on
echo "tracing_on"
```

==echo {0 or 1} > {파일 위치}== 는 해당 파일을 끄고 (0) 켜기 (1)를 의미한다. 
==echo {함수명} > /sys/kernel/debug/tracing/set_ftrace_filter== 는 해당 함수가 호출되는 것을 filter로 등록해서 추적하겠다는 것이다. 

즉, 이 스크립트에서 중요한 부분은 ==echo rpi_get_interrupt_info > /sys/kernel/debug/tracing/set_ftrace_filter== 이다. 
이러면 rpi_get_interrupt_info 함수를 누가 호출했는지 알 수 있다. 

이 스크립트를 실행하면 ftrace 로그가 특정 디렉토리에 파일로 저장되는데
이걸 현재 디렉토리에 불러와서 확인해보고 싶다. 로그를 가져오는 스크립트는 다음과 같다. 

```shell
#!/bin/bash
echo 0 > /sys/kernel/debug/tracing/tracing_on
echo "ftrace off"
sleep 3
cp /sys/kernel/debug/tracing/trace .
mv trace ftrace_log.c
```

이 스크립트를 실행하면 ftrace의 log 파일이 현재 디렉터리에 ==ftrace_log.c== 라는 이름으로 저장된다. 
해당 파일을 열어서 확인해보면 `=>` 라는 표현으로 해당 함수를 호출한 함수가 어떤건지 이름이 나온다. 
아래로 갈수록 초반에 호출한 함수가 나온다. 

>[!tip] 현재 디렉터리에 있는 파일에서 특정 문자열을 검색하는 명령어는 다음과 같다. 
>egrep -nr {문자열} `*`


### 3.2. printk

우리가 콘솔에 무언가 출력할 때 printf를 쓰듯이 printk 는 커널 로그를 확인할 때 쓰인다. 

서식 지정자는 다음과 같다.
![[Pasted image 20250509021502.png]]

추가로, Pointer를 출력하고 싶으면 %p를 사용하면 된다.

printk를 실습해보기 위해서 함수 포인터를 디버깅하는 실습을 확인해보자. 

```c
static void insert_wq_barrier(struct pool_workqueue *pwq,
    struct wq_barrier *barr,
    struct work_struct *target, struct worker *worker)
{
    struct list_head *head;
    unsigned int linked = 0;
    
	printk("[+] process: %s \n", current->comm);
	printk("[+][debug] message [F: %s, L:%d]: caller:(%pS)\n",
	    __func__, __LINE__, (void *)__builtin_return_address(0));
}
```

8,9,10 번째 줄은 insert_wq_barrier 함수를 호출한 프로세스, 함수 정보를 출력하는 로직이다. 

current -> comm 에서 current는 현재 프로세스의 테스크 디스크립터 주소를 가르킨다. comm은 해당 프로세스의 이름이다. 
`__`func`__` 은 현재 실행 중인 함수의 이름이다. 
`__`LINE`__` 은 현재 실행 중인 코드 라인이다.
`__`builtin_return_address(0)`__` 은 현재 실행 중인 함수를 호출한 함수의 주소이다. 

위의 __ 로 감싸져 있는 코드들은 gcc 컴파일러에서 제공하는 매크로이다. 

이렇게 직접 printk 를 찍어서 로그를 확인해 볼수도 있다. 


### 3.3. dump_stack() 함수

dump_stack() 함수로도 커널 로그를 확인할 수 있다. 

```c
#include <linux/kernel.h> 
```

해당 헤더 파일을 추가해주어야 한다. 

인자와 리턴 값이 모두 void라서 그냥 호출만 해주면 된다.
호출만 해주면 ftrace 스택을 로그 파일에 저장해준다. 

### 3.4. ftrace 

앞선 두 방식은 모두 유용하지만 best practice는 아니다. 
만약 특정 함수가 자주 호출되는 함수라면 printk나 dump_stack이 엄청 호출되어서 로그를 확인하기 힘들거나, 시스템이 다운될 수 있다. 

커널 로그 확인의 best는 ftrace이다. 

ftrace를 쓰려면 ftrace 코드가 추가된 커널 소스를 빌드해서 설치해야 한다. 
config 파일에서 ftrace 설정에 대해서 활성화 작업을 해주어야 한다. 

근데, ftrace는 리눅스 커널의 공통 기능이라서 기본적으로 지원해준다. 라즈비안도 리눅스 기반이라서 설치만 하면 ftrace를 자동으로 지원해준다. 

>[!tip] 리눅스에선 보통 다음과 같은 경로에 ftrace 드라이버 설정 폴더와 파일이 존재한다.
>/sys/kernel/debug/tracing

우리는 3장 초반에 했던 방식처럼 스크립트를 작성하고 실행하는 방식으로 ftrace 설정을 해줄 계획이다.

```shell
echo 0 > /sys/kernel/debug/tracing/tracing_on
sleep 1
echo "tracing_off"

echo 0 > /sys/kernel/debug/tracing/events/enable
sleep 1
echo "events disabled"

echo  secondary_start_kernel  > /sys/kernel/debug/tracing/set_ftrace_filter
sleep 1
echo "set_ftrace_filter init"

echo function > /sys/kernel/debug/tracing/current_tracer
sleep 1
echo "function tracer enabled"

echo 1 > /sys/kernel/debug/tracing/events/sched/sched_wakeup/enable
echo 1 > /sys/kernel/debug/tracing/events/sched/sched_switch/enable

echo 1 > /sys/kernel/debug/tracing/events/irq/irq_handler_entry/enable
echo 1 > /sys/kernel/debug/tracing/events/irq/irq_handler_exit/enable

echo 1 > /sys/kernel/debug/tracing/events/raw_syscalls/enable
sleep 1
echo "event enabled"

echo  schedule ttw_do_wakeup > /sys/kernel/debug/tracing/set_ftrace_filter

sleep 1
echo "set_ftrace_filter enabled"

echo 1 > /sys/kernel/debug/tracing/options/func_stack_trace
echo 1 > /sys/kernel/debug/tracing/options/sym-offset
echo "function stack trace enabled"

echo 1 > /sys/kernel/debug/tracing/tracing_on
echo "tracing_on"
```

- 1, 36번 코드
	echo 0 > /sys/kernel/debug/tracing/tracing_on
	echo 1 > /sys/kernel/debug/tracing/tracing_on
	
	각각 트레이서를 활성화/비활성화 하는 명령어다. 
	tracing_on 파일에 0 을 넣으면 트레이서가 비활성화 되는 것이고,
	1을 넣으면 활성화 되는 것이다. 

- 13번 코드
	echo function > /sys/kernel/debug/tracing/current_tracer
	
	ftrace는 다양한 로그를 출력하는 방식에 따른 다른 타입의 트레이서를 지원한다. 
	1. nop : 기본 트레이서로, ftrace 이벤트만 출력한다.
	2. function : 함수 트레이서로, set_ftrace_filter 에 지정한 함수를 누가 호출했는지를 출력한다.
	3. function_graph : 함수 실행 시간과 세부 호출 정보를 그래프 포맷으로 출력한다. 
	
	우리 예시에선 function 트레이서를 사용한다. 

- 5, 17, 18, 20, 21, 23번 코드
	echo 0 > /sys/kernel/debug/tracing/events/enable
	
	모든 이벤트를 비활성화 하는 코드다. 
	
	echo 1 > /sys/kernel/debug/tracing/events/sched/sched_wakeup/enable
	echo 1 > /sys/kernel/debug/tracing/events/sched/sched_switch/enable
	echo 1 > /sys/kernel/debug/tracing/events/irq/irq_handler_entry/enable
	echo 1 > /sys/kernel/debug/tracing/events/irq/irq_handler_exit/enable
	echo 1 > /sys/kernel/debug/tracing/events/raw_syscalls/enable
	
	특정 event를 활성화하는 코드다. 

- 9, 27번 코드
	echo  secondary_start_kernel  > /sys/kernel/debug/tracing/set_ftrace_filter
	echo  schedule ttw_do_wakeup > /sys/kernel/debug/tracing/set_ftrace_filter
	
	function 트레이서의 대상 함수를 filter에 등록하는 코드다. 
>[!tip] 리눅스 커널에 존재하는 모든 함수를 filter에 등록할 수 있는 것이 아니라, available_filter_functions 파일에 포함된 함수만 지정할 수 있다. 
>/sys/kernel/debug/tracing/available_filter_functions

>[!warning] filter 설정을 하지 않으면 모든 함수를 트레이싱 하려고 시도해서 시스템이 다운된다.

- 32,33번 코드
	echo 1 > /sys/kernel/debug/tracing/options/func_stack_trace
	echo 1 > /sys/kernel/debug/tracing/options/sym-offset
	
	각각 트레이서의 option을 설정해주는 것이다. 
	설정 파일에 1을 넣어줌으로써 활성화하는 것이다. 
	func_stack_trace 옵션은 필터로 지정된 함수의 콜 스택을 기록하는 옵션이다.
	sym-offset 옵션은 함수를 호출한 주소를 같이 출력하는 옵션이다. 16진수 형태로 뒤에 붙는다. 
	
	그 외에도 다양한 설정 파일이 있고, 활성화 시키려면 1을 echo 해주면 된다. 


### ftrace 로그 해석하는 법

![[Pasted image 20250509030533.png]]

ftrace 로그의 기본 포맷은 위의 사진과 같다. 

d... 로 되어있는 컨텍스트 정보는 세부적으로 다음과 같다. 
![[Pasted image 20250509031322.png]]
![[Pasted image 20250509031337.png]]


```log
chromium-browse-1436 [002] d...  9445.131875: sched_switch: prev_comm=chromium-browse prev_pid=1436 prev_prio=120 prev_state=S ==> next_comm=kworker/2:3 next_pid=1454 next_prio=120
```

위 로그는 가장 대표적인 이벤트인  sched_switch 이벤트의 로그이다. 
로그의 헤더는 앞에서 포맷으로 설명했고, 뒤의 세부 내용은 다음과 같다. 

![[Pasted image 20250509031459.png]]

해석하면 =="kworker/2:1" 프로세스에서 "ksoftirqd/2" 프로세스로 스케줄링하는 동작을 출력한다==는 뜻이다.

```log
kworker/0:1-31 [000] d.h. 592.790968: irq=17 name=3f00b880.mailbox
kworker/0:1-31 [000] d.h. 592.791016: irq_handler_exit: irq=17 ret=handled
```

또 다른 예시로, 위 로그는 대표적인 이벤트인 irq_handler와 irq_handler_exit 이벤트의 로그이다. 

첫 번째 로그를 해석하면 ==pid가 31인 kworker/0:1 프로세스가 실행되는 도중 17번 인터럽트가 발생했다==이다. 
두 번째 로그는 ==17번 3f00b880.mailbox 인터럽트에 대한 핸들러를 끝냈다==이다. 

### 이벤트를 출력하는 함수 이름 규칙

특정 이벤트를 출력하는 커널 함수에는 규칙이 있다. 
=="trace_" + "ftrace_event_name"== 이다. 

![[Pasted image 20250509032348.png]]

즉, 만약 sched_switch 이벤트를 어떤 파일에서 발생시키는지 소스코드를 뜯어 보고 싶다면 위에서 배운 egrep 명령어를 사용하면 된다. 

```shell 
egrep -nr trace_sched_switch *
```

