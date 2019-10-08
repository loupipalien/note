---
layout: post
title: "n 个骰子的点数"
date: "2019-07-25"
description: "n 个骰子的点数"
tag: [algorithm]
---

### n 个骰子的点数

#### 题目
把 n 个骰子仍在地上, 所有骰子朝上的一面的点数之和为 s; 输入 n, 打印出 s 的所有可能的值出现的概率

##### 思路
想求出 n 个骰子的点数和, 可以先把 n 个骰子分为两堆: 第一堆只有一个, 另一堆有 n - 1 个; 单独的那个有 6 种可能, 后续还需要计算 1 ~ 6 的每一种点数个剩余 n - 1 骰子来计算点数和; 按照上述规则, 将堆分解为 1 个骰子; 这是一种递归的思路; 这里可以定义一个长度为 6n - n + 1 的数组, 将和为 s 的点数出现的次数保存到数组的第 s - n 的数组

##### 实现
```
public class Solution {
    public static void main(String[] args) {
        dicesProbability(2);
    }

    private static void dicesProbability(int number) {
        if (number < 1) {
            return;
        }

        int[] probabilities = new int[5 * number + 1];
        dicesProbability(number, number, 0, probabilities);
        double total = Math.pow(6, number);
        for (int i = number; i <= 6 * number; i++) {
            System.out.println(String.format("sum = %s, times= %d, ratio = %.2f", i, probabilities[i - number], probabilities[i - number] / total));
        }

    }

    private static void dicesProbability(int original, int current, int sum, int[] probabiliteis) {
        if (current == 0) {
            probabiliteis[sum - original]++;
        } else {
            for (int i = 1; i <= 6; i++) {
                dicesProbability(original, current - 1, sum + i, probabiliteis);
            }
        }
    }
}
```

#### 优化
基于递归的实现较为简洁, 但是很多计算是重复的, number 较大时性能较慢

##### 思路
考虑用两个数组来存储骰子点数的每个总数出现的次数; 在第一轮循环中, 第一个数组的第 n 个数字表示骰子和为 n 出现的次数, 在下一轮循环中, 我们加上一个新的骰子, 此时何为 n 的骰子出现的次数应该等于上一轮循环中骰子点数和为 n - 1, n - 2, n - 3, n - 4, n - 5, n - 6 个数字之和

##### 实现
```
public class Solution {
    public static void main(String[] args) {
        dicesProbability(2, 6);
    }

    private static void dicesProbability(int number, int maxValue) {
        if (number < 1) {
            return;
        }

        int[][] probabilities = new int[2][];
        probabilities[0] = new int[number * maxValue + 1];
        probabilities[1] = new int[number * maxValue + 1];
        // flag 用于区别两个数组 probabilities[0] 和 probabilities[1]
        int flag = 0;
        for (int i = 1; i <= maxValue; i++) {
            probabilities[flag][i] = 1;
        }

        for (int i = 2; i <= number; i++) {
            // 当加入一个骰子, 需重新计算 i 到 i * maxValue 出现的次数
            for (int j = i; j <= i * maxValue; j++) {
                // 叠加上一轮循环中骰子点数和为 j - 1 到 j - maxValue 出现的次数
                for (int k = 1; k <= j && k <= maxValue; k++) {
                    probabilities[1 - flag][j] += probabilities[flag][j - k];
                }
            }
            flag = 1 - flag;
        }

        double total = Math.pow(maxValue, number);
        for (int i = number; i <= maxValue * number; i++) {
            System.out.println(String.format("sum = %s, times= %d, ratio = %.2f", i, probabilities[flag][i], probabilities[flag][i] / total));
        }

    }
}
```
