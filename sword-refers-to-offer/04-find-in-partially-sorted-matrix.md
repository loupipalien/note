---
layout: post
title: "二维数组的查找"
date: "2019-05-29"
description: "二维数组的查找"
tag: [algorithm]
---

### 二维数组的查找

#### 题目一
在一个二维数组中, 每一行都按照从左到右递增的顺序排序, 每一列都按照从上到下递增的顺序排序; 请完成一个函数, 输入这样的一个二维数组和一个整数, 判断数组中是否含有该整数

##### 思路
为了找到是否包含目标数字, 那么必然需要与矩阵中的数字比较; 由于矩阵按行从左到右, 按列从上到下递增, 是一个有序的矩阵; 那么我们可以考虑模仿二分法在有序数组中查找目标数字的思维, 将目标数字与中间值比较, 来缩小比较的范围直到结束; 这里的中间值可以是矩阵右上角 (或者是左下角), 当目标值大于右上角的值那么第一行则可以排除, 如果小于则可以排除最后一列, 在子矩阵中重复这个过程到结束即可

##### 实现
```
public class FindInPartiallySortedMatrix {
    public static void main(String[] args) {
        int[][] matrix = new int[][]{{1, 2, 8 ,9}, {2, 4, 9, 12}, {4, 7, 10, 13}, {6, 8, 11, 15}};
        int target = 7;
        System.out.println(findInPartiallySortedMatrix(matrix, target));
    }

    private static boolean findInPartiallySortedMatrix(int[][] matrix, int target) {
        // check matrix
        if (matrix == null) {
            return false;
        }
        for (int i = 0; i < matrix.length; i++) {
            if (matrix[i] == null) {
                return false;
            }
        }

        int row = 0;
        int column = matrix[0].length - 1;
        while (row < matrix.length && column >= 0) {
            if (matrix[row][column] < target) {
                row++;
            } else if (matrix[row][column] > target) {
                column--;
            } else {
                return true;
            }
        }
        return false;
    }
}
```
这个方法的时间复杂度为 $ O(n) $
