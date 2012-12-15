[content]: https://github.com/1184893257/simplelinux/blob/master/README.md#content

[回目录][content]

<a name="top"></a>

<h1 align="center">变量名、函数名
</h1>

　　C程序在执行的时候直接用内存地址去定位变量、函数，
而不是根据名字去搜索，所以C程序执行的速度比脚本语言要快不少。

　　对于函数中的局部变量来说，编译为汇编的时候，
名字就已经被彻彻底底地忘记了，
因为局部变量在函数帧中，这一帧要占多少字节，
各局部变量在帧中的相对位置，
都在编译成汇编的时候就可以确定下来，
生成目标文件、可执行文件的时候也不需要再更改。

　　而 全局变量、static变量、函数 由于要将所有目标文件、
库链接到一起之后才能最终确定它们的绝对地址，
所以在链接前名字还是标志着它们的存在。
它们的信息存储在符号表（符号数组）中，
其中每一项除了有符号名，还有符号地址（链接后填入），
所以 nm 命令可得到 地址-符号名 映射。
虽然程序运行时用不到符号表，
但是默认情况下可执行文件中还是存着符号表，
看下面这个程序（name.c）：

	#include <stdio.h>
	
	int globalvar;
	
	int main()
	{
		static int staticval;
		return 0;
	}

`　　`name.c 中有全局变量、static变量、函数(main)，
查看它编译后的目标文件、可执行文件的 地址-符号 映射：

	[lqy@localhost notlong]$ gcc -c name.c
	[lqy@localhost notlong]$ nm name.o
	00000004 C globalvar
	00000000 T main
	00000000 b staticval.1672
	[lqy@localhost notlong]$ gcc -o name name.c
	[lqy@localhost notlong]$ nm name | sort
	08048274 T _init
	080482e0 T _start
	08048310 t __do_global_dtors_aux
	08048370 t frame_dummy
	08048394 T main
	...
	此处省略X行
	...
	08049604 b staticval.1672
	08049608 B globalvar
	0804960c A _end
	         U __libc_start_main@@GLIBC_2.0
	         w __gmon_start__
	         w _Jv_RegisterClasses
	[lqy@localhost notlong]$ 

`　　`可执行文件中的 地址-符号 映射还有什么存在的意义呢？
它可用于汇编级调试的时候设置断点，
比如linux内核编译后就生成了 System.map 文件，
便于进行内核调试：

	00000000 A VDSO32_PRELINK
	00000040 A VDSO32_vsyscall_eh_frame_size
	000001d3 A kexec_control_code_size
	00000400 A VDSO32_sigreturn
	0000040c A VDSO32_rt_sigreturn
	00000414 A VDSO32_vsyscall
	00000424 A VDSO32_SYSENTER_RETURN
	01000000 A phys_startup_32
	c1000000 T _text
	c1000000 T startup_32
	c1000054 t default_entry
	c1001000 T wakeup_pmode_return
	c100104c t bogus_magic
	c100104e t save_registers
	c100109d t restore_registers
	c10010c0 T do_suspend_lowlevel
	c10010d6 t ret_point
	c10010e8 T _stext
	c10010e8 t cpumask_weight
	c10010f9 t run_init_process
	c1001112 t init_post
	c10011b0 T do_one_initcall
	...

[回目录][content]
