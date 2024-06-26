## 数据结构
```
数组、链表、队列、栈、堆
```


### 链表
```
表是一种常见的数据结构，它由一系列节点组成，每个节点包含两部分：数据和指向下一个节点的指针。链表中的节点可以在内存中不连续地存储，而通过指针来连接彼此，形成一个链式结构。
```
* 单链表每个节点只有一个指针指向下一个节点
* 双向链表中，每个节点有两个指针，一个指向前一个节点，另一个指向后一个节点。
```
Redis 中，List 使用双向链表来存储数据.这种结构使得在 List 的头部和尾部执行插入、删除等操作时的时间复杂度都是 O(1)，因为只需要修改节点的指针即可，而不需要像数组那样需要移动其他元素
```

### 数组与链表的区别
#### 1、存储方式
* 数组：在内存中连续存储元素，每个元素占据固定大小的内存空间。
* 链表：元素在内存中可以不连续存储，每个元素通常包含两部分：数据和指向下一个元素的指针。
#### 2、访问效率：
* 数组：可以通过索引直接访问任何元素，因此查找速度较快，时间复杂度为 O(1)。
* 链表：需要从头节点开始逐个遍历直到找到目标元素，因此查找速度相对较慢，时间复杂度为 O(n)。
#### 3、插入和删除操作：
* 数组：插入和删除操作可能涉及元素的移动，特别是在中间或开头插入/删除元素时。如果需要调整大小，可能需要重新分配内存和复制元素，时间复杂度为 O(n)。
* 链表：插入和删除操作只需更新相邻节点的指针，不需要移动其他元素，因此在链表中进行这些操作效率较高，时间复杂度为 O(1)。
#### 4、空间复杂度：
* 数组：因为是连续存储，需要一块连续的内存空间，如果预留了大量空间但未完全使用，会浪费一部分内存。
* 链表：节点之间可以分散存储，不需要连续的内存空间，因此可以更灵活地利用内存。
* 综上，数组适合随机访问和频繁对元素进行查找的场景，而链表适合频繁进行插入和删除操作的场景。

### 树
#### 二叉树
* 1、二叉树：每个节点最多有两个子节点的树
```
      1
     / \
    2   3
   / \
  4   5
```
* 2、二叉搜索树（BST）：一种特殊的二叉树，左子树上的节点值都小于根节点的值，右子树上的节点值都大于根节点的值。
```
      4
     / \
    2   6
   / \ / \
  1  3 5  7
```
* 3、平衡二叉树：一种特殊的二叉搜索树，保持左右子树的高度差不超过1，以确保高效的插入、删除和查找操作。
```
       4
     /    \
    2      6
   / \    / \
  1   3  5   7
```
* 4、红黑树：一种自平衡的二叉搜索树，通过在每个节点上增加存储位来确保树在任何给定时间都保持平衡。
```
        4
       / \
      2   6
     / \ / \
    1  3 5  7
```
* 5、AVL树：一种自平衡的二叉搜索树，确保树的任何节点的两个子树的高度差不超过1。


#### 多叉树
* B树和B+树
```
一种多叉树，用于文件系统和数据库中的索引结构，具有高度平衡和高效的查找特性。
```
#### 字典树
```
一种用于存储关联数组的树形数据结构，通常用于字符串检索和前缀匹配。
```
