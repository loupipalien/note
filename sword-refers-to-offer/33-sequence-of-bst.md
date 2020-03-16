---
layout: post
title: "二叉搜索树的后续遍历序列"
date: "2019-06-28"
description: "二叉搜索树的后续遍历序列"
tag: [algorithm]
---

### 二叉搜索树的后续遍历序列

#### 题目
输入一个整数数组, 判断该数组是不是某二叉搜索树的后续遍历结果; 如果是则返回 `true`, 如果不是则返回 `false`; 假设输入的数组的任意两个数字互不相同; 例如, 输入数组 `{5, 7, 6, 9, 11, 10, 8}`, 则返回 `true`, 因为这个整数序列是下图二叉搜索树的后续遍历结果; 如果输入的数组是 `{7, 4, 6, 5}`, 则由于没有哪棵二叉搜索树的后续遍历结果是这个序列, 因此返回 `false`
```
   --- 8 ---   
   |       |   
 - 6 -   - 10 -
 |   |   |    |
 5   7   9   11
```

##### 思路
首先后续遍历的最后一个节点是根节点, 之前的部分可以分为两部分, 前半部分是根节点的左子树, 后半部分是根节点的右子树; 其次二叉搜索树的符合节点的左子树节点的值都比此节点小, 有子树节点的值都比此节点大; 按照以上性质, 则可以推断
数组 `{5, 7, 6, 9, 11, 10, 8}`, 其中 `8` 是跟节点, `{5, 7, 6}` 是左子树, `{9, 11, 10}` 是右子树, 递归推断子树也符合此性质, 那么此数组是某一个二叉搜索树的后续遍历序列; 数组 `{7, 4, 6, 5}`, 其中 `5` 是根节点, 可判定此节点没有左子树, 但右子树 `{7, 4, 6}` 中有比 `5` 小的节点, 不符合二叉搜索树的定义

##### 实现
```Java
import java.util.Arrays;

public class Solution {
    public static void main(String[] args) {
        System.out.println(sequenceOfBST(new int[]{5, 4, 3, 2, 1}));
        System.out.println(sequenceOfBST(new int[]{5, 7, 6, 9, 11, 10, 8}));
        System.out.println(sequenceOfBST(new int[]{7, 4, 6, 5}));
    }

    private static boolean sequenceOfBST(int[] sequence) {
        if (sequence == null || sequence.length < 1) {
            return false;
        }

        int root = sequence[sequence.length - 1];
        // 确定右子树根节点位置
        int rightChildrenStartIndex = -1;
        for (int i = 0; i < sequence.length - 1; i++) {
            if (sequence[i] > root) {
                rightChildrenStartIndex = i; break;
            }
        }

        // 将根节点的右子树中的节点与根节点比较
        for (int i = rightChildrenStartIndex; i > -1 && i < sequence.length - 1; i++) {
            if (sequence[i] < root) {
                return false;
            }
        }

        if (rightChildrenStartIndex > 0 && rightChildrenStartIndex < sequence.length - 1) {
            return sequenceOfBST(Arrays.copyOfRange(sequence, 0, rightChildrenStartIndex))
                    && sequenceOfBST(Arrays.copyOfRange(sequence, rightChildrenStartIndex, sequence.length - 1));
        }
        return true;
    }
}
```
