---
layout: post
title: "两个链表的第一个公共节点"
date: "2019-07-17"
description: "两个链表的第一个公共节点"
tag: [algorithm]
---

### 两个链表的第一个公共节点

#### 题目
输入两个链表, 找出它们的第一个公共节点; 链表节点定义如下:
```
class ListNode {
    int value;
    ListNode next;

    ListNode(int value) {
        this.value = value;
    }
}
```
##### 思路
最直观的方法是蛮力法, 遍历第一个链表的节点, 然后和第二个链表中的节点比较, 这样的时间复杂度为 $O(n^2)$; 通常蛮力法都是不最好的办法  
两个链表有公共节点, 那么第一个公共节点后的节点都是公共节点, 即两个链表有相同的一段; 如果能从链表尾部开始变量, 遍历到最后一个公共节点那么就是正向遍历的第一个公共节点; 但单向链表不能反向遍历, 为了能反向变量, 可以借助两个栈将链表中的节点依次入栈, 然后反向遍历; 此解法的时间复杂度为 $O(n)$
这里也可以不用借助辅助空间做到时间复杂度为 $O(n)$ 的解法, 假设两个链表非相同节点的部分节点数相同, 那么同时向后遍历, 就可以找到第一个相同的节点; 为了使两个链表 "长度相同", 则计算出差值, 并在长链表上跳过差值个数的节点, 然后再同时向后变量

##### 实现
```
public class Solution {
    public static void main(String[] args) {
        ListNode common = new ListNode(6).setNext(new ListNode(7));
        ListNode head1 = new ListNode(1).setNext(new ListNode(2)
                        .setNext(new ListNode(3).setNext(common)));
        ListNode head2 = new ListNode(4).setNext(new ListNode(5)
                .setNext(common));
        System.out.println(firstCommonNodesInLists(head1, head2));
    }

    private static ListNode firstCommonNodesInLists(ListNode head1, ListNode head2) {
        if (head1 == null || head2 == null) {
            return null;
        }

        ListNode current1 = head1;
        ListNode current2 = head2;
        int length1 = getListLength(head1);
        int length2 = getListLength(head2);
        int lengthDiff = length1 - length2;
        // 将长链表 "缩短" 至短链表的长度
        if (lengthDiff > 0) {
            while (lengthDiff-- > 0) {
                current1 = current1.next;
            }
        } else {
            while (lengthDiff++ < 0) {
                current2 = current2.next;
            }
        }

        // 此时两个链表剩余节点长度相同, 然后开始比较
        while (current1 != current2) {
            current1 = current1.next;
            current2 = current2.next;
        }
        return current1;
    }

    private static int getListLength(ListNode head) {
        int length = 0;
        ListNode current = head;
        while (current != null) {
            length++;
            current = current.next;
        }
        return length;
    }

    private static class ListNode {
        int value;
        ListNode next;

        ListNode(int value) {
            this.value = value;
        }

        public ListNode setNext(ListNode next) {
            this.next = next;
            return this;
        }

        @Override
        public String toString() {
            return "ListNode{" +
                    "value=" + value +
                    ", next=" + next +
                    '}';
        }
    }
}
```
