---
layout: post
title: "连续子数组的最大和"
date: "2019-07-07"
description: "连续子数组的最大和"
tag: [algorithm]
---

### 连续子数组的最大和

#### 题目
输入一个整型数组, 数组里有正数也有负数; 数据中的一个或连续多个整数组成一个子数组; 求所有子数组的和的最大值; 要求时间复杂度为 $O(n)$; 例如: 输入的数组为 `{1, -2, 3, 10, -4, 7, 2, -5}`, 和最大的子数组为 `{3, 10, -4, 7, 2}`, 因此输出为该子数组的和 18

##### 思路
- 举例分析数组的规律
试着从头到尾累加数组中的数字, 第一个数字和第二个数字的和是 -1, 所以已这两个元素开始的子数组不会是最大和, 所以直接从第三个数字开始, 第三个数字和第四个数字之和为 13, 第五个数字为 -4, 和为 9; 由于 13 的值大于 9, 把 13 保存下来, 因为它有可能是最大值, 再继续向后累加, 与第六个数字相加后和为 16, 此时更新最大值, 并继续向后累加直到结束

| 步骤 | 操作 | 累加的子数组和 | 最大的子数组和 |
| :--- | :--- | :--- | :--- |
| 1 | + 1 | 1 | 1 |
| 2 | - 2 | -1 | -1 |
| 3 | 抛弃之前的和 -1, 直接 + 3 | 3 | 3 |
| 4 | + 10 | 13 | 13 |
| 5 | - 4 | 9 | 9 |
| 6 | + 7 | 16 | 16 |
| 7 | + 2 | 18 | 18 |
| 8 | - 5 | 13 | 18 |

- 应用动态规划法

##### 实现
- 举例分析数组的规律
```Java
public class GreatestSumOfSubarrays {
    public static void main(String[] args) {
        System.out.println(greatestSumOfSubarrays(new int[]{1, -2, 3, 10, -4, 7, 2, -5}));
    }

    private static int greatestSumOfSubarrays(int[] array) {
        if (array == null || array.length == 0) {
            throw new IllegalArgumentException("Invalid Parameter.");
        }

        int sum = 0;
        int max = Integer.MIN_VALUE;
        for (int i = 0; i < array.length; i++) {
            // 当 sum 的值小于等于零时, 则抛弃之前元素的和
            sum = sum > 0 ? sum + array[i] : array[i];
            // 保存较大值
            if (sum > max) {
                max = sum;
            }
        }
        return max;
    }

}
```
