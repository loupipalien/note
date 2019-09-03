---
layout: post
title: "重建二叉树"
date: "2019-06-03"
description: "重建二叉树"
tag: [algorithm]
---

### 重建二叉树

#### 题目一
输入某二叉树的前序遍历和中序遍历的结果, 请重建该二叉树; 假设输入的前序遍历和中序遍历的结果都不含重复的数字; 例如, 输入前序遍历序列 `{1, 2, 3, 7, 3, 5, 6, 8}` 和中序遍历序列 `{4, 7, 2, 1, 5, 3, 8, 6}`, 重建二叉树并输出它的头节点; 二叉树节点的定义如下
```
class BinaryTreeNode {
    int val;
    TreeNode left;
    TreeNode right;
    TreeNode(int x) { val = x; }
}
```

##### 思路
在二叉树的前序遍历序列中, 第一个数字总是树的根节点的值, 后续依次是左子树节点的值, 右子树节点的值; 在中序遍历序列中, 根节点的值在中间, 左子树节点的值在根节点值的左边, 右子树节点的值在根节点值的右边; 找到根节点后, 同样也找到了左子树和右子树的前序遍历序列和中序遍历序列, 则可以用同样的方法分别构建左, 右子树 (即可以考虑使用递归完成)

##### 实现
```
import java.util.Arrays;

public class ConstructBinaryTree {
    public static void main(String[] args) {
        int[] preorder = new int[]{1, 2, 4, 7, 3, 5, 6, 8};
        int[] inorder = new int[]{4, 7, 2, 1, 5, 3, 8, 6};
        preorderTraversal(constructBinaryTree(preorder, inorder));
        System.out.println();
        inorderTraversal(constructBinaryTree(preorder, inorder));
    }

    private static BinaryTreeNode constructBinaryTree(int[] preorder, int [] inorder) {
        if (preorder == null || inorder == null || preorder.length == 0 || preorder.length != inorder.length) {
            throw null;
        }

        int index = -1;
        for (int i = 0; i < inorder.length; i++) {
            if (preorder[0] == inorder[i]) {
                index = i;
                break;
            }
        }
        if (index == -1) {
            throw new IllegalArgumentException("Invalid parameters.");
        }

        BinaryTreeNode node = new BinaryTreeNode(preorder[0]);
        // left child tree
        if (index > 0) {
            node.left = constructBinaryTree(Arrays.copyOfRange(preorder, 1, index + 1),
                    Arrays.copyOfRange(inorder, 0, index));
        }
        // right child tree
        if (index < inorder.length - 1) {
            node.right = constructBinaryTree(Arrays.copyOfRange(preorder, index + 1, preorder.length),
                    Arrays.copyOfRange(inorder, index + 1, inorder.length));
        }
        return node;
    }

    private static class BinaryTreeNode {
        int val;
        BinaryTreeNode left;
        BinaryTreeNode right;

        BinaryTreeNode(int x) {
            val = x;
        }
    }

    private static void preorderTraversal(BinaryTreeNode node) {
        if (node != null) {
            System.out.println(node.val + " ");
            preorderTraversal(node.left);
            preorderTraversal(node.right);
        }
    }

    private static void inorderTraversal(BinaryTreeNode node) {
        if (node != null) {
            inorderTraversal(node.left);
            System.out.println(node.val + " ");
            inorderTraversal(node.right);
        }
    }
}
```
这个方法的时间复杂度为 $ O(n) $; 但需要注意的是, 线程方法栈的内存空间有限, 当递归调用层级过深会导致栈溢出
