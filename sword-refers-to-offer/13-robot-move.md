---
layout: post
title: "机器人的运动范围"
date: "2019-06-09"
description: "机器人的运动范围"
tag: [algorithm]
---

### 机器人的运动范围

#### 题目一
地上有一个 m 行 n 列的方格; 一个机器人从坐标 (0,0) 的格子开始移动, 它每次可以向左, 右, 上, 下移动一格, 但不能进入行坐标和列坐标的数位之和大于 k 的格子; 例如, 当 k 为 18 时, 机器人能够进入方格 (35, 37), 因为 `3 + 5 + 3 + 7 = 18`; 但它不能进入方格 (35, 38), 因为 `3 + 5 + 3 + 8 = 19`; 请问机器人能够到达多个个格子?

##### 思路
机器人从坐标 (0,0) 开始移动; 当它准备进入坐标 (i,j) 的格子时, 通过检查坐标的数位和来判断机器人是否能够进入; 如果机器人能够进入坐标为 (i,j) 的格子, 则再判断它能否进入 4 个相邻的格子 (i, j-1), (i-1,j), (i,j+1), (i+1,j); 因此可以使用回溯法来实现

##### 实现
```
public class RobotMove {
    public static void main(String[] args) {
        System.out.println(robotMove(5, 10, 10));
    }

    private static int robotMove(int threshold, int rows, int cols) {
        if (threshold < 0 || rows <= 0 || cols <= 0) {
            return 0;
        }
        return moveCount(threshold, rows, cols, 0, 0, new boolean[rows * cols]);
    }

    private static int moveCount(int threshold, int rows, int cols, int row, int col, boolean[] visited) {
         int count = 0;

         if (row >= 0 && row < rows && col >= 0 && col < cols
                 && checkDigitSum(threshold, row, col)
                 && !visited[row * cols + col]) {
             visited[row * cols + col] = true;
             // 相邻格子: [row, col + 1], [row + 1, col], [row, col -1], [row - 1, col]
             count = 1 + moveCount(threshold, rows, cols, row, col + 1, visited)
                     + moveCount(threshold, rows, cols, row + 1, col, visited)
                     + moveCount(threshold, rows, cols, row, col -1 , visited)
                     + moveCount(threshold, rows, cols, row - 1, col, visited);
        }
        return count;
    }

    private static boolean checkDigitSum(int threshold, int row, int col) {
        return threshold >= getDigitSum(row) + getDigitSum(col);
    }

    private static int getDigitSum(int num) {
        int sum = 0;
        while (num > 0) {
            sum += num % 10;
            num = num / 10;
        }
        return sum;
    }
}
```
