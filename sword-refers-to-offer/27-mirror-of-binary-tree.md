---
layout: post
title: "二叉树的镜像"
date: "2019-06-22"
description: "二叉树的镜像"
tag: [algorithm]
---

### 二叉树的镜像

#### 题目一
请完成一个函数, 输入一颗二叉树, 该函数输出它的镜像; 二叉树节点的定义如下
```Java
class BinaryTreeNode {
    double value;
    BinaryTreeNode left;
    BinaryTreeNode right;
}
```

##### 思路
从根节点开始, 如果有孩子节点, 则将两个孩子节点左右互换, 然后再将两个孩子节点作为根节点, 循环上述过程, 直到根节点没有孩子为止

##### 实现
```Java
public class MirrorOfBinaryTree {
    public static void main(String[] args) {
        BinaryTreeNode node = new BinaryTreeNode(8)
                .setLeft(new BinaryTreeNode(6)
                .setLeft(new BinaryTreeNode(5))
                .setRight(new BinaryTreeNode(7)))
                .setLeft(new BinaryTreeNode(10)
                .setLeft(new BinaryTreeNode(9))
                .setRight(new BinaryTreeNode(11)));
        // TODO
    }

    private static void mirrorOfBinaryTree(BinaryTreeNode root) {
        if (root == null) return root;

        BinaryTreeNode left = root.left;
        BinaryTreeNode right = root.right;
        root.left = mirrorTree(right);
        root.right = mirrorTree(left);
        return root;
    }

    private static class BinaryTreeNode {
        double value;
        BinaryTreeNode left;
        BinaryTreeNode right;

        public BinaryTreeNode(double value) {
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
    }
}
```
