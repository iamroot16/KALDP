# Appendix B. String.h inline assembly

```c
static inline char * strcpy(char * dest,const char *src)
{
	int d0, d1;
	__asm__ __volatile__("cld\n"
		"1:\tlodsb\n\t"
		"stosb\n\t"
		"testb %%al,%%al\n\t"
		"jne 1b"
		:"=&S"(d0),"=&D"(d1)
		: "0" (src),"1" (dest)
		:"ax");
	return dest;
}
```

* **Line 3**: **cld**\(clear direction flag\) EFLAGS의 DF 플래그를 클리어한다. DF 플래그가 0이라면, string 오퍼레이션에서 인덱스 레지스터\(ESI and/or EDI\)가 증가한다.
* **Line 4**: **lods**\(load string\) ds:esi에서 1/2/4바이트를 AL, AX 또는 EAX 레지스터에 로드한다. 읽은 만큼 esi 레지스터 값이 DF플래그에 따라 증가 또는 감소한다.
* **Line 5**: **stos**\(store string\) AL, AX 또는 EAX 레지스터에서 1/2/4 바이트를 es:edi/di에 store한다. 저장한 만큼 edi 레지스터 값이 DF플래그에 따라 증가 또는 감소한다.
* **Line 6~7**: 읽은 값\(ax\)가 NULL이 아니면 1번 라벨로 돌아간다.
* **Line 8:**  이하 생략
  * input: esi\(dummy0\), edi\(dummy1\)
  * output: esi\(src\), edi\(dest\)
  * clobber: ax

