[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">函数调用
</h1>

## 前言

　　本来打算在写完 数组、结构体、指针 等之后再写函数调用的，
因为函数调用会牵扯到这些东西，
但是我觉得那样做的话总会露出点意犹未尽的马脚，
所以还是先简单地分析一下函数调用吧，
之后再不断的完善函数调用这个大家伙。

## C源程序（double.c）

	#include <stdio.h>
	
	int Double(int b)
	{
		int c;
		
		c = b + b;
		++b;	// 会影响到 a 吗？
		
		return c;
	}
	
	int main()
	{
		int a = 1;
		int d = Double(a);
	
	    printf("a:%d d:%d\n", a, d);
	    return 0;
	}

## 反汇编

	gcc -S double.c

`　　`gcc -S double.c 默认就是把汇编源代码输出到 double.s 中，
之前一直用 -o 选项是为了避免过多的解释O(∩_∩)O~，
double.s 中的内容简化后：

<pre><code>Double:<b>
	pushl	%ebp
	movl	%esp, %ebp
	subl	$16, %esp
	movl	8(%ebp), %eax
	addl	%eax, %eax
	movl	%eax, -4(%ebp)
	addl	$1, 8(%ebp)
	movl	-4(%ebp), %eax
	leave
	ret</b>

main:
	pushl	%ebp
	movl	%esp, %ebp
	andl	$-16, %esp
	subl	$32, %esp<b>
	movl	$1, 28(%esp)
	movl	28(%esp), %eax
	movl	%eax, (%esp)
	call	Double</b>
	movl	%eax, 24(%esp)
	movl	$.LC0, %eax
	movl	24(%esp), %edx
	movl	%edx, 8(%esp)
	movl	28(%esp), %edx
	movl	%edx, 4(%esp)
	movl	%eax, (%esp)
	call	printf
	movl	$0, %eax
	leave
	ret
</code></pre>

## 分析

　　<b>粗体</b>部分是对 Double 函数的一次完整调用。

　　现在从 main 函数中第 1 条<b>粗体</b>指令开始分析：

<table border="1">

<tr><td><pre><code>
	movl	$1, 28(%esp)	# a=1
</code></pre></td>
<td rowspan="2"><img src="http://fmn.rrimg.com/fmn056/20121109/1855/original_1Ykq_291900004dfe1191.jpg" />
</td></tr>
<tr><td>
这是给 局部变量 a 赋值<br>
假设栈指针寄存器 esp 现在的值是 8000，
这条指令执行完后栈的情况如右图
</td></tr>

<tr><td><pre><code>
	movl	28(%esp), %eax
	movl	%eax, (%esp)	# (%esp) = a
</code></pre></td>
<td rowspan="2"><img src="http://fmn.rrimg.com/fmn063/20121109/1855/original_6CEt_3c0400004d8a125d.jpg" />
</td></tr>
<tr><td>
这两条指令将 a 的值 1 写入了地址为 8000 的内存块（4字节），
这可能是 b 吧？<br>现在还不要妄下结论。
</td></tr>

<tr><td><pre><code>
	call	Double
</code></pre></td>
<td rowspan="2"><img src="http://fmn.rrfmn.com/fmn058/20121109/1855/original_cK4p_448e000041101190.jpg" />
</td></tr>
<tr><td>
call 指令的执行，可分为两个步骤：<br>
<ol>
	<li>将 call 指令的下一条指令的地址压栈
	<li>将 eip 修改为 Double 的地址
</ol>
然后就跳转到 Double 处取指令执行了。<br>
假设 call 指令之后的那条 movl	%eax, 24(%esp) 
的地址是 1000，那么 call 执行后，
栈的情况会变为右图的样子
</td></tr>

<tr><td><pre><code>
	pushl	%ebp
	movl	%esp, %ebp
</code></pre></td>
<td rowspan="2"><img src="http://fmn.rrfmn.com/fmn058/20121109/1855/original_5Rl9_5de000004db9118f.jpg" />
</td></tr>
<tr><td>
先将旧的 ebp 压栈，然后把这个时候的 esp 赋值给 ebp，
后面会看到这样做的目地
</td></tr>

<tr><td><pre><code>
	subl	$16, %esp	# esp -= 16
</code></pre></td>
<td rowspan="2"><img src="http://fmn.rrimg.com/fmn056/20121109/1855/original_7ikk_0fad00004dc5118e.jpg" />
</td></tr>
<tr><td>
Double 的局部变量空间就这么被开拓了（虽然有点浪费）
</td></tr>

<tr><td><pre><code>
	movl	8(%ebp), %eax
	addl	%eax, %eax
	movl	%eax, -4(%ebp)	# c = b + b
	addl	$1, 8(%ebp)		# ++b
</code></pre></td>
<td rowspan="2"><img src="http://fmn.rrimg.com/fmn065/20121109/1855/original_VXpq_05be00004de5118d.jpg" />
</td></tr>
<tr><td>
8(%ebp) 当前刚好表示 8000 内存块，
从这 4 条指令对 8000 内存块的值进行的处理，
我们可以确定 8000 就是参数 b，
而 -4(%ebp) 则是 c。
</td></tr>

<tr><td><pre><code>
	movl	-4(%ebp), %eax	# eax = c
	leave
	ret
</code></pre></td>
<td rowspan="2"><img src="http://fmn.rrimg.com/fmn056/20121109/1850/original_zq1t_433a00004e92125b.jpg" />
</td></tr>
<tr><td>
这就是最后的收尾部分了，<b>函数的返回值要存储在累加寄存器 eax 中</b>，
leave 等同于以下两条指令：<br>
movl %ebp, %esp<br>
popl %ebp<br>
与刚进入 Double 时的两条指针首尾呼应。<br>
<b>结果就是局部空间被回收了，ebp 也复原了。</b>
<p>而 ret 则相当于 popl %eip，
程序就继续执行 call 指令之后的指令了。</p>
虽然 Double 函数的局部空间被回收了，
但是其中的值还是保持不变的，一直到之后调用 printf 函数的时候，
Double 函数的局部变量以及参数的内容才被覆盖。
</td></tr>

</table>

## 整理

　　一步步的分析结束后，再从大粒度上回顾一次 Double 函数：

<pre><code>Double:
	pushl	%ebp				#----------帧指针 ebp 切换
	movl	%esp, %ebp			#---------/
	subl	$16, %esp			#----------开拓局部变量空间
	movl	8(%ebp), %eax		#\
	addl	%eax, %eax			#-\
	movl	%eax, -4(%ebp)		#-C 的操作
	addl	$1, 8(%ebp)			#/
	movl	-4(%ebp), %eax		#-----将返回值保存到 eax 寄存器
	leave						#--------\
	ret							#---------退栈、恢复帧指针、返回
</code></pre>

　　其他函数编译为汇编后，指令也可以这样来进行划分。

　　<b>由于 Double 函数太过简单，所以没有出现保护寄存器的指令
（eax 寄存器不需要保护，每个函数都知道它是用来存返回值的，
在函数的最后部分肯定会被修改），复杂的函数会在函数的最前面
pushl 将被修改的寄存器，而在 ret 之前 popl 寄存器以恢复原来的值。</b>

## 小结

　　通过对 Double 函数的分析，
我们注意到 ebp 跟 esp 的作用类似，
函数中经常也通过 ebp + 偏移 的方式来访问 参数 和 局部变量，
现在可以向大家承诺了：<b>esp 或 ebp 加偏移就是
局部变量最后的表现形式</b>。
关于 帧指针寄存器 ebp 后面会再出一篇来进行分析。

　　同时我们也可以明显地看出值传递的过程：
调用前将 a 复制到 b（值拷贝，它们分别是不同的内存块），
函数调用完后 b 直接被忽略了，也不会再拷贝回 a，
所以虽然我在 Double 中 ++b 了，而 a 的值仍然为 1。
程序的运行结果如下：

	[lqy@localhost temp]$ gcc -o double double.c
	[lqy@localhost temp]$ ./double
	a:1 d:2
	[lqy@localhost temp]$ 

`　　`下一篇中再继续讨论值传递。

[回目录][content]
