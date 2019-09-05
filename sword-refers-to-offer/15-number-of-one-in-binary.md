---
layout: post
title: "二进制中 1 的个数"
date: "2019-06-10"
description: "二进制中 1 的个数"
tag: [algorithm]
---

### 二进制中 1 的个数

#### 题目一
请实现一个函数, 输入一个整数, 输出该数二进制表示中 1 的个数; 例如: 把 9 表示成二进制是 `1011`, 有 2 位是 1; 因此, 如果输入 9, 则该函数输出 2

##### 思路
先判断整数二进制表示中最右边一位是不是 1; 接着把输入的整数右移一位, 此时原来处于从右边数起的第二位被移到最右边, 在判断是不是 1; 如此循环直到整数为 0

##### 实现
```
public class NumberOfOneInBinary {
    public static void main(String[] args) {
        System.out.println(numberOfOneInBinary(0x80000000));
        System.out.println(numberOfOneInBinary(9));
    }

   private static int numberOfOneInBinary(int n) {
        int number = 0;
        while (n != 0) {
            if ((n & 1) == 1) {
                number++;
            }
            n = n >>> 1;
        }
        return number;
   }
}
```
如果不想改变原有数字的值, 可以使用 flag 和左移实现
```
public class NumberOfOneInBinary {
    public static void main(String[] args) {
        System.out.println(numberOfOneInBinary(0x80000000));
        System.out.println(numberOfOneInBinary(9));
    }

    private static int numberOfOneInBinary(int n) {
        int number = 0;
        int flag = 1;
        while (flag != 0) {
            if ((n & flag) != 0) {
                number++;
            }
            flag = flag << 1;
        }
        return number;
    }
}
```
以上算法的复杂度为 $O(n)$ 其中 n 为二进制的位数

#### 优化一
考虑当 n 不等于 0 时 (其二进制表示必包含一个 1)
- n 最后一位为 1 时减去 1, 只有最后一位变为 0, 它之前的位不变
- n 最后一位不为 1 时, 最右边的 1 会变为 0, 它右边的位变为 1, 它左边的位不变

在这两种情况中, 发现把一个整数减去 1, 都是变最右边的 1 变为了 0, 它的左边的位不变; 这样就找到一个 1, 如果再把它右边变为 1 的为全部置 0 (例如: `n & (n -1)`), 再循环以上过程直到整数变为 0, 那么就可以统计出这个整数二进制表示中 1 的个数
```
public class NumberOfOneInBinary {
    public static void main(String[] args) {
        System.out.println(numberOfOneInBinary(0x80000000));
        System.out.println(numberOfOneInBinary(9));
    }

    private static int numberOfOneInBinary(int n) {
        int number = 0;
        while (n != 0) {
            number++;
            n = n & (n - 1);
        }
        return number;
    }
}
```
以上算法的复杂度为 $O(n)$, 其中 n 为二进制的位数中 为 1 的个数

#### 优化二
参考 `Integer#bitCount` 方法和[解析](https://segmentfault.com/a/1190000015763941), 主要思路为分组统计 1 的个数, 在逐步将分组统计相加
```
private static int numberOfOneInBinary4(int n) {
    n = n - ((n >>> 1) & 0x55555555);
    n = (n & 0x33333333) + ((n >>> 2) & 0x33333333);
    n = (n + (n >>> 4)) & 0x0f0f0f0f;
    n = n + (n >>> 8);
    n = n + (n >>> 16);
    return n & 0x3f;
}
```

#### 扩展
TODO
