# doubly-linked-list

使用rust实现linux里面的list\_head数据结构，只实现所需要的功能，并不是完全体。

## 数据结构

```rust
pub struct ListHead {
    pub prev: *mut ListHead,
    pub next: *mut ListHead,
}
```

## 公开接口

```rust
list_head! //声明一个ListHead并初始化
list_head_init! //初始化一个存在的listhead
list_add! //插入队列  head<-->3<-->2<-->1
list_add_tail! //尾端插入 head<-->1<-->2<-->3
is_list_empty! //队列是否空
list_del!      //从队列中删除
offset_of!     // 结构体字段偏移量
container_of!  // 从结构体字段推导其宿主结构体
to_list_head_ptr! //主要list_had字段转为裸指针
align_to!		// 向上对齐
```

## 使用

目前用在slab模块中，由于slab不能使用rust内与堆分配相关的数据结构以及智能指针，因此底层只能以裸指针的形式构造双端链表。此结构不能单独使用，一般作为某个字段放在一个自定义结构体中。

```rust
#[derive(Debug)]
struct Demo {
    list_head: ListHead,
    first: usize,
    second: usize,
}
list_head!(head);
let mut demo1 = Demo {
    list_head: ListHead::default(),
    first: 1,
    second: 2,
};
list_head_init!(demo1.list_head);
list_add!(to_list_head_ptr!(demo1.list_head), to_list_head_ptr!(head));
//list_add_tail!(to_list_head_ptr!(demo1.list_head), to_list_head_ptr!(head));
list_del!(to_list_head_ptr!(demo1.list_head));
```

计算字段偏移

```rust
let mut demo1 = Demo {
            list_head: ListHead::default(),
            first: 1,
            second: 2,
        };
let list_head_ptr = to_list_head_ptr!(demo1.list_head);
let list_head_offset = offset_of!(Demo, list_head);
let list_head_ptr2 = list_head_ptr as usize - list_head_offset;
let demo1_ptr = list_head_ptr2 as *mut Demo;
assert_eq!(demo1_ptr, &mut demo1 as *mut Demo);
```

获取宿主结构体

```rust
let mut demo1 = Demo {
            list_head: ListHead::default(),
            first: 1,
            second: 2,
        };
let list_head_ptr = to_list_head_ptr!(demo1.list_head);
let demo1_ptr = container_of!(list_head_ptr as usize, Demo, list_head);
assert_eq!(demo1_ptr, &mut demo1 as *mut Demo);
```

使用`iter`迭代

```rust
 list_head!(head); //链表头
list_head!(head2); //链表头
list_head!(head3); //链表头
list_add_tail!(to_list_head_ptr!(head2), to_list_head_ptr!(head));
list_add_tail!(to_list_head_ptr!(head3), to_list_head_ptr!(head));
let x = &head;
x.iter().for_each(|_list_head|{
    //println!("list_head:{:?}", list_head);
});
assert_eq!(x.next, to_list_head_ptr!(head2));
assert_eq!(x.len(), 2);
```



