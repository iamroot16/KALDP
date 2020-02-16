---
description: '"The Linux Kernel 0.01 Commentary" 문서를 기반합니다.'
---

# Linux Kernel 0.01

 해당 문서는 오랜 시간 전의 리눅스 커널에 대해 설명하고 있다. 운영체제, 하드웨어에 관심이 있는 독자라면  해당 문서를 통해, 80386 아키텍처에 대해 배울 수 있으며, 기본적인 운영체제 구현에 대해 배울 수 있습니다.

 해당 문서의 본 저자들과 나 모두 숙달된 커널 해커는 아니기에 기술적으로 완벽하지 않을 수 있습니다. 따라서 독자 분들의 많은 양해 부탁드립니다.

### Environment

 해당 버전은 다음과 같은 환경에서 실험되었습니다.

```bash
cc (Ubuntu 7.4.0-1ubuntu1~18.04.1) 7.4.0
GNU assembler (GNU Binutils for Ubuntu) 2.30
GNU ld (GNU Binutils for Ubuntu) 2.30
```

### Todo

* 환경에 맞춰 빌드할 수 있도록 파일을 수정한다.
  * 컴파일 오류 ☑
  * 어셈블 오류 ☑
  * 링크 오류 ☑
  * 이미지 파일 생성 오류 ☑
* 현황에 맞춰 내용을 수정 및 번역한다.\(ongoing\)
  * Chapter 1. Getting Started ☑ 
  * Chapter 2. Building Kernel 🔄 
  * Chapter 3. Processor Architecture
  * Chapter 4. The Big Picture
  * Chapter 5. Flow of control through the Kernel
  * Chapter 6. Journey to the Center of the Code

### Appendix\(계속 수정 중\)

* 링커 스크립트 추가 설명
* binutils의 각종 옵션들 설명
* qemu + gdb 설명\(+ real mode debugging\)

### Original File

{% file src="../../.gitbook/assets/0.01.pdf" caption="The Linux Kernel 0.01 Commentary 원본" %}



