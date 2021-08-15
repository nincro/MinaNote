## 由什么组成
state：包括counter，waiter以及semaphore
counter：目前尚未完成的个数
waiter：目前调用了wait的总数
sema：信号量的地址，runtime将所有sema组织为一个map，每一个bucket会组织为一个平衡树，然后去进行查找。每一个节点会搞出一个阻塞的
