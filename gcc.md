[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">照妖镜和火眼金睛
</h1>

　　如果在 linux 下编写 C 程序，
那么你将获得两个犀利的法宝：

---

## 照妖镜

　　一个C程序（max.c）：

	#define MAX(a,b) ((a)>=(b)?(a):(b))
	
	int main(){
		int c=MAX(1,2); // 注注注注释
		return 0;
	}

`　　`程序很简单，就是定义和使用一个MAX宏，
宏在正式编译前是会被替换为本来面目的，
我们现在看到的不是它的真身。让我们用照妖镜来照照：

	gcc -E -o max2.c max.c

`　　`<em>这里的 -o max2.c 是让 gcc 
把要输出东西输出到 max2.c 文件中</em>。

　　妖怪！快快现形吧：

	# 1 "max.c"
	# 1 "<built-in>"
	# 1 "<命令行>"
	# 1 "max.c"
	
	
	int main(){
	 int c=((1)>=(2)?(1):(2));
	 return 0;
	}

`　　`上面就是max2.c中的内容，MAX(1,2) 被替换成了 
((1)>=(2)?(1):(2))，这只孽畜终于现形了！

　　照妖镜的作用就是替换宏，但是宏好像大家都不太用。
不过宏在 现代 linux 内核源代码中简直是运用到了极致，
甚至可以说 linux 内核是由 C、宏、汇编 写出来的。
宏是可以嵌套的，也就是说宏的 参数 或 右部 
中还可以出现能够被替换的宏，
所以情况就相当复杂了——十个字符的简单的一条语句，
当被还原为本来面目时，可能就变成七八十个字符了，
要分析这样的语句，照妖镜就大显神威了。

　　关于宏，后面会独立出一篇来介绍。

---

## 火眼金睛

　　照妖镜应该是不如火眼金睛的，
火眼金睛可以看到及其微小的细节。下面我写了个Hello World
（hello.c）：

	#include <stdio.h>

	int main(){
		printf("Hello, World!\n");
		return 0;
	}

`　　`Hello World 就不用解释了吧，鼎鼎有名啊！O(∩_∩)O~，
然后我们用火眼金睛来看一下：

	gcc -S -o hello.s hello.c

`　　`Hello World 的汇编版就出来了(hello.s)：

		.file	"hello.c"
		.section	.rodata
	.LC0:
		.string	"Hello, World!"
		.text
	.globl main
		.type	main, @function
	main:
		pushl	%ebp
		movl	%esp, %ebp
		andl	$-16, %esp
		subl	$16, %esp
		movl	$.LC0, (%esp)
		call	puts
		movl	$0, %eax
		leave
		ret
		.size	main, .-main
		.ident	"GCC: (GNU) 4.5.1 20100924 (Red Hat 4.5.1-4)"
		.section	.note.GNU-stack,"",@progbits

`　　`其内容我们后面再慢慢分析，
现在只要知道怎么用“火眼金睛”就行了，
接下来的几篇都得靠悟空了。

---

　　照妖镜和火眼金睛其实都是靠截断编译过程
得到中间产物的，gcc的完整编译过程是：

	预处理->编译->汇编->链接

`　　`使用不同的编译选项可以得出不同的中间产物：

<table border="1">
 <tr>
  <th>编译阶段</th>
  <th>命令</th>
  <th>截断后的产物</th>
 </tr>
 <tr>
  <td></td>
  <td></td>
  <td>C源程序</td>
 </tr>
 <tr>
  <td>预处理</td>
  <td>gcc -E</td>
  <td>替换了宏的C源程序(没有了#define,#include…),
	删除了注释</td>
 </tr>
 <tr>
  <td>编译</td>
  <td>gcc -S</td>
  <td>汇编源程序</td>
 </tr>
 <tr>
  <td>汇编</td>
  <td>gcc -c</td>
  <td>目标文件，二进制文件，
  允许有不在此文件中的外部变量、函数</td>
 </tr>
 <tr>
  <td>链接</td>
  <td>gcc</td>
  <td>可执行程序，一般由多个目标文件或库链接而成，
	二进制文件，所有变量、函数都必须找得到</td>
 </tr>
</table>

　　也许有同学发现了 -c 我还没讲呢！
二进制文件的分析后面也有用到，但是很少，用到的时候再说吧。

[回目录][content]
