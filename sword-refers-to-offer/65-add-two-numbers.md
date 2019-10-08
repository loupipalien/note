---
layout: post
title: "不用加减乘除做加法"
date: "2019-07-30"
description: "不用加减乘除做加法"
tag: [algorithm]
---

### 不用加减乘除做加法

#### 题目
写一个函数, 求两个整数之和, 要求在函数体内不得使用 `+, -, *, /` 四则运算符号

##### 思路
不能使用四则运算, 则只能使用二进制的位运算求解  
考虑 5 + 17 = 22 的十进制运算过程, 可以分为以下三步
- 只做各位相加不进位
- 计算进位, 5 + 7 中有进位值 10
- 把前面两个结果加起来, 12 + 10 = 22

考虑 5 (101) + 17 (10001) = 22 (10110) 的二进制运算过程, 可以分为三步
- 只做各位相加不进位, 即两个数异或的结果, 10100
- 计算进位, 即两数相与后左移一位, 10
- 把前面两个结果加起来, 即重复以上过程, 直到进位为 0

##### 实现
```
public class Solution {
    public static void main(String[] args) {
        System.out.println(add(1, 4));
        System.out.println(add(1, -4));
        System.out.println(add(-1, 4));
    }

    private static int add(int one, int another) {
        int sum, carry;
        do {
            sum = one ^ another;
            carry = (one & another) << 1;
            one = sum;
            another = carry;
        } while (carry != 0);
        return sum;
    }
}
```

#### 相关问题
不使用新变量, 交换两个变量的值; 比如有两个变量 a, b, 我们希望交换它们的值; 有两种不同的方法, 分别是基于加减法和基于异或运算  
TODO
