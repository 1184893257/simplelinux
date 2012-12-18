[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">函数指针
</h1>

## 一、函数指针的值

　　函数指针跟普通指针一样，存的也是一个内存地址，
只是这个地址是一个函数的起始地址，
下面这个程序打印出一个函数指针的值（func1.c）：

	#include <stdio.h>
	
	typedef int (*Func)(int);
	
	int Double(int a)
	{
		return (a + a);
	}
	
	int main()
	{
		Func p = Double;
		printf("%p\n", p);
		return 0;
	}

`　　`编译、运行程序：

	[lqy@localhost notlong]$ gcc -O2 -o func1 func1.c
	[lqy@localhost notlong]$ ./func1
	0x80483d0
	[lqy@localhost notlong]$ 

`　　`然后我们用 nm 工具查看一下 Double 的地址，
看是不是正好是 0x80483d0：

	[lqy@localhost notlong]$ nm func1 | sort
	08048294 T _init
	08048310 T _start
	08048340 t __do_global_dtors_aux
	080483a0 t frame_dummy
	080483d0 T Double
	080483e0 T main
	...

`　　`不出意料，Double 的起始地址果然是 0x080483d0。

## 二、调用函数指针指向的函数

　　直接调用一个函数是 call 一个常量，
而通过函数指针调用一个函数显然不能这么做，
因为函数地址是可变的了，指向谁就得 call 谁。
下面比较一下直接调用和通过函数指针间接调用同一个函数的
汇编代码（func2.c）：

	#include <stdio.h>
	
	typedef int (*Func)(int);
	
	int Double(int a)
	{
		return (a + a);
	}
	
	int main()
	{
		Func p = Double;
		Double(2);	// 直接调用
		p(2);		// 间接调用
		return 0;
	}

`　　`部分汇编代码如下：

		movl	$2, (%esp)
		call	Double
		movl	$2, (%esp)
		movl	28(%esp), %eax	# 28(%esp) 是 p
		call	*%eax

`　　`可见通过函数指针间接调用一个函数，
call 指令的操作数不再是一个常量，
而是寄存器 eax（其它寄存器应该也行），
此时 eax 寄存器的值正好是 Double 函数的起始地址，
所以接着就会去执行 Double 函数的指令。

## 三、参数弱匹配

　　从上面的例子中我们也看到了函数指针也没什么特别的，
也就存了个地址，但是调用一个函数不仅需要知道它的起始地址，
还得根据它的参数列表来压栈传递参数。

　　参数列表在定义函数指针类型的时候就约定好了，
凡是具有相同参数列表的函数都可以赋值给该类型的函数指针，
而参数列表不同的函数也可以通过强制类型转换后赋值给它
（C语言的指针类型可以任意转换⊙﹏⊙），
下面这个程序就大胆的强制转换了一下（func3.c）：

	#include <stdio.h>
	
	typedef int (*Func)(int);
	
	int Double2(int a, int b)
	{
		return (a + a);
	}
	
	int main()
	{
		Func p = (Func)Double2;
		printf("%d\n", p(2));
		return 0;
	}

`　　`不强制转换的话，编译的时候会报告一个 warring
（居然不是 error ⊙﹏⊙），
上面这个程序编译的时候 0 error 0 warring，
执行也没有出错：

	[lqy@localhost notlong]$ gcc -o func3 func3.c
	[lqy@localhost notlong]$ ./func3
	4
	[lqy@localhost notlong]$ 

`　　`真算是朵奇葩了！

　　没有出错的原因是：参数 a 对应的刚好是压栈的 2，
而 b 对应的是一个危险地带，还好没用到 b，
所以这个程序依然顺利地执行完了。

　　<b>综上所述，函数指针真没什么特别的。</b>

[回目录][content]
