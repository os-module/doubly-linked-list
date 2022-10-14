# doubly-linked-list

使用rust实现linux里面的list\_head数据结构，只实现所需要的功能，并不是完全体。

## 使用

目前用在slab模块中，由于slab不能使用rust内与堆分配相关的数据结构以及智能指针，因此底层只能以裸指针的形式构造双端链表。

