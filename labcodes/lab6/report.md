#Lab6 Report
-----
## Ex0
1. LAB5更新
	
	> * proc.c的alloc_proc，加入若干字段的初始化，与之前类似
	> * trap.c的trap_dispatch，加入sched_class_proc_tick函数
		* 更新后代码编译错误，只能把sched.c中相应函数的static修饰符去掉

## Ex1
1. 请理解并分析sched_class中各个函数指针的用法，并接合Round Robin调度算法描ucore的调度执行过程。

	> * sched_class各函数指针
		* init：初始化队列
		* enqueue：压入新进程
		* dequeue：移除特定进程
		* pick_next：调度算法挑选下一个进程
		* proc_tick：时钟中断，调度算法所需
	> * ucore调度过程
		* 初始化进程队列
		* 压入初始活动进程
		* 等待时间片结束，调度算法选出下一活动进程
		* 转换运行进程
		* 移除退出的进程

1. 在实验报告中简要说明如何设计实现”多级反馈队列调度算法“，给出概要设计。

	> * 建立多个MAX_TIME_SLICE不同的队列
	> * 队列内按次序调度，队列间使用各种其他调度算法
	> * 定位具体进程及其队列可使用hash

## Ex2
1. 实现Stride Scheduling调度算法。

	> * 按提示完成init、enqueue、dequeue、pick_next以及proc_tick
	> * 前三项按照提示完成即可，比较简单
	> * 调度下一进程，按照Stride Scheduling原理，选择pass最小的进行调度
		* 切换进程前，维护该进程的pass，即加上当前stride（MAX_STRIDE/tickets）
	> * proc_tick响应时钟事件，维护当前time_slice，时间片耗尽时需要reschedule释放CPU

## 与标准答案的差异
* 思路一致，细节有出入

## 本实验中重要的知识点
* stride scheduling算法具体流程及原理
* 调试C++代码能力，比如要对新版代码的编译失败进行debug

## OS原理中很重要但在实验中没有对应上的知识点
* Lottery Scheduling没有涉及
* 当前调度框架的熟悉了解

