---
layout: post
title: "扑克牌中的顺子"
date: "2019-07-26"
description: "扑克牌中的顺子"
tag: [algorithm]
---

### 扑克牌中的顺子

#### 题目
从扑克牌中随机抽 5 张牌, 判断是不是一个顺子, 即这 5 张牌是不是连续的; 2 ~ 10 为数字本身, A 为 1, J 为 11, Q 为 12, K 为 13, 而大小王可以看成任意数字

##### 思路
首先可以将这 5 张牌排序, 然后计算非零的相邻间隔数是否大于零的个数; 如果有相同点数, 即有对子, 那可以直接判断不能称为顺子

##### 实现
```
import java.util.Arrays;

public class Solution {
    public static void main(String[] args) {
        int[] numbers = new int[]{3, 5, 5, 7, 4};
        System.out.println(continuousCards(numbers));
    }

    private static boolean continuousCards(int[] numbers) {
        if (numbers == null || numbers.length < 1) {
            return false;
        }

        // 排序
        Arrays.sort(numbers);
        // 统计零的个数
        int countOfZero = 0;
        for (int i = 0; i < numbers.length && numbers[i] == 0; i++) {
            countOfZero++;
        }
        // 统计间隔的个数
        int countOfGap = 0;
        for (int i = countOfZero; i < numbers.length - 1; i++) {
            if (numbers[i] < numbers[i + 1]) {
                countOfGap += numbers[i + 1] - numbers[i] - 1;
            } else {
                return false;
            }
        }
        return countOfZero >= countOfGap ? true : false;
    }
}
```
