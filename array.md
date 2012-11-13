[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">数组和指针
</h1>

　　指针和数组有什么区别？

　　C程序（array.c）：

	#include <stdio.h>
	
	int main()
	{
		int date[3] = {2012,11,11};
		
		int *p = date;
		
		int a = date[1];
		int b = p[1];
		
		printf("a:%d b:%d\n", a, b);
		printf("date:%p\np   :%p\n", date, p);
		return 0;
	}

`　　`汇编及注释：

<table>
<tr><td>
<pre><code>    .file	"array.c"
	.section	.rodata
.LC0:
	.string	"a:%d b:%d\n"
.LC1:
	.string	"date:%p\np   :%p\n"
	.text
.globl main
	.type	main, @function
main:
	pushl	%ebp			#-帧指针切换
	movl	%esp, %ebp		#/
	andl	$-16, %esp		#-栈对齐到16字节
	subl	$48, %esp		#-开拓局部变量空间
	movl	$2012, 24(%esp)	#\
	movl	$11, 28(%esp)	#-date数组初始化
	movl	$11, 32(%esp)	#/
	leal	24(%esp), %eax	#\
	movl	%eax, 44(%esp)	#-初始化指针p
	movl	28(%esp), %eax	#\
	movl	%eax, 40(%esp)	#-初始化a
	movl	44(%esp), %eax	#\
	addl	$4, %eax		#-\
	movl	(%eax), %eax	#-初始化b
	movl	%eax, 36(%esp)	#/
</code></pre></td>
<td valign="bottom"><img src="http://fmn.rrfmn.com/fmn059/20121113/1850/original_fmSH_48030000e812125c.jpg" /></td>
</tr>

<tr><td>
<pre><code>    movl    $.LC0, %eax
	movl	36(%esp), %edx
	movl	%edx, 8(%esp)
	movl	40(%esp), %edx
	movl	%edx, 4(%esp)
	movl	%eax, (%esp)
	call	printf		#第一次调用 printf 函数
</code></pre></td>
<td valign="bottom"><img src="http://fmn.rrimg.com/fmn062/20121113/1850/original_6dpg_28e80000e7fc1191.jpg" /></td>
</tr>

<tr><td>
<pre><code>    movl    $.LC1, %eax
	movl	44(%esp), %edx
	movl	%edx, 8(%esp)
	leal	24(%esp), %edx
	movl	%edx, 4(%esp)
	movl	%eax, (%esp)
	call	printf		#第二次调用 printf 函数
</code></pre></td>
<td valign="bottom"><img src="http://fmn.rrimg.com/fmn056/20121113/1850/original_ECol_44540000c23f1190.jpg" /></td>
</tr>

<tr><td colspan="2">
<pre><code>    movl    $0, %eax    #return 0
	leave
	ret
	.size	main, .-main
	.ident	"GCC: (GNU) 4.5.1 20100924 (Red Hat 4.5.1-4)"
	.section	.note.GNU-stack,"",@progbits
</code></pre></td>
</tr>
</table>

　　大家对 lea 指令可能比较陌生，它跟 mov 指令很像，
不过 lea 指令传递的是内存的地址，而 mov 指令传递的的是
内存的值，例如：

	leal 24(%esp), %eax		# 将 24+esp 的结果给 eax
	movl 24(%esp), %eax		# 将 2012 给 eax

## 总结

　　首先，我们看到 main 也是一个函数，它编译后的汇编代码
跟<em>函数调用</em>中分析过的 Double 函数有一样的结构。

　　其次，指针是有存储空间的（32位机是4个字节的大小），
其中存储的是一个内存地址；而数组只有数组中的元素有存储空间，
而 C 代码中经常用到的数组名（被当做数组首地址来使用）
可以说是没有存储空间的。假设将数组名赋值给一个指针，
数组名的存在形式是：

* 如果是全局数组，它将是 mov 指令中的一个立即数（常量）
* 如果是局部数组，它将是 lea 指令中由 esp 或 ebp + 
常量计算的结果

`　　`<b>所以数组的首地址是一条机器指令的操作数，
不在局部变量空间中，也不在全局变量空间中，
而是被固化在代码中</b>。

　　printf 函数的调用跟别的函数一样：先传参数，
然后 call printf，打印指针都可以使用 %p 格式描述符。
程序的运行结果如下：

	[lqy@localhost temp]$ gcc -o array array.c
	[lqy@localhost temp]$ ./array
	a:11 b:11
	date:0xbf9c3b68
	p   :0xbf9c3b68
	[lqy@localhost temp]$ 

`　　`date 和 p 的值都是 date 数组的首地址。

[回目录][content]
