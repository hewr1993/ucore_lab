#Lab7 Report
-----
## Ex0
1. LAB6更新
	
	> * trap.c的trap_dispatch，引入run_timer_list函数

## Ex1
1. 在实验报告中给出内核级信号量的设计描述，并说其大致执行流流程

	> * 信号量结构体
	```
	typedef struct {
		int value;
		wait_queue_t wait_queue;
	} semaphore_t;
	```
		* 其中value表示资源量
		* wait_queue为未申请到资源的进程队列
	> * 信号量P操作见sem.c的__down
		* 每次P操作，检查value是否为0
		* 若value>0，表明有资源，返回申请成功
		* 否则把进程加入等待队列，并等待调度
	> * V操作见sem.c的__up
		* 每次V操作，检查等待队列
		* 若等待队列为空，则value加一
		* 否则，取出队头，并唤对应进程

1. 在实验报告中给出给用户态进程/线程提供信号量机制的设计方案，并比较说明给内核级提供信号量机制的异同

	> * 可以借鉴内核态的信号量机制
	> * 用户态的信号量托管在内核态中，但对用户不可见，即用户使用过程体验与在用户态中一样
	> * 需要使用信号量时，通过中断从用户态跳转到内核态，维护信号量，再反馈到用户态

## Ex2
1. 完成内核级条件变量和基于内核级条件变量的哲学家就餐问题

	> * moniter.c的cond_signal
		* 按照提示填写即可，下同
		* 父monitor可通过cvp->owner获得
		* 先up自身，再down下一进程即可
	> * moniter.c的cond_wait
		* 根据等待数量next_count确定up哪个信号量
		* 再down自身即可
	> * check_sync.c的phi_take_forks_condvar
		* 先设自己为hungry
		* 再调用phi_test_condvar来确定自己是否可以变为eating
		* 如果还不能就餐，则cond_wait进入等待
	> * check_sync.c的phi_put_forks_condvar
		* 先设自己吃完了，为thinking
		* 这时候左右刀叉可以释放，调用phi_test_condvar处理左右邻居即可

## 与标准答案的差异
* 哲学家就餐部分，思路一致，具体实现不一样

## 本实验中重要的知识点
* 信号量的具体数据结构
* 同步互斥架构
* 哲学家问题的具体解决细节

## OS原理中很重要但在实验中没有对应上的知识点
* 管程具体实现

