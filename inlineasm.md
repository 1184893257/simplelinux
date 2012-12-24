[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">内联汇编
</h1>

　　内联汇编是指在 C/C++ 代码中嵌入的汇编代码，
与全部是汇编的汇编源文件不同，它们被嵌入到 C/C++ 的大环境中。

## 一、gcc 内联汇编

　　gcc 内联汇编的格式如下：

    asm ( 汇编语句
        : 输出操作数		// 非必需
        : 输入操作数		// 非必需
        : 其他被污染的寄存器	// 非必需
        );

`　　`我们通过一个简单的例子来了解一下它的格式（gcc_add.c）：

	#include <stdio.h>
	
	int main()
	{
		int a=1, b=2, c=0;
	
		// 蛋疼的 add 操作
		asm(
			"addl %2, %0"		// 1
			: "=g"(c)			// 2
			: "0"(a), "g"(b)	// 3
			: "memory");		// 4
	
		printf("现在c是:%d\n", c);
		return 0;
	}

`　　`内联汇编中：

1. 第1行是汇编语句，用双引号引起来，
<b>多条语句用 ; 或者 \n\t 来分隔。</b>
2. 第2行是输出操作数，都是 "=?"(var) 的形式，
var 可以是任意内存变量（输出结果会存到这个变量中），
? 一般是下面这些标识符
（表示内联汇编中用什么来代理这个操作数）：
	* a,b,c,d,S,D 分别代表 eax,ebx,ecx,edx,esi,edi 寄存器
	* r 上面的寄存器的任意一个（谁闲着就用谁）
	* m 内存
	* i 立即数（常量，只用于输入操作数）
	* g 寄存器、内存、立即数 都行（gcc你看着办）

	<b>在汇编中用 %序号 来代表这些输入/输出操作数，
序号从 0 开始。为了与操作数区分开来，
寄存器用两个%引出，如：%%eax</b>
3. 第3行是输入操作数，都是 "?"(var) 的形式，
<b>? 除了可以是上面的那些标识符，还可以是输出操作数的序号，
表示用 var 来初始化该输出操作数</b>，
上面的程序中 %0 和 %1 就是一个东西，初始化为 1（a的值）。
4. 第4行标出那些在汇编代码中修改了的、
又没有在输入/输出列表中列出的寄存器，
这样 gcc 就不会擅自使用这些"危险的"寄存器。
还可以用 "memory" 表示在内联汇编中修改了内存，
之前缓存在寄存器中的内存变量需要重新读取。

`　　`上面这一段内联汇编的效果就是，
把a与b的和存入了c。当然这只是一个示例程序，
谁要真这么用就蛋疼了，
<b>内联汇编一般在不得不用的情况下才使用</b>。

## 二、VC 内联汇编

　　gcc 内联汇编被设计得很复杂，初学者看了往往头大，
而 VC 的内联汇编就简单多了：

	__asm{
		汇编语句
	}

`　　`一个例子程序如下（vc_add.c）：

	#include <stdio.h>
	
	int main()
	{
		int a=1, b=2, c=0;
	
		// 蛋疼的 add 操作
		__asm{
			push eax	// 保护 eax
	
			mov eax, a	// eax = a;
			add eax, b	// eax = eax + b;
			mov c, eax	// c = eax;
	
			pop eax		// 恢复 eax
		}
	
		printf("现在c是:%d\n", c);
		return 0;
	}

`　　`VC 的内联汇编中可以直接以变量名的形式使用局部变量，
这就方便多了。<b>但是，
VC 内联汇编中有些变量名是保留的，比如：size，
使用这些变量名就会报错（把b改成size，
上面的程序就编译不通过了）。所以，起名字一定要小心！</b>

　　因为 VC 没有输入/输出操作数列表，
它也不看你的汇编代码（直接拿去用），
所以它不知道你修改了哪些寄存器，
这些要修改的寄存器可能保存着重要数据，
所以用 push/pop 来 保护/恢复 要修改的寄存器。
而 gcc 就不需要，它能从输入/输出列表中获得丰富的信息
来调剂各个寄存器的使用，
并进行优化，所以从效率上说 VC 完败！

## 三、为什么用内联汇编

　　用内联汇编的主要目的是为了提高效率：
假设有一个比较文本差异的程序 diff，
它花了 99% 的时间在 strcmp 这个函数上，
如果用内联汇编实现的一个高效的 strcmp 比用 C 语言实现的快
 1 倍，那么专家花在这个小小函数上的心思就能够将整个程序的效率
提高差不多 1 倍，这是很值得去做的"斤斤计较"。

　　还有一个目的就是为了实现 C 语言无法实现的部分，
比如说 IO 操作，还有我们上一篇中提到的自主修改 esp 寄存器
也是必须用汇编才能实现的。

## 四、memcpy

　　学 gcc 内联汇编最好的导师莫过于 linux 内核，
有很多常用的小函数如 memcpy、strlen、strcpy、……
其中都有短小精悍的内联汇编版本，
如在 linux 2.6.37 中的 memcpy 函数：

	// 位于 /arch/x86/boot/compressed/misc.c
	void *memcpy(void *dest, const void *src, size_t n)
	{
		int d0, d1, d2;
		asm volatile(
			"rep ; movsl\n\t"
			"movl %4,%%ecx\n\t"
			"rep ; movsb\n\t"
			: "=&c" (d0), "=&D" (d1), "=&S" (d2)
			: "0" (n >> 2), "g" (n & 3), "1" (dest), "2" (src)
			: "memory");
	
		return dest;
	}

`　　`与 gcc_add.c 相比，这个函数要复杂不少：

* 关键字 volatile 是告诉 gcc 不要尝试去移动、
删除这段内联汇编。
* rep ; movsl 的工作流程如下：

		while(ecx) {
			movl (%esi), (%edi);
			esi += 4;
			edi += 4;
			ecx--;
		}

	rep ; movsb 与此类似，只是每次拷贝的不是双字（4字节），
而是字节。
* <b>"=&D" (d1) 不是想将 edi 的最终值输出到 d1 中，
而是想告诉 gcc edi的值早就改了，
不要认为它的值还是初始化时的 dest，
避免"吝啬的" gcc 把修改了的 edi 还当做 dest 来用。</b>
而 d0、d1、d2 在开启优化后会被 gcc 无视掉
（输出到它们的值没有被用过）。

`　　`memcpy 先复制一个一个的双字，
到最后如果还有没复制完的（少于4个字节），
再一个一个字节地复制。
我最终实现的 d_printf 就模仿了这个函数。

<b>深入研究：</b><br>
<a target="_blank" href="http://www.ibiblio.org/gferg/ldp/GCC-Inline-Assembly-HOWTO.html">gcc 内联汇编 HOWTO 文档</a><br>
<a target="_blank" href="http://lxr.free-electrons.com/ident">Linux Cross Reference——各版本 linux 内核函数检索</a>

[回目录][content]