{% hint style="warning" %}
원본 코드에는 input 레지스터가 clobber에 들어가 있지만, 지금은 그와 같은 문법을 허용하지 않는다. 해당 [링크](https://stackoverflow.com/questions/48381184/can-i-modify-input-operands-in-gcc-inline-assembly)를 참고하라.
{% endhint %}

```c
static inline char * strncpy(char * dest,const char *src,int count)
{
	int d0, d1, d2;
	__asm__ __volatile__("cld\n"
		"1:\tdecl %2\n\t"
		"js 2f\n\t"
		"lodsb\n\t"
		"stosb\n\t"
		"testb %%al,%%al\n\t"
		"jne 1b\n\t"
		"rep\n\t"
		"stosb\n"
		"2:"
		:"=&S" (d0),"=&D" (d1),"=&c" (d2)
		:"0" (src), "1" (dest), "2" (count)
		:"ax");
	return dest;
}
```

* **Line 4~5**: count의 값을 1 감소 시킨다. 음수라면, 2번 라벨로 점프한다.
* **Line 6~9**: strcpy의 내용과 동일
* **Line 10~12**: 남은 count만큼 stosb 반복한다. 즉 count만큼 NULL을 edi에 복사한다.

```c
static inline char * strcat(char * dest,const char * src)
{
	int d0, d1, d2, d3;
	__asm__("cld\n\t"
		"repne\n\t"
		"scasb\n\t"
		"decl %1\n"
		"1:\tlodsb\n\t"
		"stosb\n\t"
		"testb %%al,%%al\n\t"
		"jne 1b"
		:"=&S"(d0), "=&D"(d1), "=&a"(d2), ="=&c" (d3)
		:"0" (src),"1" (dest),"2" (0),"3" (0xffffffff));
	return dest;
}
```

* **Line 3:** DF 플래그 클리어.
* **Line 4~6**:
  * **repne**: zero flag가 0일때 동안, 
  * **scas**\(scan string\) ax/al/eax를 edi 레지스터의 값과 비교\(cmp\)하여 플래그를 설정한다.
  * 즉, src의 NULL 위치 바로 전까지 이동한다는 말이다.
* **Line 7~10**: strcpy와 동일

```c
static inline char * strncat(char * dest,const char * src,int count)
{
	int d0, d1, d2, d3;
	__asm__("cld\n\t"
		"repne\n\t"
		"scasb\n\t"
		"decl %1\n\t"
		"movl %8,%7\n"
		"1:\tdecl %7\n\t"
		"js 2f\n\t"
		"lodsb\n\t"
		"stosb\n\t"
		"testb %%al,%%al\n\t"
		"jne 1b\n"
		"2:\txorl %2,%2\n\t"
		"stosb"
		:"=&S" (d0), "=&D" (d1), "=&a" (d2), "=&c" (d3)
		:"0" (src),"1" (dest),"2" (0),"3" (0xffffffff),"g" (count)
		);
	return dest;
}
```

* **Line 4:** DF 플래그 클리어.
* **Line 5~7**:
  * **repne**: zero flag가 0일때 동안, 
  * **scas**\(scan string\) ax/al/eax를 edi 레지스터의 값과 비교\(cmp\)하여 플래그를 설정한다.
  * 즉, src의 NULL 위치 바로 전까지 이동한다는 말이다.
* **Line 8**: ecx를 인자로 받은 count로 세팅한다.
* **Line 9~14**: ecx를 감소시키며 1바이트를 복사한다\(esi -&gt; edi\)

```c
static inline int strcmp(const char * cs,const char * ct)
{
	int d0, d1;
	register int __res __asm__("ax");
	__asm__("cld\n"
		"1:\tlodsb\n\t"
		"scasb\n\t"
		"jne 2f\n\t"
		"testb %%al,%%al\n\t"
		"jne 1b\n\t"
		"xorl %%eax,%%eax\n\t"
		"jmp 3f\n"
		"2:\tmovl $1,%%eax\n\t"
		"jl 3f\n\t"
		"negl %%eax\n"
		"3:"
		:"=a" (__res, "=&D" (d0),"=&S" (d1)
		:"1" (cs), "2" (ct));
	return __res;
}
```

* **Line 5:** DF 플래그 클리어.
* **Line 6~7**: ds:esi에서 1바이트를 ax에 로드하고 es:edi의 값과 cmp한다.
* **Line 8**: 같지 않다면 2번 라벨로 점프한다.
* **Line 9~12**: ax가 NULL인지 확인한다. 아니라면 1번 라벨로 점프한다. 맞다면, eax를 0으로 세팅하고 리턴한다.
* **Line  13~15**: 1을 리턴할지, 1을 리턴할지 결정한다.

```c
static inline int strncmp(const char * cs,const char * ct,int count)
{
	int d0, d1, d2;
	register int __res __asm__("ax");
	__asm__("cld\n"
		"1:\tdecl %6\n\t"
		"js 2f\n\t"
		"lodsb\n\t"
		"scasb\n\t"
		"jne 3f\n\t"
		"testb %%al,%%al\n\t"
		"jne 1b\n"
		"2:\txorl %%eax,%%eax\n\t"
		"jmp 4f\n"
		"3:\tmovl $1,%%eax\n\t"
		"jl 4f\n\t"
		"negl %%eax\n"
		"4:"
		:"=a" (__res), "=&D" (d0), "=&S" (d1), "=&c" (d2)
		:"1" (cs),"2" (ct),"3" (count));
	return __res;
}
```

* **Line 5:** DF 플래그 클리어.
* **Line 6~7**: count를 감소하고 음수라면 2번 라벨로 점프한다.
* **Line 8~9**: ds:esi에서 1바이트를 ax에 로드하고 es:edi의 값과 cmp한다.
* **Line 10**: 같지 않다면 3번 라벨로 점프한다.
* **Line 11~14**: ax가 NULL인지 확인한다. 아니라면 1번 라벨로 점프한다. 맞다면, eax를 0으로 세팅하고 리턴한다.
* **Line  15~17**: 1을 리턴할지, 1을 리턴할지 결정한다.

```c
static inline char * strchr(const char * s,char c)
{
	int d0;
	register char * __res __asm__("ax");
	__asm__("cld\n\t"
		"movb %%al,%%ah\n"
		"1:\tlodsb\n\t"
		"cmpb %%ah,%%al\n\t"
		"je 2f\n\t"
		"testb %%al,%%al\n\t"
		"jne 1b\n\t"
		"movl $1,%1\n"
		"2:\tmovl %1,%0\n\t"
		"decl %0"
		:"=a" (__res), "=&S" (s)
		:"1" (s),"0" (c));
	return __res;
}
```

* **Line 5:** DF 플래그 클리어.
* **Line 6**: c를 ah으로 이동
* **Line 7~8**:  ds:esi로부터 1바이트를 읽고, al에 저장하고 ah와 비교한다.
* **Line 9**: 동일하면 2번 라벨로 점프한다.
* **Line 10~11**: NULL에 도달할 때까지 1번 라벨로 점프한다.
* **Line 12~14**: 못찾으면 0을 리턴하고, 찾았다면 찾은 주소\(esi\)를 리턴한다.

```c
static inline char * strrchr(const char * s,char c)
{
	int d0, d1;
	register char * __res __asm__("dx");
	__asm__("cld\n\t"
		"movb %%al,%%ah\n"
		"1:\tlodsb\n\t"
		"cmpb %%ah,%%al\n\t"
		"jne 2f\n\t"
		"movl %%esi,%0\n\t"
		"decl %0\n"
		"2:\ttestb %%al,%%al\n\t"
		"jne 1b"
		:"=d" (__res), "=&S" (d0), "=&a" (d1)
		:"0" (0),"1" (s),"2" (c));
	return __res;
}
```

* **Line 5:** DF 플래그 클리어.
* **Line 6**: c를 ah으로 이동
* **Line 7~8**:  ds:esi로부터 1바이트를 읽고, al에 저장하고 ah와 비교한다.
* **Line 9**: 동일하지 않으면 2번 라벨로 점프한다.
* **Line 10~11**: 발견한 주소를 \_\_res에 저장한다.
* **Line 12~14**: NULL에 도달하지 않았다면 1번 라벨로 돌아간다.

```c
static inline int strspn(const char * cs, const char * ct)
{
	int d0, d1;
	register char * __res __asm__("si");
	__asm__("cld\n\t"
		"movl %6,%%edi\n\t"
		"repne\n\t"
		"scasb\n\t"
		"notl %%ecx\n\t"
		"decl %%ecx\n\t"
		"movl %%ecx,%%edx\n"
		"1:\tlodsb\n\t"
		"testb %%al,%%al\n\t"
		"je 2f\n\t"
		"movl %6,%%edi\n\t"
		"movl %%edx,%%ecx\n\t"
		"repne\n\t"
		"scasb\n\t"
		"je 1b\n"
		"2:\tdecl %0"
		:"=S" (__res), "=&a" (d0), "=&c" (d1)
		:"1" (0),"2" (0xffffffff),"0" (cs),"g" (ct)
		:"dx","di");
	return __res-cs;
}
```

* **Line 5:** DF 플래그 클리어.
* **Line 6~11**: ct의 길이를 구하여 edx에 저장.
* **Line 12~14**: esi에서 1바이트를 읽고 al에 저장, NULL인지 확인 
* **Line 15~19**: ct 중 al과 일치하는 바이트가 있는지 확인, 없다면 리턴한다.
* **Line 20**: 일치하는 문자가 있다면\(je\) 1번 라벨로 돌아가 다음 문자를 비교한다.

```c
static inline int strcspn(const char * cs, const char * ct)
{
	int d0, d1;
	register char * __res __asm__("si");
	__asm__("cld\n\t"
		"movl %6,%%edi\n\t"
		"repne\n\t"
		"scasb\n\t"
		"notl %%ecx\n\t"
		"decl %%ecx\n\t"
		"movl %%ecx,%%edx\n"
		"1:\tlodsb\n\t"
		"testb %%al,%%al\n\t"
		"je 2f\n\t"
		"movl %6,%%edi\n\t"
		"movl %%edx,%%ecx\n\t"
		"repne\n\t"
		"scasb\n\t"
		"jne 1b\n"
		"2:\tdecl %0"
		:"=S" (__res), "=&a" (d0), "=&c" (d1)
		:"1" (0),"2" (0xffffffff),"0" (cs),"g" (ct)
		:"dx","di");
	return __res-cs;
}
```

* **Line 5:** DF 플래그 클리어.
* **Line 6~11**: ct의 길이를 구하여 edx에 저장.
* **Line 12~14**: esi에서 1바이트를 읽고 al에 저장, NULL인지 확인 
* **Line 15~19**: ct 중 al과 일치하는 바이트가 있는지 확인, 있다면 리턴한다.
* **Line 20**: 일치하는 문자가 없다면\(jne\) 1번 라벨로 돌아가 다음 문자를 비교한다.

```c
static inline char * strpbrk(const char * cs,const char * ct)
{
	int d0, d1;
	register char * __res __asm__("si");
	__asm__("cld\n\t"
		"movl %6,%%edi\n\t"
		"repne\n\t"
		"scasb\n\t"
		"notl %%ecx\n\t"
		"decl %%ecx\n\t"
		"movl %%ecx,%%edx\n"
		"1:\tlodsb\n\t"
		"testb %%al,%%al\n\t"
		"je 2f\n\t"
		"movl %6,%%edi\n\t"
		"movl %%edx,%%ecx\n\t"
		"repne\n\t"
		"scasb\n\t"
		"jne 1b\n\t"
		"decl %0\n\t"
		"jmp 3f\n"
		"2:\txorl %0,%0\n"
		"3:"
		:"=S" (__res), "=&a" (d0), "=&c" (d1)
		:"1" (0),"2" (0xffffffff),"0" (cs),"g" (ct)
		:"dx","di");
	return __res;
}
```

* **Line 5:** DF 플래그 클리어.
* **Line 6~11**: ct의 길이를 구하여 edx에 저장.
* **Line 12~14**: esi에서 1바이트를 읽고 al에 저장, NULL인지 확인 
* **Line 15~19**: ct 중 al과 일치하는 바이트가 있는지 확인, 있다면 찾은 주소를 리턴한다.
* **Line 20**: 일치하는 문자가 없다면\(jne\) 1번 라벨로 돌아가 다음 문자를 비교한다.

```c
static inline char * strstr(const char * cs,const char * ct)
{
int d0, d1;
register char * __res __asm__("ax");
__asm__("cld\n\t" \
    "movl %6,%%edi\n\t"
    "repne\n\t"
    "scasb\n\t"
    "notl %%ecx\n\t"
    "decl %%ecx\n\t"    /* NOTE! This also sets Z if searchstring='' */
    "movl %%ecx,%%edx\n"
    "1:\tmovl %6,%%edi\n\t"
    "movl %%esi,%%eax\n\t"
    "movl %%edx,%%ecx\n\t"
    "repe\n\t"
    "cmpsb\n\t"
    "je 2f\n\t"     /* also works for empty string, see above */
    "xchgl %%eax,%%esi\n\t"
    "incl %%esi\n\t"
    "cmpb $0,-1(%%eax)\n\t"
    "jne 1b\n\t"
    "xorl %%eax,%%eax\n\t"
    "2:"
    :"=a" (__res), "=&S" (d0), "=&c" (d1)
    :"0" (0),"2" (0xffffffff),"4" (cs),"g" (ct)
    :"dx","di");
return __res;
}
```

* **Line 5:** DF 플래그 클리어.
* **Line 6~11**: ct의 길이를 구하여 edx에 저장.
* **Line 12~14**: ct를 edi에 세팅, esi를 eax에 세팅, ecx에 구한 길이를 저장. 
* **Line 15~17**: cs에서 ct와 일치하는 문자열이 있는지 확인, 있다면 찾은 주소를 리턴한다.
* **Line 18~22**: 문자열 끝에 도달하면, 0을 리턴 아니라면 index를 증가시켜 1번 라벨로 돌아간다.

```c
static inline int strlen(const char * s)
{
int d0;
register int __res __asm__("cx");
__asm__("cld\n\t"
    "repne\n\t"
    "scasb\n\t"
    "notl %0\n\t"
    "decl %0"
    :"=c" (__res),"=&D" (d0)
    :"1" (s),"a" (0),"0" (0xffffffff));
return __res;
}
```

* **Line 5:** DF 플래그 클리어.
* **Line 6~10**: s의 길이를 구하여 \_\_res에 저장.



