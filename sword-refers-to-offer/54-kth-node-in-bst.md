---
layout: post
title: "二叉搜索树的第 k 大节点"
date: "2019-07-19"
description: "二叉搜索树的第 k 大节点"
tag: [algorithm]
---

### 二叉搜索树的第 k 大节点

#### 题目
给定一棵二叉搜索树, 请找出其中第 k 大的节点; 例如: 在下图中的二叉搜索树里, 按节点数值大小顺序, 第三大节点的值是 4
```
   ----- 5 -----      
   |           |      
--- 3 ---  --- 7 ---
|       |  |       |    
2       4  6       8
```

##### 思路
中序遍历二叉搜索树, 遍历序列的数值是递增排序的; 所以使用中序遍历, 很容易找到第 k 大的节点

##### 实现
```
import java.util.ArrayList;
import java.util.List;

public class Solution {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(5)
                .setLeft(new BinaryTreeNode(3)
                        .setLeft(new BinaryTreeNode(2))
                        .setRight(new BinaryTreeNode(4)))
                .setRight(new BinaryTreeNode(7)
                        .setLeft(new BinaryTreeNode(6))
                        .setRight(new BinaryTreeNode(8)));
        System.out.println(kthNodeInBST(root, 6));
    }

    private static BinaryTreeNode kthNodeInBST(BinaryTreeNode root, int kth) {
        if (root == null || kth < 1) {
            return null;
        }
        List<BinaryTreeNode> nodes = new ArrayList<>();
        kthNodeInBST(root, kth, nodes);
        return kth == nodes.size() ? nodes.get(nodes.size() - 1) : null;
    }

    private static void kthNodeInBST(BinaryTreeNode root, int kth, List<BinaryTreeNode> nodes) {
        if (root.left != null) {
            kthNodeInBST(root.left, kth, nodes);
        }
        if (nodes.size() < kth) {
            nodes.add(root);
        } else {
            return;
        }
        if (root.right != null) {
            kthNodeInBST(root.right, kth, nodes);
        }
    }

    private static class BinaryTreeNode {
        private int value;
        private BinaryTreeNode left;
        private BinaryTreeNode right;

        public BinaryTreeNode(int value) {
            this.value = value;
        }

        public BinaryTreeNode setLeft(BinaryTreeNode left) {
            this.left = left;
            return this;
        }

        public BinaryTreeNode setRight(BinaryTreeNode right) {
            this.right = right;
            return this;
        }

        @Override
        public String toString() {
            return "BinaryTreeNode{" +
                    "value=" + value +
                    '}';
        }
    }
}
```
