---
layout: post
title: "和为 s 的数字"
date: "2019-07-22"
description: "和为 s 的数字"
tag: [algorithm]
---

### 和为 s 的数字

#### 题目一: 和为 s 的两个数字
输入一个递增排序的数组和一个数字 s, 在数组中查找两个数, 使得它们的和正好是 s; 如果有多对数字的和等于 s, 则输出任意一对即可

##### 思路
定义两个指针, `ahead` 从前向后遍历, `behind` 从后向前遍历; 当两个数的和大于 s, `behind` 向前移动, 当两个数的和小于 s, `ahead` 向后移动

##### 实现
```
import java.util.ArrayList;
import java.util.List;

public class Solution {
    public static void main(String[] args) {
        int[] array = new int[]{1, 2, 4, 7, 11, 15};
        System.out.println(twoNumbersWithSum(array, 15));
    }

    private static List<Integer> twoNumbersWithSum(int[] array, int sum) {
        List<Integer> numbers = new ArrayList<>();
        if (array == null || array.length < 2) {
            return numbers;
        }

        int i = 0;
        int j = array.length - 1;
        while (array[i] + array[j] != sum && i < j) {
            if (array[i] + array[j] < sum) {
                i++;
            } else {
                j--;
            }
        }
        if (i < j) {
            numbers.add(array[i]);
            numbers.add(array[j]);
        }
        return numbers;
    }
}
```

#### 题目二: 和为 s 的连续正数序列
输入一个正数 s, 打印出所有和为 s 的连续正数序列 (至少含有两个数); 例如: 输入 15, 由于 `1 + 2 + 3 + 4 + 5 = 4 + 5 + 6 = 7 + 8 = 15`, 所以打印出 3 个连续序列 `1 ~ 5`, `4 ~ 6`, `7 ~ 8`

##### 思路
使用两个指针 start, end 指向连续正数序列的起始, 当连续正数序列的和小于指定值则 end 指针后移一位, 如果连续正数序列的和大于指定的值则 start 指针后移一位, 相等时则保存序列; 当 start 值大于等于指定值的一半时停止

##### 实现
```
import java.util.ArrayList;
import java.util.List;

public class Solution {
    public static void main(String[] args) {
        System.out.println(continuousSequenceWithSum(15));
    }

    private static List<List<Integer>> continuousSequenceWithSum(int sum) {
        List<List<Integer>> sequences = new ArrayList<>();
        if (sum < 3) {
            return sequences;
        }

        int start = 1;
        int end = 2;
        int curSum = 3;
        while (start <= (sum >> 1)) {
            if (curSum < sum) {
                curSum += ++end;
            } else {
                if (curSum == sum) {
                    List<Integer> sequence = new ArrayList<>();
                    for (int i = start; i <= end; i++) {
                        sequence.add(i);
                    }
                    sequences.add(sequence);
                }
                curSum -= start++;
            }
        }
        return sequences;
    }
}
```
