---
layout: post
title: "二叉数的深度"
date: "2019-07-20"
description: "二叉数的深度"
tag: [algorithm]
---

### 二叉数的深度

#### 题目一
输入一棵二叉树的根节点, 求该树的深度; 从根节点到叶节点一次经过的节点 (含根, 叶节点) 形成树的一条路径, 最长路径的长度为树的深度; 二叉树节点定义如下
```
class BinaryTreeNode {
    int value;
    BinaryTreeNode left;
    BinaryTreeNode right;

    public BinaryTreeNode(int value) {
        this.value = value;
    }
}
```

##### 思路
树的深度等于其子树的最大深度加一;

##### 实现
```Java
public class Solution {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(1)
                .setLeft(new BinaryTreeNode(2)
                        .setLeft(new BinaryTreeNode(4))
                        .setRight(new BinaryTreeNode(5)
                                .setLeft(new BinaryTreeNode(7))))
                .setRight(new BinaryTreeNode(3)
                        .setRight(new BinaryTreeNode(6)));
        System.out.println(treeDepth(root));
    }

    private static int treeDepth(BinaryTreeNode root) {
        if (root == null) {
            return 0;
        }

        int left = treeDepth(root.left);
        int right = treeDepth(root.right);
        return left > right ? left + 1 : right + 1;
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
    }
}
```

#### 题目二
平衡二叉树;
输入一棵二叉树的根节点, 判断该树是不是平衡二叉树; 如果某二叉树中任意节点的左右子树的深度相差不超过 1, 那么它就是一棵平衡二叉树; 例如下图的二叉树就是平衡二叉树
```
    ----- 1 -----      
    |           |      
--- 2 ---      3 ---
|       |          |    
4     - 5          6
      |
      7
```

##### 思路
从根节点开始比较两个子树的高度差, 当高度差超过 1 返回 false, 否则继续递归其子节点

##### 实现
```Java
public class Solution {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(1)
                .setLeft(new BinaryTreeNode(2)
                        .setLeft(new BinaryTreeNode(4))
                        .setRight(new BinaryTreeNode(5)
                                .setLeft(new BinaryTreeNode(7))))
                .setRight(new BinaryTreeNode(3)
                        .setRight(new BinaryTreeNode(6)));
        System.out.println(balancedBinaryTree(root));
    }

    private static boolean balancedBinaryTree(BinaryTreeNode root) {
        if (root == null) {
            return true;
        }

        int leftDepth = treeDepth(root.left);
        int rightDepth = treeDepth(root.right);
        return (Math.abs(leftDepth - rightDepth) <= 1)
                && (balancedBinaryTree(root.left) && balancedBinaryTree(root.right));
    }

    private static int treeDepth(BinaryTreeNode root) {
        if (root == null) {
            return 0;
        }

        int left = treeDepth(root.left);
        int right = treeDepth(root.right);
        return left > right ? left + 1 : right + 1;
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
    }
}
```

#### 优化
以上算法, 自上而下遍历节点, 计算其子树高度时有很多重复计算, 如果能自下而上的计算则能避免重复计算; 如果使用后序遍历二叉树的每一个节点, 那么在遍历到一个节点之前就已经遍历了它的左右子树, 只要在遍历每个节点的时候记录它的深度, 就可以一边遍历一边判断节点是不是平衡的
```Java
public class Solution {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(1)
                .setLeft(new BinaryTreeNode(2)
                        .setLeft(new BinaryTreeNode(4))
                        .setRight(new BinaryTreeNode(5)
                                .setLeft(new BinaryTreeNode(7))))
                .setRight(new BinaryTreeNode(3)
                        .setRight(new BinaryTreeNode(6)));
        System.out.println(balancedBinaryTree(root));
    }

    private static boolean balancedBinaryTree(BinaryTreeNode root) {
        return balancedBinaryTree(root, Holder.of(0));
    }

    private static boolean balancedBinaryTree(BinaryTreeNode root, Holder<Integer> depth) {
        if (root == null) return true;

        Holder<Integer> leftDepth = Holder.of(0);
        Holder<Integer> rightDepth = Holder.of(0);
        if (balancedBinaryTree(root.left, leftDepth) && balancedBinaryTree(root.right, rightDepth)) {
            if (Math.abs(leftDepth.get() - rightDepth.get()) <= 1) {
                depth.set(Math.max(leftDepth.get() , rightDepth.get()) + 1);
                return true;
            }
        }
        return false;
    }

    private static class Holder<T> {
        private T t;

        public static <T> Holder<T> of(T t) {
            Holder<T> holder = new Holder<>();
            holder.set(t);
            return holder;
        }

        public void set(T t) {
            this.t = t;
        }

        private T get() {
            return t;
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
    }
}
```
