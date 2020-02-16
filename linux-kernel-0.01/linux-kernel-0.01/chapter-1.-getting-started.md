# Chapter 1. Getting Started

## 1.1. Introduction

### 1.1.1. Copyright and License

Copyright \(C\) 2003 Gopakumar C.E, Pramode C.E This document is free; you can redistribute and/or modify this under the terms of the GNU Free Documentation License, Version 1.1 or any later version published by the Free Software Foundation. A copy of the license is available at www.gnu.org/copyleft/fdl.html.

### 1.1.2. Reading 0.01 source

리눅스 커널 버전 0.01은 현재의 커널과 비교하면 굉장히 작은 프로그램이다. 커널을 처음 접하는 사람은,

* 최신 버전 커널의 큰 프로그램의 일부를 읽고 이해해야 하는지,
* 옛날 버전의 더 작고 복잡한 커널을 거의 전체적으로 읽고 이해해야 하는지

와 같은 고민에 직면하게 된다. \(물론 시간과 열정이 많은 사람이라면 둘 다 할 수 있다!\) \*만약 누군가 어떤 방식을 하는게 좋겠냐고 묻는다면, 나는 후자를 추천하겠다. 최신의 커널을 분석한다면, 커널 코어까지 진입하기에 많은 시간이 걸리게 된다. 따라서 빠르고 쉽게\(?\) 커널의 전반적인 개념을 익힌 후, 최신의 커널에 도전하는 것을 추천한다.\*

### 1.1.5. 선수 지식

독자들은 다음과 같은 기술/개념에 대해 익숙해야 한다.

* 리눅스/유닉스 시스템 프로그래밍, 시스템 콜 인터페이스
* Operating System\(유저 모드와 커널 모드, 스케줄링, 메모리 관리\)
* 808386 architecture\(레지스터, 보호 모드, 가상 메모리, 인터럽트\)
* 컴파일러, 어셈블러, 링커

알아야 할 것과 알아야 할 것을 정확히 열거하는 것은 꽤 어렵다. 유일한 방법은 이 책을 진행하면서 모르는게 나오면 그것을 공부하는 것이다. 위에 필요한 사항을  먼저 학습하는 방식은 추천하지 않는다. **80386 아키텍처를 공부하기 위해 엄청난 양의 매뉴얼을 학습하는 과정은 지루하고 비능률적일 수 있다**. 왜냐하면 필요한 정보는 사실 일부에 불과하기 때문이다. 이 문서에서 찾을 수 없는 것들은 아래에 언급된 참고 자료 중 하나에서 찾을 수 있다.

1. The C programming language by K & R
2. **Design of the Unix Operating System by Maurice.J.Bach**
3. Advanced Programming in the Unix environment by W. Richard Stevens
4. The info and man pages \(mainly on gcc, as, ld, as86, ld86\)
5. The Intel 80386 manuals
6. The Indispensable PC Hardware Book by Hans-Peter Messmer

2번 책같은 경우, 굉장히 중요한게 리누스 토발즈가 해당 책을 기반으로 0.0.1버전을 구성했기 때문이다.\(\*물론 꽤나 어렵다\*\) 3번 책은 Unix programming의 레퍼런스 북으로 없이도 충분히 가능하다. 하지만 6번 책은 다른 PC 동작에 대한 레퍼런스 북이 없다면 꼭 읽어보는게 좋다. 또한 80386 아키텍처에 관한 내용은 뉴얼에서 참고하는 것이 제일 좋다. 해당 매뉴얼들은 인텔 매뉴얼 URL에서 다운받을 수 있다. 해당 문서에 나오는 그림들은 "Intel Architecture - Software Developer’s Manual 3"에서 나온 것이다. 해 문서는[http://x86.ddj.com/intel.doc/386manuals.htm](http://x86.ddj.com/intel.doc/386manuals.htm)에서 다운받을 수 있다.

### 1.1.5. 어떻게 읽어야 하는가?

\*추후 수정\*

