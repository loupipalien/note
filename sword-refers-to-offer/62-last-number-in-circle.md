---
layout: post
title: "圆圈中最后剩下的数字"
date: "2019-07-27"
description: "圆圈中最后剩下的数字"
tag: [algorithm]
---

### 圆圈中最后剩下的数字

#### 题目
`0, 1, ..., n - 1` 这 n 个数字排成一个圆圈, 从数字 0 开始, 每次从这个圆圈中删除第 m 个数字; 求出这个圆圈里剩下的最后一个数字


##### 思路
创建 n 个节点的环形链表, 模拟此过程, 直到链表中只剩一个节点

##### 实现
```Java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class Solution {
    public static void main(String[] args) {
        System.out.println(lastNumberInCircle(5, 3));
    }

    private static int lastNumberInCircle(int n, int m) {
        if (n < 1 || m < 1) return -1;
        if (m == 1) return n - 1;

        // 这里使用列表模拟环形链表, 当到列表尾的下一个时则重置到列表头
        List<Integer> circle = new ArrayList<>();
        for (int i = 0; i < n; i++) circle.add(i);

        int current = 1;
        Iterator iterator = circle.iterator();
        while (circle.size() > 1) {
            while (iterator.hasNext()) {
                iterator.next();
                if (current == m) {
                    iterator.remove();
                    current = 1;
                } else {
                    current++;
                }
            }
            iterator = circle.iterator();
        }
        return circle.get(0);
    }
}
```

##### 优化
首先定义一个关于 n 和 m 的方程 `f(n, m)`, 表示每次在 n 个数字 `0, 1, ..., n - 1` 中删除第 m 个数字最后剩下的数字  
在这 n 个数字中, 第一个被删除的数字是 `(m - 1) % n`, 为了简单起见, 我们把 `(m - 1) % n` 记为 k, 那么删除 k 之后剩下的 `n - 1` 个数字为 `0, 1, ..., k - 1, k + 1, ..., n - 1`, 并且下次删除从数字 `k + 1` 开始计数; 相当于在剩下序列中, `k + 1` 排在最前面, 从而形成 `k + 1, ..., n - 1, 0, 1, ..., k - 1`; 该序列最后剩下的数字也应该是关于 n 和 m 的函数; 由于这个序列的规律和前面最初的序列不一样 (最初的序列是从 0 开始的连续序列), 因此该函数不同于前面的函数, 记为 `f'(n - 1, m)`; 最初序列最后剩下的数字 `f(n, m)` 一定是删除一个数字之后的序列最后剩下的数字, 即 `f(n, m) = f'(n - 1, m)`  
接下来把剩下的这 `n - 1` 个数字的序列 `k + 1, ..., n - 1, 0, 1, ..., k - 1` 进行映射, 映射的结果形成一个 `0 ~ n - 2` 的序列
```
k + 1 => 0
k + 2 => 1
...
n - 1 => n - 2 - k
0 => n - 1 - k
1 => n - k
...
k - 1 => n -2
```
我们把映射定义为 p, 则 `p(x) = (x - k - 1) % n`, 它表示如果映射前的数字是 x, 那么映射后的数字是 `(x - k - 1) % n`; 该映射的逆映射是 $p^{-1}(x) = (x + k + 1) % n$  
由于映射之后的序列和最初的序列具有同样的形式, 即都是从 0 开始的连续序列, 因此仍然可以用函数 f 来表示, 记为 `f(n - 1, m)`; 根据映射规则, 映射之前的序列中最后剩下的数字 $ f'(n - 1, m) = p^{-1}[f(n - 1, m)] = [f(n - 1, m) + k + 1] % n$, 把 `k = (m - 1) % n` 带入得到 `f(n, m) = f'(n - 1, m) = [f(n - 1, m) + m] % n`  
由此得到的递归公式为
$$
f(n, m) =
\begin{cases}
0, & \text{n = 1} \\
(f(n - 1, m) + m) \% n, & \text{n > 1}
\end{cases}
$$
基于以上公式实现代码为
```
public class Solution {
    public static void main(String[] args) {
        System.out.println(lastNumberInCircle(5, 3));
    }

    private static int lastNumberInCircle(int n, int m) {
        if (n < 1 || m < 1) {
            throw new IllegalArgumentException("Invalid Parameter");
        }

        int last = 0;
        for (int i = 2; i <= n ; i++) {
            last = (last + m) % i;
        }
        return last;
    }
}
```
