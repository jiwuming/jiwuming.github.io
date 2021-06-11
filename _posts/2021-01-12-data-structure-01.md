---
title: 从零开始学习数据结构01
tags: [数据结构]
date: 2021-01-12
---
# 什么是数据结构
计算机存储数据的方式

# 抽象数据类型
集合结构
线性结构(数组, 链表, 栈, 队列)
树形结构
图形结构

# big O
大O表示法各种数据结构的时间复杂度[https://www.bigocheatsheet.com/](https://www.bigocheatsheet.com/)
最坏情况下（只关心n为无穷大时）复杂度的上限，复杂情况（越往下越复杂）
- O(1)
- O(log2n) 比如log2x的意思就是求x是2的多少次幂.
- O(n)
- O(nlog2n)
- O(n²)
- O(n³)
- O(2的b次幂，b>1)
- O(n!)

由于只关心n最大情况，所以以一个函数最大的那一部分来确定big O，比如
```go
O(n + c) = O(n)
O(cn) = O(n), c > 0
// 复杂函数：
f(n) = 7log(n)³ + 15n² + 2n³ + 8 => O(f(n)) = O(n³)
```

简化累加时间复杂度
```js
function sum(n) {
    var k = 0
    for(var i = 0; i <= n; i++) {
        k = k + i
    }
    console.log(k)
}
console.log("时间复杂度O(n)求和")
sum(10)

function sum2(n) {
    // 本来有一个最大数，前后相加凑成最大数，在用最大数乘以这几组求和，中间数剩余的相当于（0.5 * 最大数）。
    console.log(n * (n + 1) / 2)
}
console.log("时间复杂度O(1)求和")
sum2(10)
```

看一个算法的时间复杂度主要是看循环次数（加减改变循环次数的忽略不计，乘除改变循环次数的是log乘数/除数n）。
```c
// O(n)
void func(int n) {
    int i = 1; k = 100;
    while(i < n) {
        k++;
        i+=2
    }
}

// 0(log2n)
void func(int n) {
    int i = 1;
    while(i <= n) {
        i = i * 2;
    }
}
```
# 线性表


# 静态数组和动态数组
数组总要特性：内存连续

静态数组使用场景
1. 存储有序数据
2. 存储临时对象
3. IO
4. 查找表和反查找表
5. 从一个函数中返回多个值
6. 缓存

| 数组类型/操作 | static array | dynamic array |
|----|----|----|
|  Access  |  O(1)  |  O(1)  |
|  Search  |  O(n)  |  O(n)  |
|  Insertion  |  N/A  |  O(n)  |
|  Appending  |  N/A  |  O(1)  |
|  Deletion  |  N/A  |  O(n)  |

# 链表
(火车组)

(单链表)每一个对象都会包含对象本身及后面一个对象的首地址, 最后一个对象携带的后面地址为null

# 栈
后进先出(货架上的货品)

限定仅在表尾进行插入或删除操作的线性表。也就是说它有两个操作,且操作数都在线性表尾部

# 队列
先进先出(排队办业务)

是一种特殊的线性表，它只允许在表的前端（front）进行删除操作，而在表的后端（rear）进行插入操作。