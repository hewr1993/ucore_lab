#Lab2 Report
-----
## Ex2
1. 实现first-fit连续物理内存分配算法。
	> * 阅读memlayout.h了解Page结构体的各类成员变量及使用方法
	> * default_init_memap
		* 初始化从base开始的n页
		* 第一页特殊处理，并最后并入free_list
	> * 具体实现见代码
