---
layout: post
title: "从尾到头打印链表"
date: "2019-06-02"
description: "从尾到头打印链表"
tag: [algorithm]
---

### 从尾到头打印链表

#### 题目一
输入一个链表的头节点, 从尾到头反向打印出每个节点的值; 链表节点定义如下
```
class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}
```

##### 思路
如果是从头到尾打印链表, 那么遍历链表即可; 但从尾到头打印 (单向) 链表, 则要先找到最后一个节点, 并能反向找到后续节点; 通常的解决方法是先遍历一次, 将节点存放在栈中, 再输出打印即可; 另外一种方法则是利用递归调用 (因为递归本质上就是一个栈 (只不过是线程方法栈)), 这样则可以不使用辅助空间

##### 实现
```
public class PrintListInReversedOrder {
    public static void main(String[] args) {
        ListNode head = new ListNode(0);
        ListNode node = head;
        for (int i = 1; i < 10; i++) {
            node.next = new ListNode(i);
            node = node.next;
        }

        printListInReversedOrder(head);
    }

    private static void printListInReversedOrder(ListNode listNode) {
        if (listNode.next != null) {
            printListInReversedOrder(listNode.next);
        }
        System.out.println(listNode.val);
    }

    private static class ListNode {
        int val;
        ListNode next = null;

        ListNode(int val) {
            this.val = val;
        }
    }
}
```
这个方法的时间复杂度为 $ O(n) $; 但需要注意的是, 线程方法栈的内存空间有限, 当递归调用层级过深会导致栈溢出
