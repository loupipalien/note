---
layout: post
title: "二叉树中和为某一值的路径"
date: "2019-06-29"
description: "二叉树中和为某一值的路径"
tag: [algorithm]
---

### 二叉树中和为某一值的路径

#### 题目
输入一颗二叉树和一个整数, 打印出二叉树中节点值的和为输入整数的所有路径; 从树的根节点开始往下一直到叶节点所经过的节点形成一条路径; 二叉树节点的定义如下
```
class BinaryTreeNode {
    int value = 0;
    BinaryTreeNode left;
    BinaryTreeNode right;

    public BinaryTreeNode(int value) {
        this.value = value;
    }
}
```

##### 思路
由于要遍历二叉树的节点的, 而且首先要变量根节点, 所以选择前序遍历; 在这里使用栈来保存已访问的路径的节点, 到变量到叶子节点时, 计算栈中的路径值的和是否与输入整数相同

##### 实现
```Java
import java.util.Stack;

public class Solution {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(10)
                .setLeft(new BinaryTreeNode(5)
                        .setLeft(new BinaryTreeNode(4))
                        .setRight(new BinaryTreeNode(7)))
                .setRight(new BinaryTreeNode(22));
        pathInTree(root, 22);
    }

    private static void pathInTree(BinaryTreeNode root, int expectedSum) {
        if (root != null) {
            findPath(root, expectedSum, new Stack(), 0);
        }
    }

    private static void findPath(BinaryTreeNode root, int expectedSum, Stack<BinaryTreeNode> traversedNodes, int traversedSum) {
        traversedNodes.push(root);
        traversedSum = traversedSum + root.value;
        if (root.isLeaf() && traversedSum == expectedSum) {
            System.out.println(traversedNodes);
        } else  {
            if (root.left != null) {
                findPath(root.left, expectedSum, traversedNodes, traversedSum);
                traversedNodes.pop();
            }
            if (root.right != null) {
                findPath(root.right, expectedSum, traversedNodes, traversedSum);
                traversedNodes.pop();
            }
        }
    }

    private static class BinaryTreeNode {
        int value;
        BinaryTreeNode left;
        BinaryTreeNode right;

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

        public boolean isLeaf() {
            return left == null && right == null;
        }

        @Override
        public String toString() {
            return String.valueOf(value);
        }
    }
}
```
