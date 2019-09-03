---
layout: post
title: "斐波那契数列"
date: "2019-06-06"
description: "斐波那契数列"
tag: [algorithm]
---

### 斐波那契数列

#### 题目一
求斐波那契数列的第 n 项  
写一个函数, 输入 n, 求斐波那契数列的第 n 项; 斐波那契的定义如下:
$$
f(n) =
\begin{cases}
0, & \text{n = 0} \\
1, & \text{n = 1} \\
f(n-1) + f(n-2), & \text{n > 1}
\end{cases}
$$

##### 思路
斐波那契数列方程有平凡情况(递归基) 和一般情况(递推方程), 所以可以使用递归实现, 实现也很简单

##### 实现
```
public class Fibonacci {
    public static void main(String[] args) {
        System.out.println(fibonacci(10));
    }

   private static long fibonacci(int n) {
        if (n <= 0) {
            return 0;
        }
        if (n == 1) {
            return 1;
        }
        return fibonacci(n -1) + fibonacci(n - 2);
   }
}
```
但通常递归的解法会有很严重的效率问题; 可以分析以上 `f(10)` 递归方法计算的过程, `f(10)` 需要计算出 `f(9)` 和 `f(8)`, `f(9)` 又需要计算出 `f(8)` 和 `f(7)` 等等, 可知以上过程有许多重复的计算, 当计算的 n 越大重复计算就越多, 效率很低 (可尝试 n = 100 的耗时)

##### 优化一
为了优化计算, 首先想到的是避免重复计算的部分; 为了实现避免重复计算, 可以为计算结果做缓存, 在计算时先查询缓存, 未命中则计算出结果然后缓存; 更直接的办法时自底向上计算, 这样还可以省去用于缓存的辅助空间; 实现代码如下
```
public class Fibonacci {
    public static void main(String[] args) {
        System.out.println(fibonacci(10));
    }

   private static long fibonacci(int n) {
       if (n <= 0) {
           return 0;
       }
       if (n == 1) {
           return 1;
       }

       long subItemTwo = 0;
       long subItemOne = 1;
       long result = 0;
        for (int i = 2; i <= n; i++) {
            result = subItemOne + subItemTwo;
            subItemTwo = subItemOne;
            subItemOne = result;
        }
        return result;
   }
}
```
此算法时间复杂度为 $O(n)$

##### 优化二
时间复杂度为 $O(logn)$ 但不够使用的解法
TODO

#### 题目二
青蛙跳台阶问题:  
一只青蛙一次可以跳上 1 级台阶, 也可以跳上 2 级台阶; 求青蛙跳上一个 n 级台阶总共有多少种跳法

##### 思路
分析平凡情况和一般情况的跳法
- 有 1 级台阶, 那显然只有一种跳法
- 有 2 级台阶, 就有两种跳法: 每次跳 1 级跳两次, 一次跳 2 级跳一次
- 有 n 级台阶, (将其看为 n 的函数, 记为 f(n)), 第一次跳有两种选择: 一是第一次跳一级, 后续台阶的跳法则为 n-1 级台阶的跳法数目 (记为 f(n-1)); 二是第一次跳 2 级, 此时跳法数目等于后面剩下的 n-2 级台阶的跳法数目(记为 f(n-2))

不难看出这实际上就是斐波那契数列了

#### 扩展题
TODO

#### 相关题
TODO
