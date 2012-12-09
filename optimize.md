[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">编译优化
</h1>

　　C语言没有汇编快，因为C语言要由编译器翻译为汇编，
编译器毕竟是人造的，翻译出来的汇编源代码总有那么N条
指令在更智能、更有创造性的我们看来是多余的。

　　C语言翻译后的汇编有如下恶劣行径：

1. <b>C语言偏爱内存</b>。我们写的汇编一般偏爱寄存器，
寄存器比内存要快很多倍。当然，寄存器的数量屈指可数，
数据多了的话也必须用内存。
2. <b>内存多余读</b>。假如在一个 for 循环中经常要执行
++i 操作，编译后的汇编可能是这样的情形：

		movl i, %eax
		addl $1, %eax
		movl %eax, i

	即使 eax 寄存器一直存着 i 的值，
C语言也喜欢操作它前先读一下，以上3条指令浓缩为一条
incl %eax 速度就快上好几倍了。

`　　`尽管C语言"如此不堪"，但是考虑到高级语言带来的
源码可读性和开发效率在数量级上的提高，我们还是原谅了它。
而且很多编译器都有提供优化的选项，
开启优化选项后C语言翻译出来的汇编代码几近无可挑剔。

　　VC、VS有 Debug、Release 编译模式，
Release 下编译后，程序的大小、执行效率都有显著的改善。
gcc 也有优化选项，我们来看看 gcc 优化的神奇效果：

　　我故意写了一个垃圾程序（math.c）：

	#include <stdio.h>
	
	int main()
	{
		int a=1, b=2;
		int c;
		
		c = a + a*b + b;
		
		printf("%d\n", c);
		return 0;
	}

且看看不优化的情况下，汇编代码有多么糟糕：

编译命令：gcc -S math.c  
main部分的汇编代码：

	main:
		pushl	%ebp
		movl	%esp, %ebp
		andl	$-16, %esp
		subl	$32, %esp
		movl	$1, 28(%esp)	# 28(%esp) 是 a
		movl	$2, 24(%esp)	# 24(%esp) 是 b
		movl	24(%esp), %eax	#\
		addl	$1, %eax		#-\
		imull	28(%esp), %eax	#-eax=(b+1)*a
		addl	24(%esp), %eax	#\
		movl	%eax, 20(%esp)	#-c=(b+1)*a+b
		movl	$.LC0, %eax
		movl	20(%esp), %edx
		movl	%edx, 4(%esp)
		movl	%eax, (%esp)
		call	printf
		movl	$0, %eax
		leave
		ret

汇编代码规模庞大，翻译水平中规中矩。
现在开启优化选项：

编译命令：gcc -O2 -S math.c

	main:
		pushl	%ebp
		movl	%esp, %ebp
		andl	$-16, %esp
		subl	$16, %esp
		movl	$5, 4(%esp)
		movl	$.LC0, (%esp)
		call	printf
		xorl	%eax, %eax
		leave
		ret

`　　`规模变为原来的一半，而且 gcc 发现了 a、b、c 
变量是多余的，直接将结果 5 传给 printf 打印了出来
——计算器是编译器必备的一大技能。
初中那时候苦逼地做计算题，怎么就不学学C语言呢O(∩_∩)O~

[回目录][content]
