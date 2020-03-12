# Chapter 5. Flow of control through the Kernel

1단계에서 우리는 현재 일어나고 있는 일의 큰 그림을 모두 한 그림에서 보았다. 이제 그림을 좀 더 멀리서 보자. 여기서는 1단계에서처럼 아주 상세하게 설명하지 않을 것이다. 우리는 0.01 커널이 부팅되는 동안과 후에 어떤 코드에 어떤 코드가 포함되어 있는지 그리고 어떤 코드 시퀀스가 뒤따르는지 간략히 언급할 것이다. 따라서 이 단계가 완료되면 "파일 이름" 수준에서 코드의 흐름을 숙지할 수 있을 것이다. 3단계에서는 "파일 이름"이 아닌 "기능 이름"에 의해 코드 경로를 추적할 수 있도록 각 파일의 코드를 설명할 것이다 :-\)

## 5.1.1. Normal Activities In a Running OS

OS는 부팅 후 무엇을 하는가? 주요 활동은 인터럽트/예외를 핸들링하는 것이며, 이를 통해 다른 모든 추상적 활동이 이루어진다. 예를 들어, OS가 가동되고 셸이 실행된 후, 사용자는 키보드가 인터럽트를 생성하는 것에 해당하는 명령을 입력할 수 있다. 쉘은 그러한 문자를 읽고, 명령으로 해석하며, 필요한 경우 시스템 호출을 하는데, 이 역시 OS에 의해 처리되는 인터럽트의 한 형태다. 또는 타이머 인터럽트를 처리하면서 새로운 태스크가 스케줄링될 수  있다.

## 5.1.2. linux/boot Directory

### 5.1.2.1. Boot Up - boot/boot.s

우리는 0.01을 빌드하여, 0x00을 시작 주소\(엔트리 포인트\)로 설정된 이미지를 가지게 되었다. 그러나 누가 이 코드를 커널 이미지를 0x00에 올려주는가? 바로 이 파일에서 그렇게 만들어준다. 386 프로세서는 전원이 켜지면, 고정된 시작 주소에서 실행된다\(0xFFFFFFF0\) 그런다음, ROM의 시작주소로 JMP하게 된다. boot ROM 코드는 각종 디바이스에\(floppy, hard disk\) 부팅 가능한 이미지가 있는지 확인한다. 우리의 경우는 플로피 디스크를 하고, 첫 섹터에 시그니처 값\(0xAA 0x55\)이 있으면, 해당 섹터를 고정된 위치\(0x7c00\)에 로드하고 해당 위치로 JMP한다. 이렇게 로드된 부분을 부트 로더라 부른다.  부트 로더는 0.01 소스에 일부이며, 입맛대로 수정할 수 있다. 부트 로더는 플로피 디스크에서 커널 이미지를 메모리에 로드해야 한다. 커널 이미지를 로드할때 0x00로 복사를 하며 복사가 완료된 후, 그 주소로 JMP한다. 또한 JMP하기 전에 real 모드에서 protected 모드로 전환하는 일도 수행한다. 

### 5.1.2.2. The “head” Of The Kernel - boot/head.s

커널 코드는 head.s로 시작하게 된다\(이게 바로 head.s라 불리는 이유다\). 이전 섹션에서 얘기했듯이 head.s로 진입히기 전에 더미 IDT, GDT, LDT를 세팅하고 protected모드로 전환 된다. 따라서 우리는 이런 더미 GDT, LDT, IDT를 제대로 세팅하는 것이다. 또 다른 과제는 메인 커널 코드를 실행하기 전에 페이징을 활성화 해야 한다. 이런 모든 과정이 수행되어야 인터럽트를 활성화 할 수 있고, 메인 커널 코드를 실행할 수 있다. 

## 5.1.3. linux/init Directory

### 5.1.3.1. The “main” Kernel - init/main.c

main.c 파일은 이름대로 main 함수를 포함하고 있다. head.s에서 main\(\)함수로 JMP하게된다. main 함수는 커널이 실제로 동작하기 전에 필요로 하는 초기화 과정을 수행한다\(예를 들어 콘솔, 하드 디스크, 파일 시스템\). 이러한 것들이 준비가 되서야 멀티 태스킹을 수행할 수 있다. 이제 모든 준비가 되었다면, 메인 함수는 fork를 통해 새로운 프로세스\(init  프로세스\)를 생성한다. 이 모든 과정을 수행하면,  0.01은 완전한 기능을 수행할 수 있다. 사용자 프로그램을 실행하며, 태스크를 스케줄링하고,  시스템 콜을 서비스할 것이다. 

## 5.1.4. linux/kernel Directory

### 5.1.4.1. Software int 0x80 - kernel/system\_call.s

### 5.1.4.2. System Calls - kernel/sys.c

### 5.1.4.3. Exceptions - kernel/asm.s, kernel/traps.c

### 5.1.4.4. Console Functions - kernel/console.c, kernel/tty\_io.c

### 5.1.4.5. Miscellaneous Files - kenel/panic.c, kernel/mktime.c, kernel/vsprintf.c, kernel/printk.c

### 5.1.4.6. Device Handling Code - kernel/hd.c, kernel/keyboard.s, kernel/rs\_io.s

### 5.1.4.7. Important System Calls - fork\(\) & exit\(\) - kernel/fork.c, kernel/exit.c

### 5.1.4.8. The “heart” of the Kernel - scheduler - os/sched.c

## 5.1.5. linux/mm Directory

### 5.1.6.1. fs/super.c

### 5.1.6.2. fs/buffer.c

### 5.1.6.3. fs/bitmap.c, fs/inode.c, fs/namei.c, fs/truncate.c

### 5.1.6.4. fs/block\_dev.c, fs/file\_dev.c, fs/char\_dev.c

### 5.1.6.5. fs/file\_table.c

### 5.1.6.6. fs/open.c, fs/read\_write.c

### 5.1.6.7. fs/fcntl.c, fs/ioctl.c, fs/tty\_ioctl.c, fs/stat.c

### 5.1.6.8. fs/pipe.c

### 5.1.6.9. fs/exec.c

## 5.1.7. linux/lib Directory

## 5.1.8. linux/include Directory

## 5.1.9. linux/tools Directory

