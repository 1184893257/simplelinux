[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">函数帧
</h1>

　　这标题一念出来我立刻想到了一个名人：白素贞……当然，
此女与本文无关，下面进入正题：

<pre>其实程序运行就好比一帧一帧地放电影，每一帧是一次函数调用，电影放完了，我们就看到结局了。</pre>

　　我们用一个递归求解阶乘的程序来看看这个放映过程（fac.c）：

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

## main 帧

　　首先 main 函数被调用（程序可不是从 main 开始执行的）：

<table>
<tr><td>
<pre><code>main:
	pushl	%ebp
	movl	%esp, %ebp
	andl	$-16, %esp
	subl	$32, %esp
	movl	$3, 28(%esp)	# n = 3
	movl	28(%esp), %eax
	movl	%eax, (%esp)
	call	fac
	movl	%eax, 24(%esp)	# 返回值存入 ans
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
</code></pre></td>
<td><img src="http://fmn.rrimg.com/fmn056/20121124/1940/original_D0zG_726b00003e04118c.jpg" /></td>
</tr></table>

`　　`main 函数创建了一帧：

* 从 esp 到 ebp + 4
* 上边是本次调用的返回地址、旧的 ebp 指针
* 然后是 main 的局部变量 n、ans
* 最下边是参数的空间，右上图显示的是 main 中调用 printf 
前的栈的使用情况

`　　`进入 main 函数，前 4 条指令开辟了这片空间，
在退出 main 函数之前的 leave ret 回收了这片空间
（<b>C++ 在回收这片空间之前要析构此函数中的所有局部对象</b>）。
<b>在 main 函数执行期间 ebp 一直指向 帧顶 - 4 的位置，
ebp 被称为帧指针也就是这个原因</b>。

## 调用惯例

　　调用函数的时候，先传参数，然后 call，
具体这个过程怎么实现有相关规定，这样的规定被称为<b>调用惯例</b>，
C语言中有多种调用惯例，它们的不同之处在于：

1. 参数是压栈还是存入寄存器
2. 参数压栈的次序（从右至左 | 从左至右）
3. 调用完成后是调用者还是被调用者来恢复栈

`　　`各种调用惯例<em>《程序员的自我修养》——链接、装载与库</em>
这本书中有简要介绍，我照抄后在本文后面列出。C语言默认的
调用惯例是 cdecl：

1. 参数从右至左压栈
2. 调用完成后调用者负责恢复栈

`　　`可以从 printf("%d! = %d\n", n, ans); 的调用过程
中看出。

　　虽然 VC、gcc 都默认使用 cdecl 调用惯例，
但它们的实现却各有风格：

* VC 一般是从右至左 push 参数，call，add esp, XXX
* 而 gcc 在给局部变量分配空间的时候也给参数分配了足够的空间，
所以只要从右至左 mov 参数, XXX(%esp)，call 就可以了，
调用者根本不用去恢复栈，因为传参数的时候并没有修改栈指针 esp。

## fac 帧

　　说完调用惯例我们接着来看第一次调用 fac：

<table>
<tr><td>
<pre><code>fac:
	pushl	%ebp
	movl	%esp, %ebp
	subl	$24, %esp
	cmpl	$1, 8(%ebp)
	jg	.L2			# n > 1 就跳到 .L2
	movl	$1, %eax
	jmp	.L3			# 无条件跳到 .L3
.L2:
	movl	8(%ebp), %eax
	subl	$1, %eax
	movl	%eax, (%esp)
	call	fac		#  fac(n-1)
	imull	8(%ebp), %eax	# eax = n * eax
.L3:
	leave
	ret
</code></pre></td>
<td><img src="http://fmn.rrfmn.com/fmn058/20121124/1940/original_XcDN_08c400005ab9125d.jpg" /></td>
</tr></table>

　　fac(3) 开辟了第一个 fac 帧：

* 从 esp 到 ebp + 4（fac 还能"越界"地读到参数 n）
* 上边是 返回地址、旧的 ebp 指针（指向 main 帧）
* fac 没有局部变量，又浪费了很多字节
* 参数占了最下边的 4 字节（需要递归时使用）

`　　`这时还不满足递归终止条件，于是fac(3)又递归地调用了fac(2)，
fac(2)又递归的调用了fac(1)，到这个时候栈变成了如下情况：

![total](http://fmn.rrimg.com/fmn062/20121124/1940/original_y9zg_1a3800005b5a118e.jpg)

　　上图的箭头的含义很明显：
<b>从 ebp 可回溯到所有的函数帧</b>，
这是由于每个函数开头都来两条 pushl %ebp、movl %esp, %ebp造成的。

　　参数总是调用者写入，被调用者来读取（被调用者修改参数毫无意义），
这是一种默契^_^。

程序继续运行：

1. fac(1) 满足了递归终止条件，fac(1) 返回 1，fac(1)#3 帧消亡
2. 继续执行 fac(2)，fac(2) 返回 1\*2，fac(2)#2 帧消亡
3. 继续执行 fac(3)，fac(3) 返回 2\*3，fac(1)#1 帧消亡
4. 继续执行 main，printf 结果，返回 0，main 帧消亡
5. 继续执行 ？？？（且听下回分解）

最终程序结束（进程僵死，一会儿后操作系统会来收尸
（回收内存及其他资源））。

## 小结

　　函数帧保存的是函数的一个完整的局部环境，
保证了函数调用的正确返回（函数帧中有返回地址）、
返回后继续正确地执行，因此函数帧是 C语言 能调来调去的保障。

<h2 align="center">主要的调用惯例</h2>
<table border="1">
 <tr>
  <th>调用惯例</th>
  <th>出栈方</th>
  <th>参数传递</th>
  <th>名字修饰</th>
 </tr>
 <tr>
  <td>cdecl</td>
  <td>函数调用方</td>
  <td>从右至左的顺序压参数入栈</td>
  <td>下划线+函数名</td>
 </tr>
 <tr>
  <td>stdcall</td>
  <td>函数本身</td>
  <td>从右至左的顺序压参数入栈</td>
  <td>下划线+函数名+@+参数的字节数，
  如函数 int func(int a, double b)的修饰名是
  _func@12</td>
 </tr>
  <tr>
  <td>fastcall</td>
  <td>函数本身</td>
  <td>头两个 DWORD(4字节)类型或者更少字节的参数
  被放入寄存器，其他剩下的参数按从右至左的顺序入栈</td>
  <td>@+函数名+@+参数的字节数</td>
 </tr>
 <tr>
  <td>pascal</td>
  <td>函数本身</td>
  <td>从左至右的顺序入栈</td>
  <td>较为复杂，参见pascal文档</td>
 </tr>
 </table>

[回目录][content]
