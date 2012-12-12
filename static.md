[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">static变量 及 作用域控制
</h1>

## 一、static变量

　　<b>static变量放在函数中，就只有这个函数能访问它；
放在函数外就只有这个文件能访问它。</b>
下面我们看看两个函数中重名的static变量是怎么区别开来的
（static.c）：

	#include <stdio.h>
	
	void func1()
	{
		static int n = 1;
		n++;
	}
	
	void func2()
	{
		static int n = 2;
		n++;
	}
	
	int main()
	{
		return 0;
	}

`　　`下面是编译后的部分汇编：

	func1:
		pushl	%ebp
		movl	%esp, %ebp
		movl	n.1671, %eax
		addl	$1, %eax
		movl	%eax, n.1671
		popl	%ebp
		ret
	
	func2:
		pushl	%ebp
		movl	%esp, %ebp
		movl	n.1674, %eax
		addl	$1, %eax
		movl	%eax, n.1674
		popl	%ebp
		ret

`　　`好家伙！编译器居然"偷偷"地改了变量名，
这样两个static变量就容易区分了。

　　<b>其实static变量跟全局变量一样被放置在 .data段 或 
.bss段 中，所以它们也是程序运行期间一直存在的，
最终也是通过绝对地址来访问</b>。
但是它们的作用域还是比全局变量低了一级：
static变量被标识为LOCAL符号，全局变量被标识为GLOBAL符号，
目标文件在寻找外部变量时只在GLOBAL符号中找，
所以static变量别的源文件是"看不见"的。

## 二、作用域控制

　　作用域控制为的是提高源代码的可读性，
一个变量的作用域越小，它可能出没的范围就越小。

　　C语言中的变量按作用域从大到小可分为四种：
全局变量、函数外static变量、函数内static变量、局部变量：

1. 全局变量是杀伤半径最大的：<b>不仅在定义该变量的源文件中可用，
而且在任一别的源文件中只要用 extern 声明它后也可以使用</b>，
因此，当你看到一个全局变量的时候应该心生敬畏！
2. 函数外的static变量处于文件域中，
只有定义它的源文件中可以使用。如果你看到一个static变量，
那是作者在安慰你：哥们（妹子），这个变量不会在别的文件中出现。
3. 函数内static变量在函数的<b>每次调用</b>中可用（只初始化一次），
<b>它同以上两种变量一样在程序运行期间一直存在</b>，
所以它的功能是局部变量无法实现的。
4. 局部变量在函数的<b>一次调用</b>中使用，
调用结束后就消失了。

`　　`显然，作用域越小越省心，
该是局部变量的就不要定义成全局变量，
如果"全局变量"只在本源文件中使用那就加个static。

　　即便是局部变量也还可以压缩其作用域：

　　有的同学写的函数一开头就声明了函数中要用到的所有局部变量，
一开始我也这么做，因为我担心：<b>如果把变量定义在循环体内，
是不是每一次循环都会给它们分配空间、回收空间，从而降低效率？</b>
但事实是它们的空间在函数的开头就一次性分配好了（scope.c）：

	#include <stdio.h>
	
	int main()
	{
		int a = 1;
		{
			int a = 2;
			{
				int a = 3;
			}
			{
				int a = 4;
			}
		}
		return 0;
	}

编译后的汇编代码如下：

	main:
		pushl	%ebp
		movl	%esp, %ebp
		subl	$16, %esp
		movl	$1, -4(%ebp)
		movl	$2, -8(%ebp)
		movl	$3, -12(%ebp)
		movl	$4, -16(%ebp)
		movl	$0, %eax
		leave
		ret

`　　`各层局部环境中的变量a是subl $16, %esp一次性分配好的。
<b>由此可见不是每个{}都要分配回收局部变量，
一个函数只分配回收一次</b>。因此，
如果某个变量只在某个条件、循环中用到的话，
还是在条件、循环中定义吧，这样，
规模比较大的函数的可读性将提高不少，而效率丝毫没有下降，
可谓是百利而无一害！

[回目录][content]
