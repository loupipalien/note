---
layout: post
title: "顺时针打印矩阵"
date: "2019-06-24"
description: "顺时针打印矩阵"
tag: [algorithm]
---

### 顺时针打印矩阵

#### 题目一
输入一个矩阵, 按照从外向里以顺时针的顺序依次打印出每一个数字; 例如, 如果输入如下矩阵
```
1   2   3   4
5   6   7   8
9   10  11  12
13  14  15  16
```
则依次打印出数字: 1, 2, 3, 4, 8, 12, 16, 15, 14, 13, 9, 5, 6, 7, 11, 10

##### 思路
从外向里以顺时针打印出每一个数字, 可以把这个任务拆解为按圈打印: 即先顺时针打印矩阵最外一圈的矩形, 再依次打印内部的矩形, 直到矩形的左上角点的坐标小于右下角的坐标; 这里需要注意退化情况的处理, 即一行或一列的情况

##### 实现
```
import java.util.ArrayList;
import java.util.List;

public class PrintMatrix {
    public static void main(String[] args) {
        int[][] matrix = {
                {1, 2, 3, 4},
                {5, 6, 7, 8},
                {9, 10, 11, 12},
                {13, 14, 15, 16}
        };
        System.out.println(printMatrix(matrix));
    }

    private static List<Integer> printMatrix(int[][] matrix) {
        List<Integer> list = new ArrayList<>();
        if (matrix == null || matrix[0] == null) {
            return list;
        }

        int topLeftR = 0;
        int topLeftC = 0;
        int bottomRightR = matrix.length - 1;
        int bottomRightC = matrix[bottomRightR].length - 1;
        while (topLeftR <= bottomRightR && topLeftC <= bottomRightC) {
            list.addAll(printEdge(matrix, topLeftR++, topLeftC++, bottomRightR--, bottomRightC--));
        }
        return list;
    }

    private static List<Integer> printEdge(int[][] matrix, int topLeftR, int topLeftC, int bottomRightR, int bottomRightC) {
        List<Integer> list = new ArrayList<>();
        // 只有一行时
        if (topLeftR == bottomRightR) {
            for (int i = topLeftC; i <= bottomRightC; i++) {
                list.add(matrix[topLeftR][i]);

            }
        }
        // 只有一列时
        else if (topLeftC == bottomRightC) {
            for (int i = topLeftR; i <= bottomRightR ; i++) {
                list.add(matrix[i][topLeftC]);
            }
        }
        // 一般情况: 矩形
        else {
            // 矩形的上边
            for (int i = topLeftC; i <= bottomRightC; i++) {
                list.add(matrix[topLeftR][i]);
            }
            // 矩形的右边
            for (int i = topLeftR + 1; i <= bottomRightR; i++) {
                list.add(matrix[i][bottomRightC]);
            }
            // 矩形的下边
            for (int i = bottomRightC - 1; i >= topLeftC ; i--) {
                list.add(matrix[bottomRightR][i]);
            }
            // 矩形的左边
            for (int i = bottomRightR - 1; i > topLeftR ; i--) {
                list.add(matrix[i][topLeftC]);
            }
        }
        return list;
    }
}
```
