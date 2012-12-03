[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">未初始化全局变量
</h1>

　　为下一篇介绍进程内存分布做准备，
这一篇先来介绍一下未初始化全局变量：

　　未初始化全局变量，这名字就很直白，就是 C 程序中定义成
全局作用域而又没有初始化的变量，我们知道这种变量在程序运行
后是被自动初始化为 全0 的。编译器编译的时候会将这类变量
收集起来集中放置到 .bss 段中，<b>这个段只记录了段长，
没有实际上的内容（全是0，没必要存储），
在程序被装载时操作系统会
为它分配等于段长的内存，并全部初始化为0</b>。

　　这有两个 C程序，都定义了全局数组 data（长度为1M，
占用内存4MB），一个部分初始化（bss\_init1.c），
一个未初始化（bss\_uninit1.c）：

bss_init1.c：

	#include <stdio.h>
	#include <windows.h>
	
	#define MAXLEN 1024*1024
	
	int data[MAXLEN]={1,};
	
	int main()
	{
		Sleep(-1);
		return 0;
	}

bss_uninit1.c：

	#include <stdio.h>
	#include <windows.h>
	
	#define MAXLEN 1024*1024
	
	int data[MAXLEN];
	
	int main()
	{
		Sleep(-1);
		return 0;
	}

`　　`编译以上两个程序后：

![bss1](http://fmn.rrfmn.com/fmn059/20121203/1935/original_4q5M_35d80000b351118d.jpg)

　　可以看到有初始化的可执行文件的大小差不多是4MB，
而未初始化的只有47KB！这就是 .bss 段有段长，
而没有实际内容的表现。用 UltraEdit 打开 bss_init1.exe 
可看到文件中大部分是全0（data数组的内容）：

![bss5](http://fmn.rrimg.com/fmn065/20121203/1935/original_RbRN_5afd0000b341125d.jpg)

　　但是接下来运行（return 0 之前的 Sleep(-1) 保证了
程序暂时不会退出）的时候，却发现 bss_init1.exe 
占用的空间明显少于 4MB，这是怎么回事呢？

![bss2](http://fmn.rrimg.com/fmn065/20121203/1935/original_ejt4_363e0000b309118d.jpg)

　　这就涉及程序装载的策略了。早期的操作系统（如：linux 0.01）
采用的是一次装载：将可执行文件一次性完整装入内存后再执行程序。
不管程序是 1KB 还是 60MB，都要等全部装入内存后才能执行，
这显然是不太合理的。

　　而现在的操作系统都是采用延迟装载：
<b>将进程空间映射到可执行文件之后就开始执行了</b>，
执行的时候如果发现要读/写的页不在内存中，
就根据映射关系去读取进来，然后继续执行应用程序
（应该是在页保护异常的处理中实现的）。

　　bss_init1.exe 肯定是被映射了，<b>而程序中又没有对 data 
数组进行读/写操作</b>，所以操作系统也就懒得去装入这片内存了。
下面修改一下这两个程序：在 Sleep(-1) 前将 data 数组
的每个元素赋值为 -1：

	int i;
	for(i=0; i<MAXLEN; ++i)
		data[i] = -1;

`　　`再运行，它们占用的内存都是 4M 了：

![bss4](http://fmn.rrimg.com/fmn061/20121203/1935/original_OHR0_75660000b3e5118c.jpg)

[回目录][content]
