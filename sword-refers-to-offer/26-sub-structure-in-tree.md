---
layout: post
title: "树的子结构"
date: "2019-06-21"
description: "树的子结构"
tag: [algorithm]
---

### 树的子结构

#### 题目一
输入两颗二叉树 A 和 B, 判断 B 是不是 A 的子结构; 二叉树节点定义如下
```Java
class BinaryTreeNode {
    double value;
    BinaryTreeNode left;
    BinaryTreeNode right;
}
```

##### 思路
要在 A 树中查找 B 树结构一样的子树, 实现过程可以分为两步
- 在树 A 中找到和树 B 的根节点的值一样的节点 R
- 判断树 A 以 R 为根节点的子树是不是包含和树 B 一样的结构

##### 实现
```Java
public class SubStructureInTree {
    public static void main(String[] args) {
        BinaryTreeNode node4 = new BinaryTreeNode(4);
        BinaryTreeNode node7 = new BinaryTreeNode(7);
        BinaryTreeNode node2 = new BinaryTreeNode(2).setLeft(node4).setRight(node7);
        BinaryTreeNode node9 = new BinaryTreeNode(9);
        BinaryTreeNode node8 = new BinaryTreeNode(8).setLeft(node9).setRight(node2);
        BinaryTreeNode root = new BinaryTreeNode(8).setLeft(node8).setRight(new BinaryTreeNode(7));
        BinaryTreeNode subTreeRoot = new BinaryTreeNode(8).setLeft(new BinaryTreeNode(9)).setRight(new BinaryTreeNode(2));

        System.out.println(subStructureInTree(root, subTreeRoot));
    }

    private static boolean subStructureInTree(BinaryTreeNode root, BinaryTreeNode subTreeRoot) {
        boolean result = false;

        if (root != null && subTreeRoot != null) {
            if (Double.compare(root.value, subTreeRoot.value) == 0) {
                result = doesTreeHasSubTree(root, subTreeRoot);
            }
            if (!result) {
                result = subStructureInTree(root.left, subTreeRoot);
            }
            if (!result) {
                result = subStructureInTree(root.right, subTreeRoot);
            }
        }
        return result;
    }

    private static boolean doesTreeHasSubTree(BinaryTreeNode root, BinaryTreeNode subTreeRoot) {
        if (subTreeRoot == null) {
            return true;
        }
        if (root == null) {
            return false;
        }
        if (Double.compare(root.value, subTreeRoot.value) != 0) {
            return false;
        }
        return doesTreeHasSubTree(root.left, subTreeRoot.left) && doesTreeHasSubTree(root.right, subTreeRoot.right);
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
在计算机内判断两个浮点数时, 由于精度的误差, 不能使用 `==` 来判断两个浮点数的大小, 应该使用其包装类型的 `compare` 方法; 另外, 也可以先将浮点数转换为包装类型或者 BigDecimal 类型, 然后使用 `compareTo` 方法比较
