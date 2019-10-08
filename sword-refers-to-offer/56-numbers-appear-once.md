---
layout: post
title: "数组中数字出现的次数"
date: "2019-07-21"
description: "数组中数字出现的次数"
tag: [algorithm]
---

### 数组中数字出现的次数

#### 题目一: 数组中只出现一次的数字
整型数组里除了两个数字之外, 其他数字都出现了两次; 请写出程序找出这两个只出现一次的数字; 要求时间复杂度是 $O(n)$, 空间复杂度是 $O(1)$

##### 思路
首先简化这个问题为, 整型数组里除了一个数字之外, 其他数字都出现了两次; 这里可以考虑到异或的性质, 任何一个数和自己异或都等于零; 那么从头到尾异或所有数字, 最后得到的就是只出现一次的数字  
现在整型数组里有两个数字只出现一次, 那么这两个数字必然是不同的数字, 那么必然有一位分别为 1 和 0; 按照这一位的分别可以将数组分成两组, 只出现一次的数字分别在一个组里, 这时问题就简化为整型数组里只有一个数字出现了一次的情况

##### 实现
```
public class Solution {
    public static void main(String[] args) {
        int[] array = {2, 4, 3, 6, 3, 2, 5, 5};
        int num1[] = new int[1];
        int num2[] = new int[1];
        numbersAppearOnce(array, num1, num2);
        System.out.println(num1[0]);
        System.out.println(num2[0]);
    }

    private static void numbersAppearOnce(int [] array, int num1[], int num2[]) {
        if (array == null || array.length < 2) {
            throw new IllegalArgumentException("Invalid Parameter.");
        }

        int number = xorNumbers(array);
        int index = getIndexOfFirstBitOne(number);
        for (int i = 0; i < array.length; i++) {
            if (!isBitOneAtIndexOfNumber(array[i], index)) {
                num1[0] ^= array[i];
            } else {
                num2[0] ^= array[i];
            }
        }
    }

    private static boolean isBitOneAtIndexOfNumber(int number, int index) {
        int thePowerOfTwo = (int) Math.pow(2, index - 1);
        return number == (number | thePowerOfTwo);
    }

    private static int getIndexOfFirstBitOne(int number) {
        if (number == 0) {
            return -1;
        }

        int index = 0;
        int thePowerOfTwo = (int) Math.pow(2, index++);
        while (number != (number | thePowerOfTwo)) {
            thePowerOfTwo = (int) Math.pow(2, index++);
        }
        return index;
    }

    private static int xorNumbers(int... numbers) {
        if (numbers == null || numbers.length == 0) {
            throw new IllegalArgumentException("Invalid Parameter.");
        }

        int result = numbers[0];
        for (int i = 1; i < numbers.length; i++) {
            result ^= numbers[i];
        }
        return result;
    }
}
```

#### 题目二: 数组中唯一只出现一次的数字
在一个数组中除一个数字只出现一次之外, 其他数字都出现了三次; 请找出那个只出现一次的数字

##### 思路
如果一个数字出现三次, 那么它的二进制表示的每一位 (0 或 1) 也出现三次, 如果把所有出现三次的数字的二进制表示的每一位都分别加起来, 那么每一位的和都能被 3 整除, 那么那个只出现一次的数字的二进制表示中表示为 0, 否则为 1

##### 实现
```
public class Solution {
    public static void main(String[] args) {
        int[] array = new int[]{1, 2, 4, 2, 2, 6, 1, 1, 4, 4};
        System.out.println(numberAppearingOnce(array));
    }

    private static int numberAppearingOnce(int[] array) {
        if (array == null || array.length == 0) {
            throw new IllegalArgumentException("Invalid Parameter.");
        }

        int[] bitsSum = new int[32];
        for (int i = 0; i < array.length; i++) {
            int bitMask = 1;
            int number = array[i];
            for (int j = 31; j >= 0 ; j--) {
                bitsSum[j] += (number & bitMask) != 0 ? 1 : 0;
                bitMask = bitMask << 1;
            }
        }

        int result = 0;
        for (int i = 0; i < 32; i++) {
            result = result << 1;
            result += bitsSum[i] % 3;
        }
        return result;
    }
}
```
