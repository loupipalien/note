---
layout: post
title: "复杂链表的复制"
date: "2019-06-30"
description: "复杂链表的复制"
tag: [algorithm]
---

### 复杂链表的复制

#### 题目
请实现函数 `ComplexListNode clone(ComplexListNode head)`, 复制一个复杂链表; 在复杂链表中, 每个节点除了有一个 `next` 指针指向下一个节点, 还有一个 `slibing` 指针指向链表中的任意节点或者 `null`; 节点定义如下
```
class ComplexListNode {
    int value;
    ComplexListNode next;
    ComplexListNode sibling;
}
```

##### 思路
蛮力的解法是将复制过程分成两步
- 第一步是复制原始链表撒谎功能的每个节点, 并用 `next` 指针连接起来
- 第二步是设置每个节点的 `slibing` 指针
这里的第二步, 由于要先找到每个节点 `slibing` 指针指向节点的时间复杂度为 $O(n)$, 整个链表的时间复杂度为 $O(n^2)$  

为了减少查找 `slibing` 指针指向节点的时间复杂度, 可以使用一个 Map 将每个节点与其复制节点的映射保存起来, 在为复制链表的节点设置 `slibing` 节点时, 只需要 $O(1)$ 的时间就可找到应该指向的节点; 这是一个使用空间换时间的方法  
这里有一种方法可以在不使用辅助空间的情况下实现 $O(n)$ 的时间效率:
- 首先仍然是根据原始链表中的每个节点创建复制节点, 但这次将复制节点连接在其原始节点之后
- 然后在根据原始节点的 `slibing` 的指向设置复制节点的 `slibing` 的指向, 这次设置时只需要找到原始节点的 `next` 节点即可, 只需要 $O(1)$ 的时间
- 最后将链表按奇偶拆成两个链表, 其中一个为原始链表, 另一个为复制链表

##### 实现
```Java
public class Solution {
    public static void main(String[] args) {
        // {1,2,3,4,5,3,5,#,2,#}
        ComplexListNode node1 = new ComplexListNode(1);
        ComplexListNode node2 = new ComplexListNode(2);
        ComplexListNode node3 = new ComplexListNode(3);
        ComplexListNode node4 = new ComplexListNode(4);
        ComplexListNode node5 = new ComplexListNode(5);
        node1.setNext(node2).setSibling(node3);
        node2.setNext(node3).setSibling(node5);
        node3.setNext(node4);
        node4.setNext(node5).setSibling(node2);

        copyComplexList(node1);
    }

    private static ComplexListNode copyComplexList(ComplexListNode head) {
        cloneNodes(head);
        connectSiblingNodes(head);
        return reconnectNodes(head);
    }

    private static void cloneNodes(ComplexListNode head) {
        ComplexListNode current = head;
        while (current != null) {
            ComplexListNode cloned = new ComplexListNode(current.value);
            cloned.next = current.next;
            current.next = cloned;
            current = cloned.next;
        }
    }

    private static void connectSiblingNodes(ComplexListNode head) {
        ComplexListNode current = head;
        while (current != null) {
            ComplexListNode cloned = current.next;
            if (current.sibling != null) {
                cloned.sibling = current.sibling.next;
            }
            current = cloned.next;
        }
    }

    private static ComplexListNode reconnectNodes(ComplexListNode head) {
        ComplexListNode clonedHead = head.next;
        // 两个列表的当前节点
        ComplexListNode current = head;
        ComplexListNode clonedCurrent = current.next;
        while (current != null) {
            // 为了使原链表保持原样
            current.next = clonedCurrent.next;
            current = current.next;
            if (current != null) {
                clonedCurrent.next = current.next;
                clonedCurrent = clonedCurrent.next;
            }
        }
        return clonedHead;
    }

    private static class ComplexListNode {
        int value;
        ComplexListNode next;
        ComplexListNode sibling;

        public ComplexListNode(int value) {
            this.value = value;
        }

        public ComplexListNode setNext(ComplexListNode next) {
            this.next = next;
            return this;
        }

        public ComplexListNode setSibling(ComplexListNode sibling) {
            this.sibling = sibling;
            return this;
        }
    }
}
```
