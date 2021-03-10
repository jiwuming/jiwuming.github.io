---
title: 从零开始学习数据结构01
tags: [数据结构]
date: 2021-01-12
---
# 什么是数据结构

# 抽象数据类型

# big O
最坏情况下（只关心n为无穷大时）复杂度的上限，复杂情况（越往下越复杂）
- O(1)
- O(log(n))
- O(n)
- O(nlog(n))
- O(n²)
- O(n³)
- O(n的b次幂，b>1)
- O(n!)

由于只关心n最大情况，所以以一个函数最大的那一部分来确定big O，比如
```go
O(n + c) = O(n)
O(cn) = O(n), c > 0
// 复杂函数：
f(n) = 7log(n)³ + 15n² + 2n³ + 8 => O(f(n)) = O(n³)
```

## O(1)
```go
a := 1
b := 2
c := a + 5 * b

i := 0
While i < n Do
    i = i + 1
f(n) = n
O(f(n)) = O(n)

i := 0
While i < n Do
    i = i + 3
f(n) = n / 3
O(f(n)) = O(n)
```

## O(n²)
```go
For(i := 0; i < n; i++) {
    For(j := 0; j < n; j++) {
        ...
    }
}
// 双循环，外圈n从第一个数字开始起内圈就要执行n次，那n个数字所要执行的次数就是n * n也就是n方
f(n) = n * n = n²，O(f(n)) = O(n²)

For(i := 0; i < n; i++) {
    For(j := i; j < n; j++) {
        ...
    }
}
// 首先，内圈执行次数
n + (n - 1) + (n - 2) + (n - 3) ... 3 + 2 + 1
// 有n个上述次数，可以以一个数字带入比如10，也就是10 + 9 + 8 ... + 2 + 1 + 0
n * (n + 1) / 2
// 简化之后
O(n * (n + 1) / 2) => O(n² / 2 + n / 2) = O(n²) 

i := 0
While i < n Do
    j = 0
    While j < 3 * n Do
        j = j + 1
    j = 0
    While j < 2 * n Do
        j = j + 1
    i = i + 1

// 最外层执行了n遍，第一层j执行了3n遍，第二层j执行了2n遍
f(n) = n * (3n + 2n) = 5n²
O(f(n)) = O(n²)
```

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
