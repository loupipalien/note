---
layout: post
title: "对称的二叉树"
date: "2019-06-23"
description: "对称的二叉树"
tag: [algorithm]
---

### 对称的二叉树

#### 题目一
请实现一个函数, 用来判断一棵二叉树是不是对称的; 一棵二叉树和它的镜像一样, 那么它是对称的; 例如下图所示的 3 棵二叉树中, 第一棵二叉树是对称的, 而另外两棵不是
```
   --- 8 ---        --- 8 ---         --- 7 ---
   |       |        |       |         |       |
 - 6 -   - 6 -    - 6 -   - 9 -     - 7 -   - 7
 |   |   |   |    |   |   |   |     |   |   |   
 5   7   7   5    5   7   7   5     7   7   7   
```

##### 思路
遍历二叉树通常有三种不同的遍历方式: 前序遍历, 中序遍历, 后序遍历; 如果采用前序遍历二叉树得到的序列, 与前序遍历的对称遍历得到的序列相同, 那么就认为此二叉树是对称的; 前序遍历的对称遍历则是先遍历父节点, 再遍历右孩子, 最后遍历左孩子, 这里需考虑 null 的情况, 因为 null 节点也是对称的一部分

##### 实现
```
public class SymmetricalBinaryTree {
    public static void main(String[] args) {
        BinaryTreeNode node = new BinaryTreeNode(8)
                .setLeft(new BinaryTreeNode(6)
                .setLeft(new BinaryTreeNode(5))
                .setRight(new BinaryTreeNode(7)))
                .setRight(new BinaryTreeNode(6)
                .setLeft(new BinaryTreeNode(7))
                .setRight(new BinaryTreeNode(5)));
        // TODO
    }

    private static boolean symmetricalBinaryTree(BinaryTreeNode node) {
        return symmetricalBinaryTree(node, node);
    }

    private static boolean symmetricalBinaryTree(BinaryTreeNode one, BinaryTreeNode another) {
        if (one == null && another == null) {
            return true;
        }
        if (one == null || another == null) {
            return false;
        }
        if (one.value != another.value) {
            return false;
        }
        return symmetricalBinaryTree(one.left, another.right)
                && symmetricalBinaryTree(one.right, another.left);
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
