---
layout: post
title: "构建乘积数组"
date: "2019-07-31"
description: "构建乘积数组"
tag: [algorithm]
---

### 构建乘积数组

#### 题目
给定一个数组 `A[0, 1, ... n - 1]`, 请构建一个数组 `B[0, 1, ... n - 1]`, 其中 B 中的元素 `B[i] = A[0] * A[1] * ... * A[i - 1] * A[i + 1] * ... * A[n -1]`; 不能使用除法

##### 思路
可以考虑 `B[i] = A[0] * A[1] * ... * A[i - 1] * A[i + 1] * ... * A[n -1]` = `C[i] * D[i]`, 其中 `C[i] =  A[0] * A[1] * ... * A[i - 1]`, `D[i] = A[i + 1] * ... * A[n -1]`, 那么可以先计算 C[i] 赋值到 B[i] 中, 再计算 D[i] 乘以 B[i] 的值再赋值给 B[i]; 由于 C[i] 和 D[i] 这两部分是连续乘积的, 后一步的结果可以直接基于前一步的结果

##### 实现
```
public class Solution {
    public static void main(String[] args) {
        int[] array = new int[]{1, 2, 3, 4};
        int[] result = multiply(array);
        for (int i = 0; i < result.length; i++) {
            System.out.print(result[i] + " ");
        }
    }

    private static int[] multiply(int[] array) {
        if (array == null || array.length == 0) {
            return array;
        }

        int[] result = new int[array.length];
        int temp = 1;
        for (int i = 0; i < array.length; i++) {
            result[i] = temp;
            temp *= array[i];
        }
        temp = 1;
        for (int i = array.length - 1; i >=0 ; i--) {
            result[i] *= temp;
            temp *= array[i];
        }
        return result;
    }
}
```
