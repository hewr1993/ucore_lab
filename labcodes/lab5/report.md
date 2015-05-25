#Lab5 Report
-----
## Ex0
1. LAB1更新

	> * 每个时间片更新时，将当前进程的need_resched置1
	> * 注意要把LAB1中输出时钟信息的语句注释掉，不然自旋锁的检测将因额外输出失败

1. LAB4更新
	
	> * 按照提示增加结构信息即可，思路与LAB4一致

## Ex1
1. 加载应用程序并执行

	> * 按照提示完成代码即可
	> * 实现细节见代码
	> * 最后一行不知如何填写，参考了lab_result

1. 描述当创建一个用户态进程并加载了应用程序后，CPU是如何让这个应用程序最终在用户态执行起来的。即这个用户态进程被ucore选择占用CPU执行（RUNNING态）到具体执行应用程序第一条指令的整个经过。

	> * init_main调用kernel_thread，再do_fork复制一个内核线程user_main
	> * user_main调用kernel_execve产生SYS_exec系统调用
	> * 系统调用经由trap，调用sys_exec，并调用do_execve
	> * do_execve释放页表、内存空间，调用load_icode加载ELF格式程序
	> * load_icode读取ELF文件、建立页表及堆栈，构造trap_frame的用户态
	> * 内核线程对应的用户进程被唤醒吼，用户进程运行，context的eip指向forkret，调用trapentry.S中的__trapret，并调用iret进入用户态
	> * 此时trap_frame的eip对应程序第一条指令

## Ex2
1. 补充copy_range的实现，确保能够正确执行。

	> * 按照提示完成代码即可
	> * 实现细节见代码
	> * 需要配合Ex0完成

1. 简要说明如何设计实现“Copy on Write 机制”

	> * 拷贝进程时，页表所有页标记为只读，父子进程共享一个页表映射
	> * 当写只读页时，修改进程对应页的映射，各进程对应页权限改为可读写

## Ex3
1. 分析fork/exec/wait/exit在实现中是如何影响进程的执行状态的。

	> * do_fork创建新进程，并设置状态为RUNNABLE
	> * do_execve调用load_icode加载用户程序，进程等待状态
	> * do_wait状态设为SLEEPING，等待子进程结束
	> * do_exit释放资源，状态设为ZOMBIE，等待系统回收

1. 请给出ucore中一个用户态进程的执行状态生命周期图（包执行状态，执行状态之间的变换关系，以及产生变换的事件或函数调用）。
	
	```
	UNINIT -> do_fork -> RUNNABLE <- schedule -> RUNNING -> do_exit -> ZOMBIE
	                       |                        |
	               do_wait / wakeup_proc    do_wait / wakeup_proc
	                       |                        |
	                       +------- SLEEPING -------+
	```

## 与标准答案的差异
* 基本一致

## 本实验中重要的知识点
* do_execve与load_icode的具体流程
* 如何利用trap_frame跳转到用户态
* 如何在用户态运行用户程序

## OS原理中很重要但在实验中没有对应上的知识点
* 如何选择加载指定用户程序

