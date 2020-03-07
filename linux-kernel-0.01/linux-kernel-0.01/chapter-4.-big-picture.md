# Chapter 4. Big picture

### 4.1. Step Wise Reﬁnement Of The 0.01 Kernel - Step 1

#### 4.1.1. The Source Tree

#### 4.1.2. The Makeﬁles

#### 4.1.3. The Big Picture

4.1.3.1. The Boot Sector

4.1.3.2. Kernel Initialization - Phase I

이 단계는 linux-0.01/boot/head.s의 코드에 해당한다. 이 단계부터, 386은 보호 모드로 작동된다. 보호 모드를 초기화하기 위해, real mode 코드\(boot.s\)는 임시 GDT와 IDT를 사용하였다. 따라서 이 단계에서는 **실제 GDT가 활성화되고 부분적으로 IDT**\(IDT를 할당하는 공간은 할당되지만, 각 엔트리는 모두 '비활성'\)를 설정한다. real mode 코드\(boot.s\)에서 인터럽트가 비활성화된 상태에서 \('cli'의 명령를 사용하여\) head.s 제어를 전달했다. 그래서 IDT가 완전하지 않아도 문제가 없다. 또한 커널이 초기에 **사용할 스택도 활성화**된다. 그런 다음 **페이징 메커니즘이 초기화되고 활성화**된다. 이 후, 제어는 linux-0.01/init/main.c 코드로 전달된다. 

이 시점까지의 모든 코드는 어셈블리 언어로 작성되었다. 표준 C를 사용할 수 없는 이유는, 기계에 종속적인 부분을 다루고 있기 때문이다. 물론 C를 사용하지 않으면 생사의 문제였다면 GCC의 인라인 어셈블리 기능를 사용하여 할 수 있었을 것이다. 그러나 C 컴파일러는 스택이 존재한다고 가정하기 때문에 여전히 스택이 초기화되지 않은 등의 문제가 있다. 그래서 어셈블리로 작성하는 것이 더 좋다. 

boot.s 코드는 부팅 섹터, 즉 512바이트에 맞도록 의도되었다. 0.01에서 boot.s 바이너리는 512바이트 미만이기 때문에 boot 섹터에 적합할 수 있다. 하지만 boot.s 파일이 512바이트를 초과하면 어떻게 될까? 새로운 커널에서는 512바이트를 초과하기에, boot.s를 boot.s와 setup.s라는 두 개의 파일로 분할한다. 이 두 파일 모두 8086 코드\(real mode code\)로 작성되어 있다. boot.s는 부팅 섹터에 들어간다. setup.s이 그 다음 섹터에 들어간다.  boot.s가 모든 일을를 마치면 setup.s로 진입하고, setup.s는 컨트롤을 헤드로 전송한다. boot.s의 크기가 증가하는 이유는 0.01에서 수행되지 않은 BIOS에서 몇 가지 파라미터를 읽고 일부 테스트를 수행하는 코드를 포함할 수 있기 때문이다. 예를 들어 플로피와 하드디스크의 종류와 같은 파라미터는 하드코딩되어 커널에 들어가는데, 이 파라미터는 BIOS에서 읽었을 수 있다. 이제 두 번째 초기화 단계로 가보자.

4.1.3.3. Kernel Initialization - Phase II

이 단계는 linux-0.01/init/main.c 파일의 코드에 해당한다. 우리는 이전 단계에서 IDT를 불완전하게 두었다고 언급했었다. 이 단계의 주요 과제는 다양한 인터럽트/예외에 대해 IDT 항목을 핸들러로 적절하게 채우는 것이다. 그 후에 인터럽트가 활성화된다\(cli 명령 사용\). 이제 인터럽트가 오면 IDT에 적절한 인터럽트 핸들러를 설치했기 때문에 적절히 처리될 것이다. 이것과 함께, 커널은 완전히 기능하고 사용자 프로그램에 서비스를 제공할 준비가 되었는가? 그러나 사용자 프로그램은 어디에서 오는가? 우리는 UNIX에서 어떤 프로그램이든 exec 시스템 콜을 사용하여 로드된다는 것을 안다. 일반적으로 exec은 fork와 함께 사용된다. 예를 들어 쉘에서 명령을 입력하면 쉘은 fork를 하여 자식 쉘을 생성하고 자식 쉘이 명령을 exec한다. 그래서 인터럽트를 초기화한 후 커널 자체가 fork하여 새로운 복사본을 만든다. 또한, 셸과 다른 프로그램들은 **File system**에 위치한다\(0.01의 경우 minix 버전 1 파일 시스템\). 따라서 **파일 시스템을 식별**해야 하고, 파일 시스템의 매개변수\(예: free blocks의 수 등\)를 결정해야 한다. 그래서 커널의 복사본은 **파일 시스템 초기화 작업**을 수행한다. 이 커널의 복사본은 1번 pid\(init 프로세스\)를 가지고 있으며, 원래 커널은 0번 pid를 가지고 있다. 이제 init 프로세스는 일반적인 fork exec 조합 기법에 의해 파일 시스템 검사기 및 쉘과 같은 몇 가지 다른 프로세스를 생성한다. 이제 0.01은 완전히 작동하게 되었다. 

init 프로세스가 실행될 때까지, 우리가 그때까지 실행하고 있던 코드가 부트 스트랩 로더에 의해 로드되었기 때문에 커널에 의해 디스크에서 메모리로 어떤 파일도 로드할 필요가 없었다는 것을 기억해야 한다. 그러나 fork나 exec과 같은 시스템 콜을 실행할 때, 또는 실행 시스템 콜을 실행할 때 새로운 파일이 메모리에 어떻게 로딩되는지에 대해서는 언급하지 않았다. 바로 이것들은 우리가 더 이야기할 사항들이다.

