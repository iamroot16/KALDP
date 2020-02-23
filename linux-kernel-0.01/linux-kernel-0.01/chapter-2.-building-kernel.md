# Chapter 2. 커널 빌드

## 2.1. Getting 0.01 Up And Running

저자들은 0.01을 실행하기 위해 거의 2주일이 걸렸다. 다양한 정보와 매뉴얼 페이지를 훑어보고 마침내 실행시킬 수 있었다. 그 과정은 그 자체로 매우 교육적이었다. 그러니 서두르지 말고 실행시키기 위해 정확히 무엇을 하는지 그리고 그 이유를 잘 이해해야 한다.

0.01 코드를 다운받고 make 명령어를 실행시켜 보자. 대부분 오류를 내며 빌드에 실패할 것이다. 해당 소스를 성공적으로 빌드를 하려면 먼저 make 파일을 수정하도록 하자.

빌드 환경은 다음과 같다.

```bash
cc (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
GNU assembler (GNU Binutils for Ubuntu) 2.30
GNU ld (GNU Binutils for Ubuntu) 2.30
```

Makefile을 우선 살펴보자.

```bash
AS86    =as -0 -a
CC86    =cc -0
LD86    =ld -0

AS      =gas
LD      =gld
LDFLAGS =-s -x -M
CC      =gcc
CFLAGS  =-Wall -O -fstrength-reduce -fomit-frame-pointer -fcombine-regs
CPP     =gcc -E -nostdinc -Iinclude

ARCHIVES=kernel/kernel.o mm/mm.o fs/fs.o
LIBS    =lib/lib.a
```

우선 binutils에 알맞는 커맨드에 맞춰야 한다.

| Before | After |
| :--- | :--- |
| AS=gas | AS=as |
| LD=gld | LD=ld |

본래의 코드로 시작해 스스로 수정하고, 정말로 문제가 있을 때만 아래에 작성한 내용을 참고하길 바란다. 우리가 더 쉽게 할 수 있었는지는 모르지만, 우리가 패치한 코드는 잘 작동한다. 우리는 우리가 진행한 패치는 다음과 같다.

* 리누스가 작성한 것으로 보아, 이전 버전의 gcc는 프로그램에 선언된 모든 변수 이름에 밑줄\('\_'\)을 접두사를 붙이곤 했다고 추측한다. 따라서 C 코드와 연결된 어셈블리 파일에서는 외부 변수를 참조할 때 \_xxx 유형의 이름으로 접근한다. 하지만 현재의 gcc 컴파일러들은 그런 일을 하지 않는다. 그래서 당신은 모든 어셈블리 파일, 헤더 파일, 그리고 인라인 어셈블리를 사용하는 부분을 살펴봐야 하고 변수 이름의 밑 줄을 삭제해야 한다. 어떤 파일을 조사해야 할지는 링크 오류가 알려줄 것이다.
* boot/boot.s안의 주석 심볼  "\|"은  ";"로 변경해야 한다.
* 옛날의 링커, 어셈블러에 대한 옵션은 새 링커, 어셈블러와 동일하지 않다. 따라서 링커의 경우 새로운 옵션 -r, -T 옵션을 추가로 주어야 한다. -r 옵션은 출력 파일이 relocatable하게 만드는 옵션이고, -T 옵션은 링커 스크립트로 링크를 하게 만드는 옵션이다. 따라서 작은 링크 스크립트로 출력 파일을 생성하되, relocatable하게 만든다. 이 스크립트는 출력 파일인 'system'에서 모든 헤더를 제거하여 절대 주소 0x0을 기준으로 'system'의 코드를 만든다. 또한 'system' 파일이 메모리에 로드될 때 bss 블록에 있을 변수를 위한 공간이 제공되어야 하기 때문에 다시 연결할 수 있는 형태의 시스템에 존재하는 bss 블록을 확장한다.

-r : 오브젝트 파일을 relocatable하게 만드는 옵션이다.

-T: 링커 스크립터로 링크를 하게 만드는 옵션이다.

* 문제. include 파일 내의 헤더 파일에 함수 선언이 들어가 중복 정의가 발생한다.\(extern inline vs static inline\)
* 문제. inline assembly의 clobber list에서 오류가 발생한다.
* 문제. symbol resolution 



수정 중...



