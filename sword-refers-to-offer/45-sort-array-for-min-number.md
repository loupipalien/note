---
layout: post
title: "把数组排成最小的数"
date: "2019-07-10"
description: "把数组排成最小的数"
tag: [algorithm]
---

### 把数组排成最小的数

#### 题目
输入一个正整数数组, 把数组里所有数字拼接起来排成一个数, 打印能拼接出的所有数字中的最小的一个; 例如: 输入数组 `{3, 32, 321}`, 则打印出这 3 个数字能排成的最小数字 321323

##### 思路
根据题目要求, 两个数字 m 和 n 能拼接成数字 mn 和 nm, 如果 nm < mn, 那么我们打印出 nm, 即 m 应该排在 n 的前面, 此时定义 m 小于 n; 反之同理

##### 实现
```Java
import java.util.Arrays;

public class Solution {
    public static void main(String[] args) {
        int[] numbers = new int[]{3, 32, 321};
        System.out.println(sortArrayForMinNumber(numbers))    ;
    }

    private static String sortArrayForMinNumber(int[] numbers) {
        if (numbers == null) {
            return "";
        }
        if (numbers.length == 1) {
            return String.valueOf(numbers[0]);
        }

        String[] strs = new String[numbers.length];
        for (int i = 0; i < numbers.length; i++) {
            strs[i] = String.valueOf(numbers[i]);
        }
        Arrays.sort(strs, (str1, str2) -> str1.concat(str2).compareTo(str2.concat(str1)));
        StringBuilder builder = new StringBuilder("");
        for (int i = 0; i < strs.length; i++) {
            builder.append(strs[i]);
        }
        return builder.toString();
    }
}
```
