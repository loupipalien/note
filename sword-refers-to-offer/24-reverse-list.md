---
layout: post
title: "反转链表"
date: "2019-06-19"
description: "反转链表"
tag: [algorithm]
---

### 反转链表

#### 题目一
定义一个函数, 输入一个链表的头节点, 反转该链表并输出反转后链表的头节点; 链表节点定义如下
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
为了反转链表, 这里设置三个指针, `reversedHead, current, next`, 其中 `reversedHead` 指针指向反转链表的头节点, `current` 指针指向遍历的当前节点, `next` 指针指向当前节点的 next 节点; 从头遍历链表, 将遍历的 `current` 节点不为 null 时, 需要将 `current` 节点处理为 `reversedHead` 节点, 为了能继续向后遍历, `next` 保存 `current` 节点的 next, 再 `current` 节点反转后, 将 `next` 节点的值赋给 `current` 节点继续向后遍历; 当前节点为 null 时, 返回 `reversedHead` 节点

##### 实现
```Java
public class ReverseList {
    public static void main(String[] args) {
        ListNode head = new ListNode(0);
        ListNode current = head;
        for (int i = 1; i < 10; i++) {
            current.next = new ListNode(i);
            current = current.next;

        }
        current = head;
        while (current != null) {
            System.out.print(current.value + " ");
            current = current.next;
        }
        System.out.println();
        current = reverseList(head);
        while (current != null) {
            System.out.print(current.value + " ");
            current = current.next;
        }
    }

    private static ListNode reverseList(ListNode head) {
        ListNode current = head;
        ListNode reversedHead = null;
        while (current != null) {
            ListNode next = current.next;
            current.next = reversedHead;
            reversedHead = current;
            current = next;
        }
        return reversedHead;
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
