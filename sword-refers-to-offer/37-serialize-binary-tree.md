---
layout: post
title: "序列化二叉树"
date: "2019-07-02"
description: "序列化二叉树"
tag: [algorithm]
---

### 序列化二叉树

#### 题目
请实现两个函数, 分别用来序列化和反序列化二叉树

##### 思路
从根节点开始使用前序遍历访问节点的顺序来序列化二叉树, 在遍历二叉树碰到 `null` 指针时, 这些 `null` 指针序列化为一个特殊的字符 (如 `$`; 根据这样的序列化规则, 以下图中的二叉树被序列化为字符串 `1, 2, 4, $, $ $, 3, 5, $, $, 6, $, $`
```
   --- 1 ---
   |        |      
 - 2     - 3 -   
 |       |    |    
 4       5    6
```    

##### 实现
```
import java.util.ArrayList;
import java.util.List;

public class SerializeBinaryTree {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(1)
                .setLeft(new BinaryTreeNode(2)
                        .setLeft(new BinaryTreeNode(4)))
                .setRight(new BinaryTreeNode(3)
                        .setLeft(new BinaryTreeNode(5))
                        .setRight(new BinaryTreeNode(6)));
        System.out.println(serializeBinaryTree(root));
    }

    private static List<String> serializeBinaryTree(BinaryTreeNode root) {
        List<String> list = new ArrayList<>();
        if (root == null) {
            list.add("$");
            return list;
        }

        list.add(String.valueOf(root.value));
        list.addAll(serializeBinaryTree(root.left));
        list.addAll(serializeBinaryTree(root.right));
        return list;
    }

    private static BinaryTreeNode deserializeBinaryTree(List<String> list) {
        if (list == null || list.isEmpty()) {
            return null;
        }

        String value = list.remove(0);
        if ("$".equals(value)) {
            return null;
        }

        BinaryTreeNode root = new BinaryTreeNode(Integer.parseInt(value));
        root.setLeft(deserializeBinaryTree(list));
        root.setRight(deserializeBinaryTree(list));
        return root;
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