4.1.3.4. System Calls - The Interface To The Kernel

시스템 콜은 커널의 공통 데이터를 조작할 수 있도록 커널 내부의 루틴이다. 시스템 콜의 권한 수준은 커널 데이터를 수정할 수 있을 정도로 상승된다\(커널 데이터뿐 아니라 사용자 데이터까지 수정할 수 있다\). 시스템 콜은 커널 데이터를 조작하는 동안 문제가 발생하지 않도록 매우 신중하게 작성된다. 따라서 사용자 프로세스가 시스템 콜만 호출하도록 허용된다면, 그들이 커널 데이터를 수정할 수 있는 유일한 방법은 시스템 호출을 통해서고, 시스템 호출 자체가 완벽하다면 어떤 악의적인 프로세스도 해악을 끼칠 수 없다. 

시스템 콜을 구현하는 방법은 여러 가지가 있을 수 있다. 일반적인 방법은 소프트웨어 인터럽트를 사용하는 것이다. 우리는 인터럽트 핸들러가 하드웨어 인터럽트 또는 INT n 명령\(IDT\[n\]\)에 의해 실행될 수 있다는 것을 알고 있다. 그래서 리눅스에서는 시스템 콜에 INT 0x80을 사용한다. 즉, IDT의 0x80 항목을 커널의 특정 함수에 대응하도록 한다\(sched\_init 함수에서 수행 됨\). INT 0x80 명령어를 사용할 때\(사용자 프로그램으로부터\) IDT 항목에 해당하는 커널의 기능이 실행된다. 이 핸들러는 우리가 원하는 시스템 호출을 나타내기 위해 'ax' 레지스터를 통해 시스템 콜 번호를\(더 많은 인자를 가질 수도 있다\)넘겨주어 그 값에 따라 커널에서 적합한 함수를 호출한다. 시스템 호출이 실행된 후, 정상 함수처럼 호출된 지점으로 되돌아간다.

모든 절차를 간단히 말해보자. 커널이 가동된 후 프로세스가 실행되기 시작한다. 프로세스는 시스템 호출을 사용하여 일시적으로 더 높은 권한 레벨로 전환하고, 몇 가지 일반적인 커널 데이터를 접근한 후, 다시 되돌아온다.그리고 이 과정은 계속된다. 또한 우리는 프로세스가 시스템 콜을하고 일부 커널 데이터를 수정할 때, **시스템 콜을 요청한 프로세스가** 커널 데이터를 수정하고 있다는 것을 이해해야 한다. 즉, 커널은 자체적으로 지속적으로 실행되는 독립적인 엔티티가 아니다. 커널은 시스템 콜, 인터럽트/예외를 통해 실행된다. 

4.1.3.5. Multi Tasking And The Timer Interrupt

 타이머 인터럽트는 선점형 멀티 태스킹 운영 체제\(preemptive multi tasking operating system\)의 "심장"이다. 자, 선점형 멀티 태스킹 OS란 무엇인가? 사전적 의미로 선점이란 말은 "재산 등의 사전 압수, 전용 또는 어떤 것에 대한 청구"로 정의한다. 사전 압수란 OS 관점에서 선점을 위한 정확한 설명이다. 즉, 우리는 어느 누구도 원하는 만큼 오랫동안 그의 일을 하도록 허락하지 않는다는 것을 의미하며, 우리는 어떤 중재자에 의해 그 태스크가 일시적으로 중단되고 다른 태스크들이 일을 할 수 있도록 허용할 것을 허락한다. 그래서 선점형 멀티 태스킹 OS는 많은 프로세스가 동시에 실행될 수 있지만, 아무도 이 중재자 역할을 할 수는 없다. 

 OS를 올릴 수 있도록 하는 보드는 타이머 칩/회로가 있다. 이 칩/회로는 프로세서의 도움 없이 주기적으로 신호를 방출하도록 프로그래밍할 수 있다. 이 신호는 프로세서에 인터럽트를 발생시킨다. 타이머 인터럽트는 8259\(pic\)의 인터럽트 라인 0이고 우선 순위가 가장 높다. 즉, 다른 인터럽트가 기다리고 있는 것이 무엇이든 8259는 먼저 386에게 타이머 인터럽트에 대해 알린다.

그러면 OS는 타이머 인터럽트가 발생하면 무엇을 하는가? 

* 태스크가 실행된 시간을 나타내는 카운터를 증가시킨다.
* 태스크에 할당된 시간이 만료되면 태스크를 중지하고 다른 태스크를 실행시킨다. 

어떻게 다른 태스크로 전환할 수 있을까? 커널은 유저 모드에서 커널 모드로 전환할 때 태스크에 대한 세부사항을 설명하는 모든 데이터 구조를 가지고 있다. 이 정보는 TSS에 386에 의해 자동으로 저장된다. 그래서 태스크를 실행시키려면 작업 레지스터\(TR\)를 실행될 새로운 태스크의 TSS를 가리키기만 하면 된다. 

커널에는 태스크의 정보\(struct task\_struct\)를 저장하는 태스크 테이블\(struct task\_struct \*task\)이 있다. 새로운 태스크를 만들 때, 추가 항목이 테이블에 추가되고, 태스크가 종료되면, 한 항목이 테이블에서 삭제된다. 실행할 프로세스가 없으면 "idle 태스크"가 실행된다. 

따라서 타이머 인터럽터 , 새로운 태스크가 스케줄 된 경우\(필수는 아니지만\), 로드된 TSS는 새로운 태스크의 TSS가 되며 이전에 중단되었던 상태에서 재개된다. 

4.1.3.6. Other Interrupts

4.1.3.7. TLB exceptions
