---
layout: post
title: "股票的最大利润"
date: "2019-07-28"
description: "股票的最大利润"
tag: [algorithm]
---

### 股票的最大利润

#### 题目
假设把某股票的价格按照时间先后顺序存储在数组中, 请问买卖该股票一次可能获得的最大利润是多少? 例如: 一只股票在某些时间节点的价格为 `{9, 11, 8, 5, 7, 12, 16, 14}`; 如果我们能在价格为 5 的时候买入并在价格 16 时卖出, 则能收获最大的利润为 11

##### 思路
使用蛮力法, 比较两个数之间的差值, 可以在 $O(n^2)$ 的时间复杂度下找到差值最大的两个数; 但这种解法包含这许多不需要计算的数对, 例如当后一个数为 16 时, 获得的最大利润其实不需要比较与 9, 11 等数字的差值, 只要与它之前的最小值比较即可, 所以在遍历时只要记住当前数之前的最小值, 再比较各个值为后一个值时获得利润的最大值即可

##### 实现
```Java
public class Solution {
    public static void main(String[] args) {
        int[] numbers = new int[]{9, 11, 8, 5, 7, 12, 16, 14};
        System.out.println(maximalProfile(numbers));
    }

    private static int maximalProfile(int[] numbers) {
        if (numbers == null || numbers.length < 2) {
            return 0;
        }

        int max = numbers[1] - numbers[0];
        int min = max < 0 ? numbers[1] : numbers[0];
        for (int i = 2; i < numbers.length; i++) {
            int currentMax = numbers[i] - min;
            if (currentMax < 0) {
                min = numbers[i];
            }
            if (currentMax > max) {
                max = currentMax;
            }
        }
        return max;
    }
}
```
