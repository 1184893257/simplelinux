[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">奇怪的宏
</h1>

　　这一篇介绍这些奇怪的宏：

## 一、do while(0)

　　为了交换两个整型变量的值，前面<em>值传递</em>中已经用
包含指针参数的 swap 函数做到了，这次用<b>宏</b>来实现（swap.c）：

	#include <stdio.h>
	
	#define SWAP(a,b)		\
		do{					\
			int t = a;		\
			a = b;			\
			b = t;			\
		}while(0)
	
	int main()
	{
		int c=1, d=2;
		int t;	// 测试 SWAP 与环境的兼容性
		
		SWAP(c,d);
		
		printf("c:%d d:%d\n", c, d);
		return 0;
	}

`　　`这个宏看起来就有点怪了：do while(0) 是写了个循环
又不让它循环，蛋疼啊！其实不然，这样写是有妙用的：

　　<b>首先</b>，SWAP 有多条语句，如果这样写：

	#define SWAP(a,b)		\
			int t = a;		\
			a = b;			\
			b = t;

`　　`那么用的时候就得这么用：

	SWAP(c,d)

`　　`<b>不能加分号</b>！不习惯吧？

　　<b>其次</b>，使用 do{...}while(0)，
中间的语句用大括号括起来了，所以是另一个命名空间，
<b>其中的新变量 t 不会发生命名冲突</b>。

　　SWAP 宏要比之前那个函数的效率要高，
因为没有发生函数调用，没有参数传递，
宏会在编译前被替换，所以只是嵌入了一小段代码。

## 二、&#35;

　　标题我没打错，这里要说的就是井号，#的功能是将其后面的
宏参数进行字符串化操作。比如下面代码中的宏： 

	#define WARN_IF(EXP) \
	do{ if (EXP) \
		fprintf(stderr, "Warning: " #EXP "\n"); } \
	while(0) 

`　　`那么实际使用中会出现下面所示的替换过程： 

	WARN_IF (divider == 0); 


`　　`被替换为 

	do { if (divider == 0) 
		fprintf(stderr, "Warning: " "divider == 0" "\n"); 
	} while(0); 

`　　`需要注意的是<b>C语言中多个双引号字符串放在一起
会自动连接起来</b>，所以如果 divider 为 0 的话，就会打印出：

	Warning: divider == 0

## 三、&#35;&#35;

　　# 还是比较少用的，## 却比较流行，
在 linux0.01 中就用到过。## 被称为连接符，
用来将两个 记号（编译原理中的词汇） 连接为一个 记号。
看下面的例子吧（add.c）：

	#include <stdio.h>
	
	#define add(Type)				\
	Type add##Type(Type a, Type b){	\
		return a+b;					\
	}
	
	// 下面两条是奇迹发生的地方
	add(int)
	add(double)
	
	int main()
	{
		int a = addint(1, 2);
		double d = adddouble(1.5, 1.5);
		
		printf("a:%d d:%lf\n", a, d);
		return 0;
	}

`　　`那两行被替换后是这个样子的：

	int addint(int a, int b){ return a+b; }
	double adddouble(double a, double b){ return a+b; }

<b>以上内容都可以使用<em>照妖镜</em>看到宏被替换后的情形</b>。

[回目录][content]
