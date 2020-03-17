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
* **Line 8:** 
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
* **Lien 14**: 생략

```c
static inline char * strcat(char * dest,const char * src)
{
	int d0, d1, d2;
	__asm__("cld\n\t"
		"repne\n\t"
		"scasb\n\t"
		"decl %1\n"
		"1:\tlodsb\n\t"
		"stosb\n\t"
		"testb %%al,%%al\n\t"
		"jne 1b"
		:"S"(d0), "D"(d1), "a"(d2)
		:"0" (src),"1" (dest),"2" (0),"c" (0xffffffff));
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
	__asm__("cld\n\t"
		"repne\n\t"
		"scasb\n\t"
		"decl %1\n\t"
		"movl %4,%3\n"
		"1:\tdecl %3\n\t"
		"js 2f\n\t"
		"lodsb\n\t"
		"stosb\n\t"
		"testb %%al,%%al\n\t"
		"jne 1b\n"
		"2:\txorl %2,%2\n\t"
		"stosb"
		::"S" (src),"D" (dest),"a" (0),"c" (0xffffffff),"g" (count)
		);
	return dest;
}
```





