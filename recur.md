[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">所有递归都可以变循环
</h1>

　　这是函数帧的应用之二。

　　还记得大一的C程序设计课上讲到汉诺塔的时候老师说：
<b>所有递归都可以用循环实现</b>。这听起来好像可行，
然后我就开始想怎么用循环来解决汉诺塔问题，
我大概想了一个星期，最后终于选择了……放弃……
当然，我不是来推翻标题的，
随着学习的深入，以及"自觉修炼"，现在我可以肯定地告诉大家：
所有递归都可以用循环实现，更确切地说：
<b>所有递归都可以用循环+栈实现</b>
（就多个数据结构，还不算违规吧O(∩_∩)O~）。

　　<b>通过在我们自定义的栈中自建函数帧，
我们可以达到和函数调用一样的效果</b>。
但是因为这样做还是比较麻烦，所以就不转换汉诺塔问题了，
而是转换之前的那个递归求解阶乘的程序（fac.c）：

	#include <stdio.h>
	
	int fac(int n)
	{
		if(n <= 1)
			return 1;
		return n * fac(n-1);
	}
	
	int main()
	{
		int n = 3;
		int ans = fac(n);
		
		printf("%d! = %d\n", n, ans);
		return 0;
	}

## 技术难点

　　我们可以在自建的函数帧中存储局部变量、存储参数，
<b>但是我们不能存返回地址，因为我们得不到机器指令的地址！</b>
不过，C语言有一个类似于指令地址的东西：switch case
中的 case子句，我们可以用一个case代表一个地址，
技术难点就此突破了。

## 源程序

　　虽然我简化了很多步骤，但源程序还是比较长（fac2.c）：

	#include <stdio.h>
	
	// 栈的设置
	#define STACKDEEPTH	1024
	int stack[STACKDEEPTH];
	int *esp = &stack[STACKDEEPTH];
	#define PUSH(a)	*(--esp) = a
	#define POP(b)	b = *(esp++)
	
	// 其它模拟寄存器
	int eax;// 存返回值
	int eip;// 用于分支选择
	
	int main()
	{
		int n = 3;
		
		// 模仿 main 调用 fac(n)
		PUSH(n);
		PUSH(10002);// 模仿返回 main 的地址
		eip = 10000;
		
		do{
			switch(eip){
			case 10000:
				--esp;// 为帧分配空间
				if(esp[2] <= 1){// 模仿递归终止条件
					eax = 1;
					++esp;// 回收帧空间
					POP(eip);
				}else{// 模仿递归计算 fac(n-1)
					esp[0] = esp[2] - 1;
					PUSH(10001);
					eip = 10000;
				}
				break;
				
			case 10001:// 返回 n * (fac(n-1)的结果)
				eax = esp[2] * eax;
				++esp;// 回收帧空间
				POP(eip);
				break;
			}
		}while(eip != 10002);
		
		printf("%d! = %d\n", n, eax);
		return 0;
	}

## 自建的函数帧

　　为了简化程序，ebp我们就不用了，
完全用esp来操作栈，一个函数帧只占用 8 个字节：

![recur1](http://fmn.rrimg.com/fmn063/20121130/1830/original_OJ3e_0ad000003200125b.jpg)

　　在计算到 fac(1) 的时候，栈中内容如下：

![recur2](http://fmn.rrimg.com/fmn056/20121130/1830/original_LBeS_30f500003160118f.jpg)

　　比起肆意挥霍栈空间的 gcc（fac帧用了32字节，
浪费了20字节，实际使用了12字节），
我们的程序真的是太节省了（一帧只用8字节）。

## 小结

　　当然，本文的方法只用于学术讨论，
说明所有递归都可以变循环，编程的时候还是不要这么用。
因为代码复杂、容易出错、难以理解，
唯一的优点是能省空间省到极限。

　　这种递归变循环的方式并没有降低时间复杂度，
但却是通用的（所有递归都可以这么变循环）；
而有一部分递归可以基于巧妙的算法变成循环，
并且大大降低时间复杂度，如：动态规划、贪心算法
（详见《算法导论》）。

[回目录][content]
