---
typora-copy-images-to: ..\images
---

# 链表

## 链表的优点

1. 高效的节点重排能力
2. 顺序性的节点访问方式
3. 灵活的调整链表长度

## Redis的链表实现

​	Redis使用的C语言并没有内置链表这种数据结构，因此Redis构建了自己的链表实现

```go
// 链表节点的实现
typedef struct listNode{
	struct listNode *prev;	// 前置节点
	struct listNode *next;	// 后置节点
	void *value; 	// 当前节点值
}listNode

// 链表的实现
typedef struct list{
	listNode *head;	// 表头节点
	listNode *tail;	// 表尾节点
	unsigned long len;	// 链表包含的节点数量
    void *(*dup) (void *prt);	// 节点值复制函数，复制链表节点保存的值
    void (*free) (void *prt);	// 节点值释放函数，释放链表节点保存的值
    void *(*match) (void *prt, void *key);	// 节点值对比函数，对比链表节点保存的值是否和另一个输入值相等
}list
```

​	多个链表节点能通过指针`prev`和`next`组成双端链表，而链表结构提供了表头、表尾节点的指针以及链表节点的数量，其中的三个成员函数用于实现多态链表所需要的特定类型函数

<img src="..\images\redis链表.jpg" alt="redis链表" style="zoom:50%;" />    	

## Redis链表的特性	

1. 双端：通过`prev`和`next`可以获取当前节点的上一个节点和下一个节点，复杂度为O(1)
2. 无环：表头的`prev`指针和表尾的`next`指针都指向null，对链表的访问以null为终点
3. 链表带表头和表尾：链表维护表头和表尾节点的指针，因此获取表头和表尾复杂度为O(1)
4. 链表长度计数器：获取链表长度复杂度为O(1)
5. 多态：节点用指针保存节点值，并且链表的`dup`、`free`、`match`为节点值设置类型特定函数，所以链表可以保存各种不同类型的值



