---
layout: post
title: "1 - n 整数中 1 出现的次数"
date: "2019-07-08"
description: "1 - n 整数中 1 出现的次数"
tag: [algorithm]
---

### 1 - n 整数中 1 出现的次数

#### 题目
输入一个整数 n, 求 1 - n 这 n 个整数的十进制表示中 1 出现的次数; 例如, 输入 12, 1 - 12 这些整数中包含 1 的数字有 1, 10, 11 和 12, 1 一共出现了 5 次

##### 思路
例如: 数字 21345
- 先计算在首位为 1 的个数, 分为首位为 1 和大于 1 的情况
- 再计算非首位为 1 的个数, 即非首位中任意一位为 1, 其余位可以是 0 ~ 9 的任意数字
- 最后计算去除首位的数字中出现 1 的个数
- 当 n 为个数时, 等于 0 时返回 0, 非 0 时返回 1

##### 实现
```
public class Solution {
    public static void main(String[] args) {
        System.out.println(numberOfOne(12));
    }

    private static int numberOfOne(int n) {
        if (n <= 0) {
            return 0;
        }

        String nStr = String.valueOf(n);
        int first = nStr.charAt(0) - '0';

        if (nStr.length() == 1 && first == 0) {
            return 0;
        }
        if (nStr.length() == 1 && first != 0) {
            return 1;
        }
        // 第一位为 1 的个数
        int numberOfOneAtFirstDigit = 0;
        if (first > 1) {
            numberOfOneAtFirstDigit = (int) Math.pow(10, nStr.length() - 1);
        } else {
            numberOfOneAtFirstDigit = Integer.valueOf(nStr.substring(1)) + 1;
        }
        // 其他位为 1 的个数
        int numberOfOnAtOtherDigits = first * (nStr.length() - 1) * (int) Math.pow(10, nStr.length() - 2);
        // 去除第一位时数字中 1 的个数
        int numberOfOneAtRecursive = numberOfOne(Integer.valueOf(nStr.substring(1)));
        return numberOfOneAtFirstDigit + numberOfOnAtOtherDigits + numberOfOneAtRecursive;
    }
}
```
