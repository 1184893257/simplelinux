[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">局部变量
</h1>

　　在接下来的几篇文章中，
我将利用"火眼金睛"来分析一个个的C程序，
为大家揭开 C 语言的奥秘。

## 怪题

　　有一天，一个朋友发现一段奇怪的 C 程序：

	int i = 3;
	int ans = (++i)+(++i)+(++i);

`　　`书上说答案是 18，我还以为是 4+5+6=15 呢。

## 验证

　　然后我就想着验证一下结果到底是怎样的，
我写了如下的测试程序（inc.c）：

	#include <stdio.h>
	
	int main()
	{
		int i = 3;
		int ans = (++i)+(++i)+(++i);
	
		printf("%d\n",ans);
		return 0;
	}

`　　`在 linux 中编译、运行，结果如下：

	[lqy@localhost temp]$ gcc -o inc inc.c
	[lqy@localhost temp]$ ./inc
	16
	[lqy@localhost temp]$ 

`　　`既然又出现了一个令人匪夷所思的 16！

## 揭秘

　　好吧，先不管 18 了，看看这个 16 是怎么来的：

	gcc -S -o inc.s inc.c

`　　`生成了汇编源文件 inc.s，
现在我们只关心其中的<b>粗体</b>部分：

<pre><code>    .file	"inc.c"
	.section	.rodata
.LC0:
	.string	"%d\n"
	.text
.globl main
	.type	main, @function
main:
	pushl	%ebp
	movl	%esp, %ebp
	andl	$-16, %esp
	subl	$32, %esp<b>
	movl	$3, 28(%esp)
	addl	$1, 28(%esp)
	addl	$1, 28(%esp)
	movl	28(%esp), %eax
	addl	%eax, %eax
	addl	$1, 28(%esp)
	addl	28(%esp), %eax
	movl	%eax, 24(%esp)</b>
	movl	$.LC0, %eax
	movl	24(%esp), %edx
	movl	%edx, 4(%esp)
	movl	%eax, (%esp)
	call	printf
	movl	$0, %eax
	leave
	ret
	.size	main, .-main
	.ident	"GCC: (GNU) 4.5.1 20100924 (Red Hat 4.5.1-4)"
	.section	.note.GNU-stack,"",@progbits
</code></pre>

　　现在我我不得不解释一下这种汇编的格式了，
它是 AT&T 格式的 x86 汇编，
我们在 windows 上见到的一般是 Intel 格式的汇编，
AT&T 汇编与 Intel 格式的汇编有些差异，
不过还是很好理解的。

　　首先是寄存器的命名格式不同，
在 Intel 格式的寄存器前加了个 %：eax 变成了 %eax  
然后就是双元指令的操作数的传递方向与 Intel 的刚好相反：
mov eax，ebx（相当于 eax=ebx;） 变成了 movl %ebx，%eax
（左向右传值）  
还有其他一些差别，不过还是容易看明白的。

　　给粗体部分加点注释吧：

	movl	$3, 28(%esp)	# i = 3;
	addl	$1, 28(%esp)	# ++i; // 4
	addl	$1, 28(%esp)	# ++i; // 5
	movl	28(%esp), %eax	# eax = i;
	addl	%eax, %eax		# eax = eax + eax; // 10
	addl	$1, 28(%esp)	# ++i; // 6
	addl	28(%esp), %eax	# eax = eax + i; // 16
	movl	%eax, 24(%esp)	# ans = eax;

`　　`然后，我们就知道了 16 是怎么来的了。

　　为什么我就肯定 28(%esp) 就是变量 i 呢？
因为只有它被写入了 3，没有别的内存被写入 3，
而且从之后操作它的各条指令也可以确定它就是 C 
代码中的变量 i（好悲催啊，
堂堂一个局部变量变成了无名无姓的相对寄存器寻址的一块内存！）。
类似的，局部变量 ans 变成了 24(%esp)，
可以推理得出：局部变量最后都变成了相对 esp 寻址的内存块
（之后的篇章会看到这个推论还不是很正确）。

## 延伸

　　同样的程序，在 VC 上编译运行，
Debug 模式的运行结果是 16，Release 模式的运行结果是 18；
而在 Visual Studio 2010 中 Debug 和 
Release 模式下都是 18。

　　VC 和 VS 也可以看反汇编代码，
在调试过程中遇到断点中断的时候，
VC 使用 Alt + 8 打开反汇编窗体，
VS 右击 C 源代码编辑区选择"转到反汇编"，打开反汇编窗体。

　　为什么 Intel 自己造的 CPU，AT&T 还出一套与 Intel 
不同格式的汇编语言呢？没法子，AT&T 也不是好惹的，
人家做了两样东西至今影响全世界：Unix、C语言。

## 小结

　　从这个例子中我们应该吸取经验：
<b>被实施递增（递减）操作的变量不应该在表达式中多次出现</b>，
否则结果就不受我们控制了，而是被编译器自由发挥：

　　C 标准规定：两个[序列点](http://blog.csdn.net/huiguixian/article/details/6438613)之间，
程序执行的顺序可以是任意的。
这样做给了编译器优化的空间。[[1]](#tip1)

　　如果想得到结果 15 的话，程序可以改成这样：

	#include <stdio.h>
	
	int main()
	{
	    int i = 3;
	    int ans = (i+2)*3;
		i += 3;
	
	    printf("%d\n",ans);
	    return 0;
	}

`　　`这个程序不论在 windows 下还是在 linux 下，
不论是 Release 还是 Debug，结果一定是 15。

　　关于 解剖 C 语言，才出了两篇，已经能够为我们解惑了，
悟空很有潜力啊！元芳，你怎么看？

[回目录][content]

<a name="tip1"></a>
[1] 由 ohyeah 指出：[http://rs.xidian.edu.cn/forum.php?mod=redirect&goto=findpost&ptid=412474&pid=8298351](http://rs.xidian.edu.cn/forum.php?mod=redirect&goto=findpost&ptid=412474&pid=8298351)
