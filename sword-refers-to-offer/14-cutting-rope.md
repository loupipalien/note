---
layout: post
title: "剪绳子"
date: "2019-06-10"
description: "剪绳子"
tag: [algorithm]
---

### 剪绳子

#### 题目一
给你一根长度为 `n` 的绳子, 请把绳子剪成 `m` 段 (m, n 都是整数, n > 1 并且 m > 1), 每段绳子的长度记为 `k[0], k[1], ... , k[m]`; 请问 `k[0] * k[1] * ... * k[m]` 可能的最大乘积是多少? 例如, 当绳子的长度为 8 时, 我们把它剪成长度分别为 `2, 3, 3` 的三段, 此时得到的最大乘积是 18

##### 思路一
TODO

##### 实现一
```Java
public int cuttingRope(int n) {
    if (n < 1) return 0;
    if (n < 2) return 0;
    if (n < 4) return n - 1;

    int a = n / 3, b = n % 3;
    if (b == 0) return (int) Math.pow(3, a);
    if (b == 1) return (int) (Math.pow(3, a - 1)) * 4;
    return (int) (Math.pow(3, a) * 2);
}
```

##### 思路二
TODO

##### 实现二
```Java
if (n < 1) return 0;
public int integerBreak(int n) {
    if (n < 1) return 0;
    if (n < 2) return 1;
    if (n < 4) return n - 1;

    int[] dp = new int[n + 1]; dp[2] = 1;
    for (int i = 3; i <= n; i++) {
        for (int j = 1; j < i; j++) {
            /**
             * dp[i]: 维持原状态不剪
             * j * (i -j): 从 j 处剪一下, 剪下来的部分是 i - j, i - j 不再剪了
             * j * dp[i - j]: 从 j 处剪一下, 剪下来的部分是 i - j, i - j 继续剪
             */
            dp[i] = Math.max(dp[i], Math.max(j * (i - j), j * dp[i - j]));
        }
    }
    return dp[n];
}
```

>**参考:**
- [面试题14- I. 剪绳子（贪心思想，清晰图解）](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/solution/mian-shi-ti-14-i-jian-sheng-zi-tan-xin-si-xiang-by/)
- [图解【暴力递归】【记忆化技术】【动态规划】【动态规划优化解法】【找规律】](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/solution/xiang-jie-bao-li-di-gui-ji-yi-hua-ji-zhu-dong-tai-/)
