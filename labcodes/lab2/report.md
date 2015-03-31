#Lab2 Report
-----
## Ex2
1. 实现first-fit连续物理内存分配算法。

	> * 阅读memlayout.h了解Page结构体的各类成员变量及使用方法
	> * default_init_memap
		* 初始化从base开始的n页
		* 第一页特殊处理，并最后并入free_list
	> * default_alloc_pages
		* 找第一个满足大小的块分配空间
		* free_list从头开始遍历，第一个大小>n的块即为所求
		* 若最后仍有剩余空间，新建块并维护free_list
	> * default_free_pages
		* 从base开始释放n页
		* 先遍历n页释放空间
		* 再寻找free_list中应插入的位置
		* 插入free_list，并在可以的时候合并前驱后继
	> * 具体实现见代码

1. 你的first fit算法是否有进一步的改进空间？

	> * 使用平衡树维护free_list
	> * 则查询、插入、删除均可做到O(logn)的复杂度
