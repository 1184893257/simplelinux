[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">可变参数
</h1>

　　	C语言的可变参数的实现非常巧妙：
大师只用了 3 个宏就解决了这个难题。

## 一、可变参数的应用

　　这里实现一个简单的可变参数函数 sum：
它将个数不定的多个整型参数求和后返回，
其第 1 个参数指明了要相加的数的个数（va.c）：

	#include <stdio.h>
	#include <stdarg.h>
	
	// 要相加的整数的个数为 n
	int sum(int n, ...)
	{
	    va_list ap;
	    va_start(ap, n);
	
	    int ans = 0;
	    while(n--)
	        ans += va_arg(ap, int);
	
	    va_end(ap);
	    return ans;
	}
	
	int main()
	{
	    int ans = sum(2, 3, 4);
	    printf("%d\n", ans);
	
	    return 0;
	}

`　　`sum 函数的第一个参数是 int n，
逗号后面是连续的 3 个英文句点，
表示参数 n 之后可以跟 0、1、2…… 个任意类型的参数。
sum 可以这么用：

	sum(0);
	sum(1, 2);
	sum(3, 1, 1, 1);

## 二、可变参数的实现

　　可以看到在 sum 函数中用到了 3 个函数一样的东西：
va\_start、va\_arg、va\_end，
它们是标准库（意味着各种平台都有）头文件 stdarg.h 中
定义的宏，这 3 个宏经过清理后是下面这个样子：

	typedef char* va_list;
	#define va_start(ap,v)  ( ap = (va_list)(&v) + sizeof(v) )
	#define va_arg(ap,t)    ( *(t *)((ap += sizeof(t)) - sizeof(t)) )
	#define va_end(ap)      ( ap = NULL )

* va\_start 将 ap 定位到可变参数列表的起始地址
* va\_arg 每次返回一个参数，并后移 ap 指针
* va\_end 将 ap 置 NULL（避免非法使用）

`　　`这 3 个宏的实现就是基于 C语言默认调用惯例是从右至左
将参数压栈的事实，比如说 va.c 中调用 sum 函数，
参数压栈的顺序为：4->3->2，
又因为 x86 CPU 的栈是向低地址增长的，
所以参数的排列顺序如下：

![args](http://fmn.rrimg.com/fmn062/20121221/1930/original_qweH_1b90000008bb125c.jpg)

　　va\_start(n, ap)
就是 ( ap = (char*)(&n) + 4 )
因此 ap 被赋值为 ebp+12 也就是变参列表的起始地址。

　　之后 va\_arg  取出每一个参数：
( *(int *)((ap += 4) - 4) )
它首先将变参指针 ap 右移到下一个参数的起始地址，
再将加赋操作的返回值减到之前的位置取出一个参数。
这样，<b>用一条语句既取出了当前参数，又后移了指针 ap</b>，
真是神了！

　　sum 中循环使用 va\_arg 就取出了 n 个要相加的整数。

## 三、变参函数的可行性

　　一个变参函数能接受个数、类型可变的参数，
需要满足以下两个条件：

1. 能定位到可变参数列表的起始地址
2. 能获知可变参数的个数、每个参数的大小（类型）

`　　`条件 1 只要有个前置参数就能满足，
而对于这样的变参函数：void func(...);
编译能通过，但是不能用 va_start 取到变参列表的起始地址，
所以基本不可行。

<hr width="50%">

　　sum 函数中参数 n 被用来定位可变参数列表的起始地址
（满足条件1）；n 的值是可变参数的个数，
类型默认全部是 int 型（满足条件2），
因此 sum 能正常工作。

<hr width="50%">

　　再看看 printf 函数是如何满足以上两个条件的，
printf 函数的原型是：

	int printf(const char *fmt, ...);

`　　`printf 的第1个参数 fmt（格式串）被用来定位其后
的可变参数的起始地址（满足条件1）；
fmt 指向的字符串中的各个格式描述符如：%d、%lf、%s 等
告诉了 printf fmt 之后参数的个数、各个参数的类型
（满足条件2），因此 printf 能正常工作。

<hr width="50%">

　　当然，sum、printf 能正常工作是设计者一厢情愿的期望，
如果使用者不按规矩传入参数、格式串，函数能正常工作才怪！
比如：

	sum(2, "111", "222");
	printf("%s", 0);

`　　`编译器可不会进行可变参数的类型检查、格式串-参数匹配，
后果将会在运行的时候出现……

[回目录][content]
