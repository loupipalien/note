---
layout: post
title: "二叉树的下一个节点"
date: "2019-06-04"
description: "二叉树的下一个节点"
tag: [algorithm]
---

### 二叉树的下一个节点

#### 题目一
给定一颗二叉树和其中的一个节点, 如何找出中序遍历序列的下一个节点? 树中的节点除了有两个分别指向左右子节点的指针, 还有一个指向父节点的指针; 二叉树节点定义如下
```
class BinaryTreeNode {
    int val;
    BinaryTreeNode left = null;
    BinaryTreeNode right = null;
    BinaryTreeNode parent = null;

    BinaryTreeNode(int x) { val = x; }
}
```

##### 思路
- 如果一个节点有右子树, 那么它的下一个节点是它的右子树中的最左的子节点
- 如果一个节点没有右子树, 并且它是父节点的左孩子, 那么它的下一个节点是它的父节点
- 如果一个节点没有右子树, 并且它是父节点的右孩子, 那么它的下一个节点是它祖先节点中离它最近的, 并且为其祖先的左孩子节点的节点, 如果没有这样的节点那么它就是最后一个节点

##### 实现
```
public class NextNodeInBinaryTrees {
    public static void main(String[] args) {
        // inorder: 4(d), 2(b), 8(g), 5(e), 9(i), 1(a), 6(f), 3(c), 7(h)
        BinaryTreeNode aNode = new BinaryTreeNode(1);
        BinaryTreeNode bNode = new BinaryTreeNode(2);
        BinaryTreeNode cNode = new BinaryTreeNode(3);
        BinaryTreeNode dNode = new BinaryTreeNode(4);
        BinaryTreeNode eNode = new BinaryTreeNode(5);
        BinaryTreeNode fNode = new BinaryTreeNode(6);
        BinaryTreeNode gNode = new BinaryTreeNode(7);
        BinaryTreeNode hNode = new BinaryTreeNode(8);
        BinaryTreeNode iNode = new BinaryTreeNode(9);
        aNode.left = bNode; bNode.parent = aNode;
        aNode.right = cNode; cNode.parent = aNode;
        bNode.left = dNode; dNode.parent = bNode;
        bNode.right = eNode; eNode.parent = bNode;
        cNode.left = fNode; fNode.parent = cNode;
        cNode.right = gNode; gNode.parent = cNode;
        eNode.left = hNode; hNode.parent = eNode;
        eNode.right = iNode; iNode.parent = eNode;

        System.out.println(nextNodeInBinaryTrees(aNode).val);
        System.out.println(nextNodeInBinaryTrees(bNode).val);
        System.out.println(nextNodeInBinaryTrees(dNode).val);
        System.out.println(nextNodeInBinaryTrees(fNode).val);
        System.out.println(nextNodeInBinaryTrees(gNode));
        System.out.println(nextNodeInBinaryTrees(iNode).val);
    }

    private static BinaryTreeNode nextNodeInBinaryTrees(BinaryTreeNode node) {
        if (node == null) {
            return null;
        }

        BinaryTreeNode next;
        if (node.right != null) {
            next = node.right;
            while (next.left != null) {
                next = next.left;
            }
        } else if (node.parent != null && node.parent.right == node) {
            next = node.parent;
            while (next.parent != null && next.parent.left != next) {
                next = next.parent;
            }
            next = next.parent;
        } else {
            next = node.parent;
        }
        return next;
    }

    private static class BinaryTreeNode {
        int val;
        BinaryTreeNode left = null;
        BinaryTreeNode right = null;
        BinaryTreeNode parent = null;

        BinaryTreeNode(int x) { val = x; }
    }
}
```
这个方法的时间复杂度为 $ O(n) $
