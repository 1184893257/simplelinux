[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">C语言的栈是静态的
</h1>

　　C语言有了可变参数之后，我们可以传任意个数的参数了，
似乎挺动态的了，但是可变参数函数还是不够动态。

## 一、鞭长莫及

　　我们可以在 main 中写出好几条参数个数不同的调用 sum 的语句，
但是具体到某一条语句，sum 的参数个数是一定的，
比如上一篇中的 sum(2, 3, 4) 的参数个数是 3。
<b>如果程序运行中调用 sum 函数的时候，
参数个数根据用户输入而定，那就不能用可变参数来实现了。</b>
也就是说不能用 sum 来实现以下这个函数的功能：

	// 将数组 a 的所有元素（个数为 n）求和后返回
	int d_sum(int n, int a[]);

`　　`当然，这个函数不用 sum 来做是很好实现的。
我再换一个问题，下面这个函数怎么用 printf 来实现：

	// fmt 存的是格式串，它描述了 n 个整数（数组 a 中）
	// 的格式，某次调用如下：
	//   int a[] = {1, 2, 3};
	//   d_printf("%d+%d=%d", 3, a);
	void d_printf(const char *fmt, int n, int a[]);

`　　`这就没法做了吧！

## 二、寻根究底

　　d_printf 没法实现的原因是这样的代码真没法写：
传给 printf 的参数的个数到运行的时候才知道，
而调用 printf 的语句又必须明确的列出所有参数。

　　其根本原因是<b>C语言的栈是静态的</b>，
上一篇的 va.c 编译后的汇编代码如下：

	main:
		pushl	%ebp
		movl	%esp, %ebp
		andl	$-16, %esp
		subl	$16, %esp	# 给main帧分配栈空间
		movl	$4, 8(%esp)
		movl	$3, 4(%esp)
		movl	$2, (%esp)
		call	sum			# 调用变参函数 sum
		movl	$.LC0, (%esp)
		movl	%eax, 4(%esp)
		call	printf		# 调用变参函数 printf
		xorl	%eax, %eax
		leave
		ret

`　　`可以看到虽然 main 函数中调用了两个变参函数，
但是栈却没有一点动态可变的意思，居然是用一条 subl $16, %esp 
分配了固定的 16 字节的栈空间
（编译的时候计算得出需要12字节，取整吧，16字节！）。

　　而在 d_printf 的实现中需要分配 4+4*n 字节的栈空间，
用于存传给 printf 的 格式串指针 和 n个整数，
用C语言是没法实现啰。

## 三、另辟蹊径

　　C语言不能直接使用寄存器，但是汇编可以，
如果我们在 d_printf 中嵌入一段汇编来修改 esp 寄存器，
达到动态分配栈空间的效果，然后存入参数，call printf，
就可以完成任务了。

　　接下来的两篇就来实现 d_printf 啰！

[回目录][content]
