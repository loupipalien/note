---
layout: post
title: "二叉搜索树与双向链表"
date: "2019-07-01"
description: "二叉搜索树与双向链表"
tag: [algorithm]
---

### 二叉搜索树与双向链表

#### 题目
输入一颗二叉搜索树, 将该二叉搜索树转换成一个排序的双向链表; 要求不能创建新的节点, 只能调整树中接待你指针的指向; 比如, 输入以下的二叉树数, 则输出转换之后的排序双向链表
```
    --- 10 ---
    |        |      
  - 6 -   - 14 -   =>  4 <-> 6 <-> 8 <-> 10 <-> 12 <-> 14 <-> 16
  |   |   |    |    
  4   8   12   16
```
二叉树节点定义如下
```
class BinaryTreeNode {
    int value;
    BinaryTreeNode left;
    BinaryTreeNode right;
}
```

##### 思路
在二叉树中, 每个节点都有两个指向子节点的指针; 在双向链表中, 每个节点也都有两个指针, 分别指向前一个节点和后一个节点; 由于这两种节点的结构相似, 同时二叉搜索树也是一种排序的数据结构, 因此在理论上是有可能实现二叉搜索树和双向链表的装换  
对二叉搜索树使用中序遍历, 左子树, 根节点, 右子树是双向链表的三个部分, 先将左子树都转换为排序双向链表, 将链表的尾节点和根节点连接起来得到新的双向链表, 再将新双向链表的尾节点与右子树的头节点连接起来, 这样转换就得到了排序双向链表

##### 实现
```Java
public class Solution {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(10)
                .setLeft(new BinaryTreeNode(6)
                        .setLeft(new BinaryTreeNode(4))
                        .setRight(new BinaryTreeNode(8)))
                .setRight(new BinaryTreeNode(14)
                        .setLeft(new BinaryTreeNode(12))
                        .setRight(new BinaryTreeNode(16)));
        BinaryTreeNode head = convertBinarySearchTree(root);
        BinaryTreeNode current = head;
        while (current != null) {
            System.out.print(current.value + " ");
            current = current.right;
        }
    }

    private static BinaryTreeNode convertBinarySearchTree(BinaryTreeNode root) {
        BinaryTreeNode tailInList = convertNode(root, null);
        // 返回头节点
        BinaryTreeNode current = tailInList;
        while (current != null && current.left != null) {
            current = current.left;
        }
        return current;
    }

    private static BinaryTreeNode convertNode(BinaryTreeNode root, BinaryTreeNode tailInList) {
        if (root == null) {
            return root;
        }
        // 左子树
        if (root.left != null) {
            tailInList = convertNode(root.left, tailInList);
        }
        // 根节点
        root.left = tailInList;
        if (tailInList != null) {
            tailInList.right = root;
        }
        tailInList = root;
        // 右子树
        if (root.right != null) {
            tailInList = convertNode(root.right, tailInList);
        }
        return tailInList;
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
    }
}
```
