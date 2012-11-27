[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">谁调用了main？
</h1>

　　这是函数帧的应用之一。

## 操作可行性

　　从上一篇中可以发现：用帧指针 ebp 可以回溯到所有的函数帧，
那么 main 函数帧之上的函数帧自然也是可以的；
而帧中 旧ebp 的上一个四字节存的是函数的返回地址，
由这个地址我们可以判断出谁调用了这个函数。

## 准备活动

　　下面就是这次黑客行动的主角（up.c）：

	#include <stdio.h>
	
	int main()
	{
		int *p;
		
		// 以下这行内联汇编将 ebp 寄存器的值存到指针 p 中
		__asm__("movl %%ebp, %0"
				:"=m"(p));
		
		while(p != NULL){
			printf("%p\n", p[1]);
			p = (int*)(p[0]);
		}
		
		return 0;
	}

`　　`首先，请允许我使用一下 gcc 内联汇编，
这里简单的解释一下：

1. "=m"(p) 表示将内存变量 p 作为一个输出操作数
2. %0 代表的是第一个操作数，那就是 p 了
3. 为了与操作数区别开来，寄存器要多加个 %，
%%ebp 表示的就是 ebp 寄存器

`　　`总之，这块内联汇编将 ebp 寄存器的值赋给了指针 p。

　　然后解释一下while循环：循环中，首先打印 p[1]，
p[1]就是该帧所存的返回地址；然后将指针 p 改为 p[0]，
p[0]是 旧ebp（上一帧的帧指针）；
这样，程序将按照<b>调用顺序的逆序</b>打印出各个返回地址。

　　为什么终止条件是 p==NULL 呢？这是 gcc 为了支援我们的
黑客行动特意在开始执行程序的时候将 ebp 清零了，
所以第一次执行某个函数的时候压栈的 旧ebp 是 NULL。

## 开始行动

　　我们使用静态链接的方式编译 up.c
（静态链接的可执行文件中包含所有用户态下执行的代码），
然后执行它：

	[lqy@localhost temp]$ gcc -static -o up up.c
	[lqy@localhost temp]$ ./up
	0x8048464
	0x80481e1
	[lqy@localhost temp]$ 

## 分析结果

　　up 打印了了两个指向代码区的地址，
接着就看它们是属于哪两个函数了：

	nm up | sort > up.txt

* nm up 可列出各个全局函数的地址
* | sort > up.txt 通过<b>管道</b>将 nm up 的输出作为 sort 的输入，
sort 排序后<b>输出重定向</b>到 up.txt 文件中（输出有1910行，
不得不这么做o(╯□╰)o）

`　　`然后发现两个地址分别位于 `__libc_start_main`、_start 中：

	...
	08048140 T _init
	080481c0 T _start
	080481f0 t __do_global_dtors_aux
	08048260 t frame_dummy
	080482bc T main
	08048300 T __libc_start_main
	080484d0 T __libc_check_standard_fds
	...

`　　`实际上程序正好是从 _start 开始执行的，
而且从 up 的反汇编结果中可看出 _start 的第一条指令
 xor %ebp,%ebp 就是那条传说中的将 ebp 清零的指令
（两个一样的数相异或的结果一定是0）。

　　那么调用 main 函数之前程序都干了些啥事呢？
比如说<b>堆的初始化</b>，如果是 C++ 程序的话，
全局对象的构造也是在 main 之前完成的
（不能让 main 中使用全局对象的时候竟然还没构造吧！），
而全局对象的析构也相当有趣地在 main 执行完了之后才执行。

　　main 在你心目中的地位是不是一落千丈了？

[回目录][content]
