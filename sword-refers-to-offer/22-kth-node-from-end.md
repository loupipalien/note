---
layout: post
title: "链表中倒数第 K 个节点"
date: "2019-06-17"
description: "链表中倒数第 K 个节点"
tag: [algorithm]
---

### 链表中倒数第 K 个节点

#### 题目
输入一个链表, 输出该链表中倒数第 k 个节点, 为了符合大多数人的习惯, 本题从 1 开始计数, 即链表的尾节点是倒数第 1 个节点; 例如, 一个链表有 6 个节点, 从头节点开始, 它们的值依次是 1, 2, 3, 4, 5, 6; 这个链表的倒数第 3 个节点是值为 4 的节点; 链表节点定义如下:
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
由于链表是单向链表, 不能从尾到头遍历链表, 那么只能从头到尾遍历; 链表的倒数第 k 个节点, 其实也是链表的第 n - k + 1 个节点; 为了输出第 n - k + 1 节点, 需要先遍历链表得到 n 的值; 这样的实现需要遍历两次链表  
为了避免遍历两次链表, 这里定义两个指针 (ahead, behind), 让两个指针保持 k - 1 个节点的距离, 当 ahead 指针到达链表尾节点时, behind 指向的就是链表倒数第 k 个节点

##### 实现
```
public class KthNodeFromEnd {
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
        System.out.println("\n" + kthNodeFromEnd(head, 5).value);
    }

    private static ListNode kthNodeFromEnd(ListNode head,int k) {
        if (head == null || k < 1) {
            return null;
        }
        ListNode ahead = head;
        ListNode behind = null;
        for (int i = 0; i < k - 1; i++) {
            if (ahead.next != null) {
                ahead = ahead.next;
            } else {
                return behind;
            }
        }

        // 有足够元素
        behind = head;
        while (ahead.next != null) {
            ahead = ahead.next;
            behind = behind.next;
        }
        return behind;
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

#### 相关
TODO
