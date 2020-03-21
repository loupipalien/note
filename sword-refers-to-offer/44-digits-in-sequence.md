---
layout: post
title: "数字序列中某一位的数字"
date: "2019-07-09"
description: "数字序列中某一位的数字"
tag: [algorithm]
---

### 数字序列中某一位的数字

#### 题目
数字以 `0123456789101112131415...` 的格式序列化到一个字符序列中; 在这个序列中, 第 5 位 (从 0 开始计数) 是 5, 第 13 位是 1, 第 19 位是 4, 等等; 请写一个函数, 求任意第 n 位对应的数字

##### 思路
0 - 9 这 10 个数字各占一位, 10 - 99 这 90 这数字各占 2 位, 100 - 999 这 900 个数字个占 3 位, 以此类推; 要获取第 n 位对应的数字, 先定位到某个数字区间, 再算出为此数字区间的第多少位数, 即可推算出为几

##### 实现
```Java
public class DigitsInSequence {
    public static void main(String[] args) {
        System.out.println(digitsInSequence(1001));
    }

    private static int digitsInSequence(int index) {
        if (index < 0) return -1;
        if (index == 0) return 0;
        // 当前数字区间的位数和个数
        double digit = 1, count = 9;
        while (index > digit * count) {
            index -= digit * count;
            digit += 1;
            count *= 10;
        }
        // 计算出第 n 位所在的数字
        int number = (int) (Math.pow(10, digit - 1) - 1 + Math.ceil(index / digit));
        // 下标从 0 开始计算
        int indexFromRight = (int) ((digit - index % digit) % digit);
        while (indexFromRight-- > 0) number /= 10;
        return number % 10;
    }
}
```
