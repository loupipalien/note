---
layout: post
title: "合并两个排序的链表"
date: "2019-06-20"
description: "合并两个排序的链表"
tag: [algorithm]
---

### 合并两个排序的链表

#### 题目一
输入两个递增排序的链表, 合并这两个链表并使新链表中的节点仍然是递增排序的; 例如, 输入以下的链表 1 和链表 2, 则合并之后的升序链表如链表 3 所示; 链表节点定义如下
```
class ListNode {
    int value;
    ListNode next = null;

    ListNode(int value) {
        this.value = value;
    }
}
```

##### 思路
从两个链表的头节点开始, 如有一个链表是 null, 则直接返回另一个链表的头节点; 如果两个链表都不为 null, 则开始比较两个节点的大小, 较小的节点赋值给到合并链表的 next, 然后自身再赋值为自己 next 节点, 循环以上过程, 直到有链表到达尾节点; 将未遍历完的节点赋值给合并链表的 next, 然后返回合并链表的头节点;

##### 实现
###### 非递归实现
```
public class MergeSortedLists {
    public static void main(String[] args) {
        ListNode head1 = new ListNode(1);
        ListNode current1 = head1;
        for (int i = 2; i < 20; i++) {
            if (i % 2 == 0) {
                current1.next = new ListNode(i);
                current1 = current1.next;
            }
        }
        current1 = head1;
        while (current1 != null) {
            System.out.print(current1.value + " ");
            current1 = current1.next;
        }
        System.out.println();
        ListNode head2 = new ListNode(1);
        ListNode current2 = head2;
        for (int i = 2; i < 20; i++) {
            if (i % 3 == 0) {
                current2.next = new ListNode(i);
                current2 = current2.next;
            }
        }
        current2 = head2;
        while (current2 != null) {
            System.out.print(current2.value + " ");
            current2 = current2.next;
        }
        System.out.println();
        ListNode current = mergeSortedLists(head1, head2);
        while (current != null) {
            System.out.print(current.value + " ");
            current = current.next;
        }
    }

    private static ListNode mergeSortedLists(ListNode head1, ListNode head2) {
        if (head1 == null) {
            return head2;
        }
        if (head2 == null) {
            return head1;
        }

        ListNode head = head1.value < head2.value ? head1 : head2;
        ListNode current = head;
        ListNode current1 = head1 == head ? head1.next : head1;
        ListNode current2 = head2 == head ? head2.next : head2;
        while (current1 != null && current2 != null) {
            if (current1.value <= current2.value) {
                current.next = current1;
                current1 = current1.next;
            } else {
                current.next = current2;
                current2 = current2.next;
            }
            current = current.next;
        }

        // 尾部处理
        if (current1 != null) {
            current.next = current1;
        }
        if (current2 != null) {
            current.next = current2;
        }
        return head;
    }

    private static class ListNode {
        int value;
        ListNode next = null;

        ListNode(int value) {
            this.value = value;
        }
    }
}
```
###### 递归实现
```
public class MergeSortedLists {
    public static void main(String[] args) {
        ListNode head1 = new ListNode(1);
        ListNode current1 = head1;
        for (int i = 2; i < 20; i++) {
            if (i % 2 == 0) {
                current1.next = new ListNode(i);
                current1 = current1.next;
            }
        }
        current1 = head1;
        while (current1 != null) {
            System.out.print(current1.value + " ");
            current1 = current1.next;
        }
        System.out.println();
        ListNode head2 = new ListNode(1);
        ListNode current2 = head2;
        for (int i = 2; i < 20; i++) {
            if (i % 3 == 0) {
                current2.next = new ListNode(i);
                current2 = current2.next;
            }
        }
        current2 = head2;
        while (current2 != null) {
            System.out.print(current2.value + " ");
            current2 = current2.next;
        }
        System.out.println();
        ListNode current = mergeSortedLists(head1, head2);
        while (current != null) {
            System.out.print(current.value + " ");
            current = current.next;
        }
    }

    private static ListNode mergeSortedLists(ListNode head1, ListNode head2) {
        if (head1 == null) {
            return head2;
        }
        if (head2 == null) {
            return head1;
        }

        ListNode head;
        if (head1.value < head2.value) {
            head = head1;
            head.next = mergeSortedLists(head1.next, head2);
        } else {
            head = head2;
            head.next = mergeSortedLists(head1, head2.next);
        }
        return head;
    }

    private static class ListNode {
        int value;
        ListNode next = null;

        ListNode(int value) {
            this.value = value;
        }
    }
}
```
递归实现更加简洁, 时间复杂度也与非递归实现相同, 在不考虑方法栈内存溢出的情况下, 推荐实现递归实现
