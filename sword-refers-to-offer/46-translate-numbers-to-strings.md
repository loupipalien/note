---
layout: post
title: "把数字翻译成字符串"
date: "2019-07-11"
description: "把数字翻译成字符串"
tag: [algorithm]
---

### 把数字翻译成字符串

#### 题目
给定一个数字, 我们按照如下规则把它翻译为字符串: 0 翻译成 "a", 1 翻译成 "b", ..., 11 翻译成 "1", ..., 25 翻译成 "z"; 一个数字可能有多个翻译; 例如, 12258 有 5 种不同的翻译, 分别是 "bccfi", "bwfi", "bczi", "mcfi", "mzi"; 请编程实现一个函数, 用来计算一个数字有多少中不同的翻译方法

##### 思路
当最开始的一个或两个数字被翻译成一个字符之后, 接着翻译后面剩下的数字, 这显然可以写成一个递归函数来计算翻译的数目; 定义函数 f(i) 表示从第 i 位数字开始的不同翻译的数目, 那么 f(i) = f(i+1) + g(i, i+1) * f(i+2), 即当第 i 位和第 i + 1 为两位数字拼接起来的数字在 10 - 25 的范围内, 函数 g(i, i+1) 的值为 1, 否则为 0  
但此问题递归是会有重复的子问题, 为了避免重复的子问题, 自下而上解决问题, 这样就可以消除重复子问题

##### 实现
```Java
public int TranslateNumbersToStrings(int number) {
    if (number < 0) return 0;

    char[] chars = String.valueOf(number).toCharArray();
    int[] dp = new int[chars.length + 1];
    dp[0] = 1;

    for (int i = 1; i <= chars.length; i++) {
        // 当前字符单独翻译时
        if (chars[i - 1] >= '0' && chars[i - 1] <= '9') {
            dp[i] = dp[i - 1];
        }
        // 当前字符和前一个字符组合翻译时
        if (i > 1) {
            int temp = (chars[i - 2] - '0') * 10  + chars[i - 1] - '0';
            if (temp >= 10 && temp <= 25) {
                dp[i] = dp[i - 1] + dp[i - 2];
            }
        }
    }

```
