[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">汇编实现的动态栈
</h1>

　　这一篇就是实现 d_printf，废话不多说，直接上代码。
由于 VC 的内联汇编还是比较清晰，那就先贴 VC 版的。

## 一、d_printf VC版

	#include <stdio.h>
	
	void d_printf(const char *fmt, int n, int a[])
	{
		static int size1, size2;
		static const char *fmt_copy;
	
		size1 = 4*n;		// 可变参数的空间大小
		size2 = size1 + 4;	// 还有 fmt 指针4字节, 恢复 esp 时用
		fmt_copy = fmt;
		
		__asm{
			// 保护要修改的 ecx/esi/edi 寄存器
			push ecx
			push esi
			push edi
	
			// 给 ecx/esi/edi 赋值
			mov ecx, n	// movsd 的执行次数
			mov esi, a	// a -> esi
			sub esp, size1
			mov edi, esp	// esp - size1 -> edi
	
			rep	movsd		// n 次4字节拷贝
			push fmt_copy	// 压栈格式串(字符串指针)
			call printf
	
			add esp, size2	// 恢复栈
			// 恢复各个寄存器
			pop edi
			pop esi
			pop ecx
		}
	}
	
	int main()
	{
		char fmt[1024];	// 格式串
		char c;			// 额外读取一个字符
		int a[1024];	// 存读到的整数
		int i;
		
		while(EOF != scanf("%s%c", fmt, &c)) // 读到 EOF 就结束
		{
			if(c == '\n') // 格式串后没有数字
			{
				printf("%s\n\n", fmt); // 直接打印, 不用 d_printf
				continue;
			}
	
			// 循环读取各个整数
			i = 0;
			do
			{
				scanf("%d%c", &a[i++], &c);
			}while(c != '\n');
	
			// 调用 d_printf, i 刚好是输入的整数的个数
			d_printf(fmt, i, a);
			printf("\n\n"); // 补个换行比较好看O(∩_∩)O~
		}
	}

## 二、d_printf gcc 版（main 函数跟 VC 版的一样）

	#include <stdio.h>
	
	void d_printf(const char *fmt, int n, int a[])
	{
		int d0, d1, d2;
		
		static int size1, size2;
		static const char *fmt_copy;
	
		size1 = 4*n;		// 可变参数的空间大小
		size2 = size1 + 4;	// 还有 fmt 指针4字节, 恢复 esp 时用
		fmt_copy = fmt;
		
		asm volatile(
			"subl %6, %%esp\n\t"
			"movl %%esp, %%edi\n\t"
			"rep ; movsl\n\t"
			"pushl %3\n\t"
			"call printf\n\t"
			"addl %7, %%esp"
			: "=&S"(d0), "=&D"(d1), "=&c"(d2)
			: "m"(fmt_copy), "0"(a), "2"(n), "m"(size1), "m"(size2));
	}
	
	int main()
	{
		char fmt[1024];	// 格式串
		char c;			// 额外读取一个字符
		int a[1024];	// 存读到的整数
		int i;
		
		while(EOF != scanf("%s%c", fmt, &c)) // 读到 EOF 就结束
		{
			if(c == '\n') // 格式串后没有数字
			{
				printf("%s\n\n", fmt); // 直接打印, 不用 d_printf
				continue;
			}
	
			// 循环读取各个整数
			i = 0;
			do
			{
				scanf("%d%c", &a[i++], &c);
			}while(c != '\n');
	
			// 调用 d_printf, i 刚好是输入的整数的个数
			d_printf(fmt, i, a);
			printf("\n\n"); // 补个换行比较好看O(∩_∩)O~
		}
	}

## 三、运行效果

　　linux 中的运行效果如下：

	[lqy@localhost temp]$ ./d_printf 
	nospaceword
	nospaceword
	
	%X 256
	100
	
	%d+%d=%d 1 2 3
	1+2=3
	
	>%3d>%03d> 3 3
	>  3>003>
	
	>%3d>%-3d> 3 3
	>  3>3  >
	
	[lqy@localhost temp]$ 

`　　`最后输入 EOF 结束：Ctrl + D（linux）、Ctrl + Z
（windows）。<b>由于转义符是编译时处理的，printf 是不管的，
所以\n什么的在这里不管用^_^</b>。

## 四、为什么用 static

　　d\_printf 中为什么将 size1、size2、fmt_copy 声明为
static 变量呢？

1. 不能用寄存器。size2 是在 call printf 之后使用的，
<b>但是后来我发现 printf 执行完后改动了好几个寄存器的值</b>，
所以如果将 size2 保存到某个寄存器中是不行的
（难怪C语言喜欢内存，寄存器太不可靠了o(╯□╰)o）。
2. 不能用局部变量。不让用寄存器就用内存呗！
<b>但是我们要自主修改 esp，
而局部变量有可能是通过 esp+常量偏移 定位的</b>
（如果是用 ebp+常量偏移（VC一般这么用）定位的就没问题），
所以 subl %6, %%esp 之后 到 addl %7, %%esp 之前
都不能使用局部变量，否则会定位错误。
所以才使用 static 变量，<b>
因为 static 变量是用绝对地址定位的，跟 esp 毫无关系</b>。

## 五、使用内联汇编的建议

1. 不要用。
2. 如果非得用的话，尽量用 C 语言实现+-*/，
如 d_printf 中给 size1、size2 赋值；
内联汇编只实现不得不用的部分（内联汇编一般都短小精悍）。

<h2 align="center">《解剖C语言》就此完结。</h2>

[回目录][content]
