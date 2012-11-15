[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">字符串
</h1>

　　这一篇分析字符串，字符串经常被使用，
但是它的秘密也不少：

## 一、字符串的存储位置

　　C源程序（string1.c）：

	#include <stdio.h>
	
	int main()
	{
		puts("Hello, World!");
		return 0;
	}

`　　`我们直接看可执行文件的反汇编结果：

	[lqy@localhost temp]$ gcc -o string1 string1.c
	[lqy@localhost temp]$ ./string1 
	Hello, World!
	[lqy@localhost temp]$ objdump -s -d string1 > string1.txt
	[lqy@localhost temp]$ 

`　　`string1.txt 中的部分内容如下：

<pre><code>Contents of section .rodata:
 8048488 03000000 01000200 00000000 48656c6c  ............Hell
 8048498 6f2c2057 6f726c64 2100               o, World!.      

...

080483b4 &lt;main>:
 80483b4:	55                   	push   %ebp
 80483b5:	89 e5                	mov    %esp,%ebp
 80483b7:	83 e4 f0             	and    $0xfffffff0,%esp
 80483ba:	83 ec 10             	sub    $0x10,%esp<b>
 80483bd:	c7 04 24 94 84 04 08 	movl   $0x8048494,(%esp)
 80483c4:	e8 27 ff ff ff       	call   80482f0 &lt;puts@plt></b>
 80483c9:	b8 00 00 00 00       	mov    $0x0,%eax
 80483ce:	c9                   	leave  
 80483cf:	c3                   	ret    
</code></pre>

　　可见 0x8048494 应该是 "Hello, World!" 的地址，
然后发现在 .rodata 段中 0x8048488 + 12 处正好存储着
 "Hello, World!" 的各个字符的
<a target="_blank" href="http://www.asciitable.com/">ASCII码</a>：
'H' 的 <a target="_blank" href="http://www.asciitable.com/">ASCII码</a>是 0x48，
而感叹号 '!' 的 <a target="_blank" href="http://www.asciitable.com/">ASCII码</a> 码是 0x21，
最后编译器还自动的为我们添了个字符串结束符 0x00：
<b>只要是双引号括起来的字符串，编译器都会自动的为我们加结束符。</b>

　　由此我们发现：
<b>字符串和全局变量一样做为静态数据存储在可执行文件中，
在使用的时候用常量地址来访问。</b>

　　但是字符串被放在了 .rodata 段中，
这个段中的数据与 .text 段（代码段）中的数据一样
在可执行文件被载入内存运行时
都是只读的（这里的只读是通过分页管理实现的：
在页表表项中有一个位，设置为 1 表示该页可写，
设置为 0 表示该页只读；如果试图向只读的页中写入数据，
CPU 就会触发页保护异常）。
<b>所以源字符串中的任何字符都不能在程序运行时更改</b>。

## 二、字符串指针 和 字符数组

　　C源程序（string2.c）：

	#include <stdio.h>
	
	int main()
	{
		char *s1 = "1234567";
		char s2[]= "1234567";
		
		puts(s1);
		puts(s2);
		return 0;
	}

`　　`可执行文件的反汇编结果的赋值部分如下：

	 80483bd:	c7 44 24 1c c4 84 04 	movl   $0x80484c4,0x1c(%esp)
	 80483c4:	08 
	 80483c5:	a1 c4 84 04 08       	mov    0x80484c4,%eax
	 80483ca:	8b 15 c8 84 04 08    	mov    0x80484c8,%edx
	 80483d0:	89 44 24 14          	mov    %eax,0x14(%esp)
	 80483d4:	89 54 24 18          	mov    %edx,0x18(%esp)

`　　`0x80484c4 是字符串"1234567"的地址，
所以：<b>常量字符串赋值给指针时传递的是源字符串的地址；
而赋值给局部字符数组时，要当成数字一个个拷贝到局部变量空间</b>。
因此第2种方式既浪费空间（4字节 vs 8字节）
又浪费时间（1个mov vs 4个mov）。但是第2种方式也不是
一无是处：<b>第1种方式传递的是源字符串的地址，
而源字符串在只读页中，无法修改，但是字符数组却可以修改</b>。

　　经验：如果程序中本来就没想改动该字符串，
那就用指针吧；否则用 字符数组 或 动态申请的空间 来存。

## 三、格式描述符 和 转义符

　　这个部分，我们来看看字符串中格式描述符和转义符的来龙去脉，
C源程序（string3.c）：

	#include <stdio.h>
		
	int main()
	{
		printf("--------\n%d\n", 123);
		return 0;
	}

### 汇编源文件中

	gcc -S string3.c

`　　`结果如下：

	.LC0:
		.string	"--------\n%d\n"

### 目标文件中

	gcc -c string3.c
	objdump -s -d string3.o > string3.txt

`　　`-c 默认输出到 string3.o 文件中，
string3.txt 中的字符串：

	Contents of section .rodata:
	 0000 2d2d2d2d 2d2d2d2d 0a25640a 00        --------.%d..   

`　　`两个'\n'被替换成了 0x0a，%d 没变

### 可执行文件中

	gcc -o string3 string3.c
	objdump -s -d string3 > string3.txt

`　　`可执行文件跟目标文件一样：

	Contents of section .rodata:
	 80484a8 03000000 01000200 00000000 2d2d2d2d  ............----
	 80484b8 2d2d2d2d 0a25640a 00                 ----.%d..       

`　　`所以我们知道了：<b>转义符在变成二进制文件后
就被转义为我们想表达的那个字符了</b>，
而格式描述符自然是要留给 printf 运行时用的。
知道这个有什么用？如果我们来设计 printf 函数，
那么转义字符我们就不用操心了，
编译器会把它们转义过去的。

　　如果你想看看 printf 大概的实现，
linux 0.01 的 kernel/vsprintf.c 中有一个简化的
（没有实现浮点数的处理）实现，其中转义字符是不操心的。

linux 各版本内核源代码：<a target="_blank" href="http://www.kernel.org/pub/linux/kernel/">http://www.kernel.org/pub/linux/kernel/</a>  
linux 0.01：<a target="_blank" href="http://www.kernel.org/pub/linux/kernel/Historic/">http://www.kernel.org/pub/linux/kernel/Historic/</a>

[回目录][content]
