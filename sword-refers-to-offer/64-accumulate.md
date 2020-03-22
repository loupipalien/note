---
layout: post
title: "求 1 + 2 + ... + n"
date: "2019-07-29"
description: "求 1 + 2 + ... + n"
tag: [algorithm]
---

### 求 1 + 2 + ... + n

#### 题目
求 1 + 2 + ... + n, 要求不能使用乘除法, for, while, if, else, switch, case 等关键字以及条件判断句 (A ? B : C)

##### 思路
求 1 + 2 + ... + n 的和, 如果不使用公式, 就只能使用循环或者递归来完成, 但循环的关键字被限制, 递归需要判断的关键字也被限制; 所以需要考虑使用其他方式来完成循环或者递归
- 使用公式, 避开乘除法
- 使用递归调用代替循环, 使用短路代替判断
- TODO: 使用静态字段保存 sum, 实例化类 n 次

##### 实现
- 使用公式, 避开乘除法
```Java
public class Solution {
    public static void main(String[] args) {
        System.out.println(accumulate(10));
    }

    private static int accumulate(int n) {
        return (int)(Math.pow(n, 2) + n) >> 1;
    }
}
```
- 使用递归调用代替循环, 使用短路代替判断
```Java
public class Solution {
    public static void main(String[] args) {
        System.out.println(accumulate(10));
    }

    private static int accumulate(int n) {
        int sum = n;
        // 使用递归代替循环, 使用短路代替判断
        boolean flag = (n > 0) && (sum += accumulate(n - 1)) > 0;
        return sum;
    }
}
```
