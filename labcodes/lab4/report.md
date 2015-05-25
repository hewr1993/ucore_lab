#Lab4 Report
-----
## Ex1
1. alloc_proc函数（位于kern/process/proc.c中）负责分配并返回一个新的struct proc_struct结构，用于存储新建立的内核线程的管理信息。ucore需要对这个结构进行最基本的初始化，你需要完成这个初始化过程。

	> * 按照提示完成代码即可
	> * 大体思路是全体清零
	> * 实现细节见代码

1. 请说明proc_struct中struct context context和struct trapframe *tf成员变量含义和在本实验中的作用是啥？（提示通过看代码和编程调试可以判断出来）

	> * context结构定义在proc.h
		* 通过观察成员变量的名称，可看出它保存了各寄存器信息
		* 在proc_run的进程调度函数中，也可以看出它保存进程运行时的寄存器数据信息，如堆栈指针等
	> * trapframe猜测与中断过程相关
		* kernel_thread和copy_thread函数里对其进行了一些调用和初始化
		* 也能看到其中有tf_esp项，有可能是fork新进程等需要调用系统中断的场合下使用

## Ex2
1. 你需要完成在kern/process/proc.c中的do_fork函数中的处理过程。

	> * 按照提示完成代码即可
	> * 需要使用Ex1完成的alloc_proc函数
	> * 实现细节见代码

1. 请说明ucore是否做到给每个新fork的线程一个唯一的id？请说明你的分析和理由。

	> * 做到了。
	> * 新fork线程由get_pid赋予id。该函数作用是返回最小的没被使用的正数id号，可以保证任何时刻进程队列没有相同pid。

## Ex3
1. 在本实验的执行过程中，创建且运行了几个内核线程？

	> * 创建且运行了2个内核线程，分别是idleproc和initproc，见proc.c的proc_init

1. 语句local_intr_save(intr_flag);....local_intr_restore(intr_flag);在这里有何作用？请说明理由。
	
	> * 作用是禁用硬件中断
	> * 进程切换涉及特权级转换，寄存器、堆栈信息等也需备份恢复，不能被硬件中断破坏

## 与标准答案的差异
* Ex2中do_fork顺序不一样

## 本实验中重要的知识点
* 了解进程新建与复制需要什么结构信息
* 了解do_fork函数工作原理
* 了解切换进程的大致过程

## OS原理中很重要但在实验中没有对应上的知识点
* 用户进程切换与内核线程之间的区别与联系
* 具体函数的解析，如kernel_thread

