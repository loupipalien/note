---
layout: post
title: "打印从 1 到最大的 n 位数"
date: "2019-06-12"
description: "打印从 1 到最大的 n 位数"
tag: [algorithm]
---

### 打印从 1 到最大的 n 位数

#### 题目一
输入数字 n, 按顺序打印出 1 到最大的 n 位十进制数; 比如输入 3, 则打印出 1, 2, 3 一直到最大 3 位数的 999

##### 思路
由于没有规定 n 的大小, 那么当 n 很大的时候, 最大的 n 位数必然超过了 int 和 long 型的数字大小, 所以考虑使用字符串来表示; 使用长度为 n 的 char 数组, 先将其所有元素初始化为 `'0'` 初始化, 然后从 1 自增到最大的 n 位数; 这里的字符数组需要模拟自增的操作, 最终打印时要考虑避免打印出前缀 `0`

##### 实现
```Java
public class PrintOneToMaxOfNDigits {
    public static void main(String[] args) {
        printOneToMaxOfNDigits(3);
    }

    private static void printOneToMaxOfNDigits(int n) {
        if (n <= 0) {
            throw new IllegalArgumentException("Invalid parameter.");
        }

        // 初始化
        char[] number = new char[n];
        for (int i = 0; i < number.length; i++) {
            number[i] = '0';
        }

        // 打印
        while (!increment(number)) {
            println(number);
        }
    }

    private static boolean increment(char[] number) {
        boolean isOverflow = false;
        int carry = 0;
        for (int i = number.length - 1; i >= 0 ; i--) {
            int nDigit = number[i] - '0' + carry;
            // 最后一位
            if (i == number.length - 1) {
                nDigit++;
            }

            // 根据是否有进位计算
            if (nDigit == 10) {
                if (i == 0) {
                    isOverflow = true;
                } else {
                    carry = 1;
                    number[i] = '0';
                }
            } else {
                number[i] = (char) ('0' + nDigit);
                break;
            }
        }
        return isOverflow;
    }

    private static void println(char[] number) {
        for (int i = 0; i < number.length; i++) {
            if (number[i] != '0') {
                System.out.println(new String(Arrays.copyOfRange(number, i, number.length)));
                break;
            }
        }
    }
}
```

#### 优化
换一种思路考虑顺序打印 1 到最大的 n 位数: n 位所有的十进制数就是 n 个 0 到 9 数字的全排列; 也就是说把数字的每一位从 0 到 9 排列一遍就得到了所有的十进制数, 然后将其打印即可
```Java
public class Solution {
    public static void main(String[] args) {
        printOneToMaxOfNDigits(1);
    }

    private static void printOneToMaxOfNDigits(int n) {
        if (n <= 0) {
            throw new IllegalArgumentException("Invalid parameter.");
        }

        // 初始化
        char[] number = new char[n];
        for (int i = 0; i < number.length; i++) {
            number[i] = '0';
        }

        // 打印
        for (int i = 0; i < 10; i++) {
            number[0] = (char) (i + '0');
            printOneToMaxOfNDigitsRecursively(number, 0);
        }

    }

    private static void printOneToMaxOfNDigitsRecursively(char[] number, int index) {
        if (index == number.length - 1) {
            println(number);
            return;
        }

        for (int i = 0; i < 10; i++) {
            number[index + 1] = (char) ('0' + i);
            printOneToMaxOfNDigitsRecursively(number, index + 1);
        }
    }

    private static void println(char[] number) {
        for (int i = 0; i < number.length; i++) {
            if (number[i] != '0') {
                System.out.println(new String(Arrays.copyOfRange(number, i, number.length)));
                break;
            }
        }
    }
}
```

#### 扩展
TODO
