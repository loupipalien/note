---
layout: post
title: "矩阵中的路径"
date: "2019-06-08"
description: "矩阵中的路径"
tag: [algorithm]
---

### 矩阵中的路径

#### 题目一
请设计一个函数, 用来判断在一个矩阵中是否存在一条包含某字符串所有字符的路径; 路径可以从矩阵中的任意一格开始, 每一步可以在矩阵中向左, 右, 上, 下移动一格; 如果一条路径经过了矩阵的某一格, 那么该路径不能再次进入该格子; 例如, 在下面 3 * 4 的矩阵中包含了一条字符串 "bfce" 的路径; 但矩阵中不包含字符串 "abfb" 的路径, 因为字符串的第一个字符 b 占据了矩阵中的第一行第二个格子之后, 路径不能再次进入这个格子
```
a  b  t  g
c  f  c  s
j  d  e  h
```

##### 思路
这是一个可以用回溯法解决的典型题, 由于回溯法的特性, 可将路径看作为一个栈; 首先在矩阵中选取任意格子作为路径的起点, 若与字符串不匹配则再选取下一个格子, 若与字符串匹配则放入栈中, 并标记为已访问; 当在矩阵中定位了路径中前 n 个字符的位置后, 在与第 n 个字符所在的格子周围都没有找到第 n + 1 个字符, 这时候需要回到第 n - 1 个字符, 并将第 n 个字符所在的格子重新标记位未未访问, 重新第定位第 n 个字符  
由于路径不能重复进入矩阵的格子, 所以还需要定义和字符矩阵大小一样的布尔值矩阵, 用于标识路径是否已经进入了每个格子  

##### 实现
```Java
public class StringPathInMatrix {
    public static void main(String[] args) {
        char[] matrix = {'a', 'b', 't', 'g', 'c', 'f', 'c', 's', 'j', 'd', 'e', 'h'};
        char[] str = {'b', 'f', 'c', 'e'};
        System.out.println(stringPathInMatrix(matrix, 3, 4, str));
    }

    private static boolean stringPathInMatrix(char[] matrix, int rows, int cols, char[] str) {
        if (matrix == null || rows < 1 || cols < 1 || str == null) {
            return false;
        }
        if (matrix.length < str.length) {
            return false;
        }
        if (matrix.length != rows * cols) {
            throw new IllegalArgumentException("Invalid Parameters.");
        }

        boolean[] visited = new boolean[rows * cols];
        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (hasPath(matrix, rows, cols, str, i, j, 0, visited)) return true;
            }
        }
        return false;
    }

    private static boolean hasPath(char[] matrix, int rows, int cols, char[] str, int row, int col, int length, boolean[] visited) {
        if (length == str.length) return true;

        boolean hasPath = false;
        // 判定当前格子
        if (row >= 0 && row < rows && col >= 0 && col < cols
                && matrix[row * cols + col] == str[length++]
                && !visited[row * cols + col]) {
            visited[row * cols + col] = true;
            // 相邻格子: [row, col + 1], [row + 1, col], [row, col -1], [row - 1, col]
            hasPath = hasPath(matrix, rows, cols, str, row, col + 1, length, visited)
                    || hasPath(matrix, rows, cols, str, row + 1, col, length, visited)
                    || hasPath(matrix, rows, cols, str, row, col - 1, length, visited)
                    || hasPath(matrix, rows, cols, str, row - 1, col, length, visited);
            if (!hasPath) visited[row * cols + col] = false;
        }
        return hasPath;
    }
}
```
