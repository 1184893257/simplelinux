[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">结构体
</h1>

　　结构体是 C 语言主要的自定义类型方案，
这篇就来认识一下结构体。

## 一、结构体的形态

　　C源程序（struct.c）：

	#include <stdio.h>
	
	typedef struct{
		unsigned short int a;
		unsigned short int b;
	}Data;
	
	int main()
	{
		Data c, d;
		
		c.a = 1;
		c.b = 2;
		d = c;
		
		printf("d.a:%d\nd.b:%d\n", d.a, d.b);
		return 0;
	}

`　　`赋值部分翻译后：

		movw	$1, 28(%esp)	# c.a = 1
		movw	$2, 30(%esp)	# c.b = 2
		movl	28(%esp), %eax	#
		movl	%eax, 24(%esp)	# d = c

`　　`可以看出：

* c.a 是在 28(%esp) 之后的2个字节
* c.b 是在 30(%esp) 之后的2个字节
* c 是 28(%esp) 之后的4个字节
* d 是 24(%esp) 之后的4个字节

`　　`不得不感叹名字（结构体名字、子元素名字）再一次被抛弃了，
子元素名代表的是相对于结构体的偏移。

## 二、结构体的复制

　　大一的时候，老师千叮咛万嘱咐：<b>数组不能复制！</b>，
但是当发现下面这个程序正常运行后，我困惑了（block.c）：

	#include <stdio.h>
	
	typedef struct{
		char data[1000];
	}Block;
	
	Block a={{'a','b','c',}};
	
	int main()
	{
		Block b;

		b=a;
		
		puts(b.data);
		return 0;
	}

`　　`Block a={{'a','b','c',}} 是对 a 的部分初始化，
'c' 后面自动填 0，写成 Block a={{"abc"}} 也一样，
C 语言对初始化还是很宽容的。

　　上面这个程序居然正常的编译、运行了，这究竟是怎样的逆天？
看看汇编部分：

	leal	24(%esp), %edx
	movl	$a, %ebx
	movl	$250, %eax
	movl	%edx, %edi	# edi = &b
	movl	%ebx, %esi	# esi = &a
	movl	%eax, %ecx	# ecx = 250
	rep movsl

`　　`我们发现程序确实通过 250 次 movsl 复制了一个"数组"。
其原因是：结构体是可以复制的，
结构体又可以包括任意类型的子元素，数组也行，
所以"数组"也被复制了。

　　那为什么纯粹的数组就不能复制呢？
我们可以这样去理解：<b>一个变量能被复制的必要条件是
我们知道它的大小</b>。结构体做为自定义类型，
在编译的时候编译器必然存储了它的子元素类型、个数等相关信息，
结构体的大小也就知道了；而数组一般只在乎它的类型和
起始地址，元素个数总是被忽视的（例如：
void func(char s[]) 可接受任何长度的字符数组做参数），
而且元素个数也没有被当做数组的一部分存入内存，
所以数组的复制是不好实现的。

## 小结

　　如果给结构体下一个实在点的定义话，那就是：
<b>有格式的字节数组</b>。有了结构体后 C 语言的
变量类型就丰富多了，但是同时也要注意：

1. 超过 4 字节的结构体不宜做参数（参数传递浪费时间、空间），
换做指针更好。
2. 超过 4 字节的结构体不宜做返回值类型
（话说一般返回值都用 eax 来存，
那么超过 4 字节的时候怎么存呢？自己去探索吧！）。

[回目录][content]
