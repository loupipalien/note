---
layout: post
title: "链表中环的入口节点"
date: "2019-06-18"
description: "链表中环的入口节点"
tag: [algorithm]
---

### 链表中环的入口节点

#### 题目一
如果一个链表中包含环, 如何找出环的入口节点? 例如, 在下图所示的链表中, 环的入口节点是节点 3
```
          -------------------
          v                 |
1 -> 2 -> 3 -> 4 -> -> 5 -> 6
```
其中链表节点定义如下
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
解决这个问题的第一步是要确定这个链表中是否有环; 这里可以使用两个指针来解决这个问题, 定义两个指针 P1, P2 都从头节点出发, P1 一次走一步, P2 一次走两步; 如果 P2 追上了 P1, 那么说明链表就有环; 如果 P2 到达了链表末尾都没有追上 P1, 那么说明链表没有环  
第二步是如何找到链表中环的入口点; 这里仍然是使用两个指针来解决这个问题; 也是定义两个指针 P1, P2 先指向链表的头节点, P1 指针先走 n 步 (这里的 n 是环包含的节点数), 然后两个指针再同时出发以相同的速度向前移动, 当两个指针相遇时便是环的入口点  
其中环节点的个数 n 可以在第一步中的统计, 由于 P2 比 P1 多走了 N 个环, 那么 P1 走的步数就是环节点的个数

##### 实现
```Java
public class EntryNodeOfLoop {
    public static void main(String[] args) {
        ListNode head = new ListNode(1);
        ListNode entry = null;
        ListNode current = head;
        for (int i = 2; i < 7; i++) {
            current.next = new ListNode(i);
            current = current.next;
            if (i == 3) {
                entry = current;
            }
        }
        current.next = entry;
        // find
        System.out.println(entryNodeOfLoop(head).value);
    }

    private static ListNode entryNodeOfLoop(ListNode head) {
        int number = numberOfNodeInLoop(head);

        ListNode ahead = head;
        ListNode behind = null;
        if (number > 0) {
            while (number-- > 0) {
                ahead = ahead.next;
            }
            behind = head;
            while (ahead != behind) {
                ahead = ahead.next;
                behind = behind.next;
            }
        }
        return behind;
    }

    private static int numberOfNodeInLoop(ListNode head) {
        if (head == null) {
            return 0;
        }

        int number = 0;
        ListNode slow = head;
        ListNode fast = head;
        while (fast.next != null && fast.next.next != null) {
            fast = fast.next.next;
            slow = slow.next;
            number++;
            if (fast == slow) {
                return number;
            }
        }
        return 0;
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
