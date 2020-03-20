---
layout: post
title: "礼物的最大价值"
date: "2019-07-12"
description: "礼物的最大价值"
tag: [algorithm]
---

### 礼物的最大价值

#### 题目
在一个 m * n 的棋盘的每一格都放有一个礼物, 每个礼物都有一定的价值 (价值大于 0); 你可以从棋盘的左上角开始拿格子里的礼物, 并每次向左或者向下移动一格, 直到到达棋盘的右下角; 给定一个棋盘及其上面的礼物, 请计算你最多能拿多少价值的礼物?

##### 思路
```Java
public class Solution {
    public static void main(String[] args) {
        int[][] values = new int[][]{
                {1, 10, 3, 8},
                {12, 2, 9, 6},
                {5, 7, 4, 11},
                {3, 7, 16, 5}
        };
        System.out.println(maxValueOfGifts(values));
    }

    private static int maxValueOfGifts(int[][] values) {
        if (values == null || values.length == 0 || values[0].length == 0) {
            return 0;
        }

        int rows = values.length;
        int cols = values[0].length;
        int[][] maxValues = new int[rows][cols];
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                int up = 0;
                int left = 0;
                if (i > 0) {
                    up = maxValues[i -1][j];
                }
                if (j > 0) {
                    left = maxValues[i][j - 1];
                }
                maxValues[i][j] = values[i][j] + Math.max(up, left);
            }
        }
        return maxValues[rows - 1][cols - 1];
    }

}
```

##### 优化
可以看到坐标 (i, j) 的格子所能拿到礼物的最大值依赖于坐标 (i, j - 1) 和 (i - 1, j); 这里可以用一个长度为棋盘列数的一维数组来代替矩阵, 数组前 j 个数字为坐标 (i, 0), (i, 1), ..., (i, j - 1) 的最大值, 后 n - j 个数字为坐标 (i - 1, j), (i - 1, j + 1), ..., (i - 1, n - 1) 的最大值; 优化后实现如下
```
public class Solution {
    public static void main(String[] args) {
        int[][] values = new int[][]{
                {1, 10, 3, 8},
                {12, 2, 9, 6},
                {5, 7, 4, 11},
                {3, 7, 16, 5}
        };
        System.out.println(maxValueOfGifts(values));
    }

    private static int maxValueOfGifts(int[][] values) {
        if (values == null || values.length == 0 || values[0].length == 0) {
            return 0;
        }

        int rows = values.length;
        int cols = values[0].length;
        int[] maxValues = new int[cols];
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                int up = 0;
                int left = 0;
                if (i > 0) {
                    up = maxValues[j];
                }
                if (j > 0) {
                    left = maxValues[j - 1];
                }
                maxValues[j] = values[i][j] + Math.max(up, left);
            }
        }
        return maxValues[cols - 1];
    }

}
```
