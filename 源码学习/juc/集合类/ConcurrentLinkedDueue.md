# ConcurrentLinkedDueue

### 前言

`ConcurrentLinkedDueue`是一个无界的双端队列，底层是双向链表，它的并发操作是基于CAS的无锁实现，所以不会产生阻塞。

`ConcurrentLinkedDueue`的思想与概念与`ConcurrentLinekedQueue`很类似。

### 概述

- 一个node的item不为`null`，被认为是live node。因为一个node的`item`为null说明它已经是逻辑上被删除了，不过注意，初始化时队列中只有一个dummy node，它的`item`为null。
- 任何时刻，队列中只有一个first node，因为只有它的`prev`指针为null；只有一个last node，因为只有它的`next`指针为null。
- first node和last node可能不是live的（比如初始化时），它们之间可以通过`prev`和`next`链相互到达。
- 当一个新node被附在first node的`prev`上，或者附在last node的`next`上时，它就成功入队了。
- head或tail并不一定指向first node或last node。但通过head肯定能找到first node，tail同理。
- active node指处于队列上的节点，它们之间通过`prev`和`next`链能相互到达，当然active node的`item`可以是null。所以active node包括live node。
- 对节点的删除有三个步骤：
	- “logical deletion”。将node的item置为null，它在逻辑上已经被认为是删除了的。
	- “unlinking”。使得从active node出发不能到达这些逻辑删除节点（反之，从逻辑删除节点节点出发还是可以到达active node的），这样，GC最终会发现这些逻辑删除节点。
	- “gc-unlinking”。使得从逻辑删除节点也不能到达active node了，这样，会使得GC更快发现它们。当然，第三步只是一种优化，没有第三步节点也会被回收。这只是为了保持GC健壮性。
		- 在这一步中，会使得node self-links（即指针指向自己），另一个指针指向end终止符（end指first端或last端）。
- head和tail可能被unlinking，但不可以被gc-unlinking。
- 通过两种方法最小化volatile写的次数：
	- 允许head或tail偏离first node或last node。
	- 对同一块内存区域使用 volatile写 和 非volatile写 两种方式结合使用。比如`lazySetNext`和`casNext`结合使用、`UNSAFE.putObject(this, itemOffset, item)`和`casItem`的结合使用。
- 一般认为队首在左边，队尾在右边。所以往prev链方向前进称为左移，往next链方向前进称为右移。

