---
layout: post
title: "数值的整数次方"
date: "2019-06-11"
description: "数值的整数次方"
tag: [algorithm]
---

### 数值的整数次方

#### 题目一
实现函数 double Power(double base, int exponent), 求 base 的 exponent 次方; 不得使用库函数, 同时不需要考虑大数问题

##### 思路
- 0 的任何次幂都为 1
- 任何数的 0 次幂都为 0
- 任何数的 1 次幂为数本身
- 非零数的负数次幂为对应正数次幂的倒数

##### 实现
```
public class Power {
    public static void main(String[] args) {
        System.out.println(power(10, 5));
        System.out.println(power(10, -5));
        System.out.println(power(0.10, 5));
        System.out.println(power(0.10, -5));
        System.out.println(power(0, 0));
        System.out.println(power(0, 5));
        System.out.println(power(0, -5));
    }

    private static double power(double base, int exponent) {
        if (base == 0) {
            return 1;
        }
        if (exponent == 0) {
            return 1;
        }
        if (exponent == 1) {
            return base;
        }

        double result = 1;
        int absExponent = exponent > 0 ? exponent : -exponent;
        for (int i = 1; i < absExponent; i++) {
            result *= base;
        }
        return exponent > 0 ? result : 1 / result;
    }
}
```
此算法的时间复杂度为 $O(n)$, 其中 n 为 exponent

#### 优化
如果输入的指数 exponent 为 32, 那么在以上算法中就需要做 31 次乘法; 换一种思路, 考虑求一个数的 32 次方, 如果已知这个数的 16 次方, 那么只需要再平方一次即可; 而 16 次方是 8 次方的平方, 这样类推, 32 次方只需要做 5 次乘法即可; 见以下公式
$$
a^n =
\begin{cases}
a^{n/2} * a^{n/2}, & \text{n 为偶数} \\
a^{(n-1)/2} * a^{(n-1)/2} * a, & \text{n 为奇数}
\end{cases}
$$
使用以上公式实现如下
```
public class Power {
    public static void main(String[] args) {
        System.out.println(power(0.001d, 3));
        System.out.println(power(0.001f, 3));
    }

    private static double power(double base, int exponent) {
        if (base == 0) {
            return 0;
        }
        if (exponent == 0) {
            return 1;
        }
        if (exponent == 1) {
            return base;
        }

        int absExponent = exponent > 0 ? exponent : -exponent;
        double result = power(base, absExponent >> 1);
        result *= result;
        if ((exponent & 0x1) == 1) {
            result *= base;
        }
        return exponent > 0 ? result : 1 / result;
    }
}
```
此算法的复杂度为 $O(logn)$, 其中 n 为 exponent
