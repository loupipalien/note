---
layout: post
title: "删除链表的节点"
date: "2019-06-13"
description: "删除链表的节点"
tag: [algorithm]
---

### 删除链表的节点

#### 题目一
在 $O(1)$ 时间内删除链表节点
给定单向链表的头指针和一个节点指针, 定义一个函数在 $O(1)$ 时间内删除该节点; 链表节点与函数的定义如下
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
在单向链表中删除一个节点, 常规的做法无疑是从链表的头节点开始, 顺序遍历查找要删除的节点, 并在链表中删除该节点; 这种操作的时间复杂度自然就是 $O(n)$; 之所以需要从头开始查找, 是因为我们需要得到将被删除的节点的前一个节点, 在单向链表中的节点没有一个指向前一个节点的指针, 所以只好从链表的头节点开始顺序查找  
为了在 $O(1)$ 的时间内删除当前节点, 其实可以将该节点的 next 节点内容覆盖当前节点, 这样就相当于删除了当前节点; 但要考虑特殊情况, 例如
- 要删除的节点位于链表的尾部, 没有 next 节点, 那么只有顺序遍历完成删除操作
- 如果链表只有一个节点, 那么要被删除节点既是头节点也是尾节点, 所以需要将头节点置为 null

##### 实现
```
public class DeleteNodeInList {
    public static void main(String[] args) {
        ListNode headNode = new ListNode(0);
        ListNode currentNode = headNode;
        for (int i = 1; i < 6; i++) {
            currentNode.next = new ListNode(i);
            currentNode = currentNode.next;
        }
        ListNode deletedNode = new ListNode(999);
        currentNode.next = deletedNode;
        currentNode = currentNode.next;
        for (int i = 6; i < 12; i++) {
            currentNode.next = new ListNode(i);
            currentNode = currentNode.next;
        }

        printListNode(headNode);
        deleteNodeInList(headNode, deletedNode);
        printListNode(headNode);
    }

    private static ListNode deleteNodeInList(ListNode head, ListNode needDeleted) {
        if (head == null || needDeleted == null) {
            return head;
        }

        // 要删除的节点不是尾节点
        if (needDeleted.next != null) {
            ListNode nextNode = needDeleted.next;
            needDeleted.value = nextNode.value;
            needDeleted.next = nextNode.next;
        } else {
            if (head != needDeleted) {
                ListNode preNeedDeleted = head;
                while (preNeedDeleted.next != needDeleted) {
                    preNeedDeleted = preNeedDeleted.next;
                }
                preNeedDeleted.next = null;
            } else {
                head = null;
            }
        }
        return head;
    }

    private static void printListNode(ListNode node) {
        ListNode currentNode = node;
        while (currentNode != null) {
            if (currentNode.next != null) {
                System.out.print(currentNode.value + " -> ");
            } else {
                System.out.print(currentNode.value + "\n");
            }
            currentNode = currentNode.next;
        }
    }

    private static class ListNode {
        int value;
        ListNode next;

        ListNode(int value) {
            this.value = value;
        }
    }
}
```
以上算法对于 n - 1 个非尾节点而言, 删除节点操作时间复杂度为 $O(1)$, 对于尾节点为 $O(n)$; 因此总的时间复杂度为 $((n - 1) * O(1) + O(n)) / n = O(1)$, 符合题目的要求

#### 题目二
删除链表中重复的节点  
在一个排序的链表中, 如何删除重复的节点? 例如: 在图 (a) 中重复的节点被删除后, 链表如图 (b) 所示
```
a: 1 -> 2 -> 3 -> 3 -> 4 -> 4 -> 5
b: 1 -> 2 -> 5
```

##### 思路
从头遍历整个链表, 如果不同则保存当前节点 (current) 为当前节点的前一个节点 (preCurrent, 也就是当前节点前最后一个非重复节点); 如果当前节点 (current) 的值和下一个节点 (next) 值相同, 那么可以认定有重复节点需要删除, 则删除重复节点并将 current 指定为重复节点后的第一个节点, 并重复以上过程直到遍历结束

##### 实现
```
public class DeleteDuplicatedNode {
    public static void main(String[] args) {
        ListNode headNode = new ListNode(1);
        ListNode currentNode = headNode;
        for (int i = 2; i < 6; i++) {
            currentNode.next = new ListNode(i);
            currentNode = currentNode.next;
            if (i % 3 == 0 || i % 4 == 0) {
                currentNode.next = new ListNode(i);
                currentNode = currentNode.next;
            }
        }

        printListNode(headNode);
        printListNode(deleteDuplicatedNode(headNode));
    }

    private static ListNode deleteDuplicatedNode(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        ListNode current = head;
        ListNode preCurrent = null;
        while (current != null) {
            ListNode next = current.next;
            boolean isDuplicated = false;
            if (next != null && next.value == current.value) {
                isDuplicated = true;
            }

            if (!isDuplicated) {
                preCurrent = current;
                current = next;
            } else {
                // 删除 current 以后的重复节点
                do {
                    current.next = next.next;
                    next = current.next;
                } while (next != null && next.value == current.value);
                // 删除重复的 current 节点, 并重指定 current
                if (preCurrent == null) {
                    head = next;
                } else {
                    preCurrent.next = next;
                }
                current = next;
            }
        }
        return head;
    }

    private static void printListNode(ListNode node) {
        ListNode currentNode = node;
        while (currentNode != null) {
            if (currentNode.next != null) {
                System.out.print(currentNode.value + " -> ");
            } else {
                System.out.print(currentNode.value + "\n");
            }
            currentNode = currentNode.next;
        }
    }

    private static class ListNode {
        int value;
        ListNode next;

        ListNode(int value) {
            this.value = value;
        }
    }
}
```
