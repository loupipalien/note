---
layout: post
title: "滑动窗口的最大值"
date: "2019-07-24"
description: "滑动窗口的最大值"
tag: [algorithm]
---

### 滑动窗口的最大值

#### 题目一: 滑动窗口的最大值
给定一个数组和滑动窗口的大小, 请找出所有滑动窗口里的最大值; 例如: 如果输入数组 `{2, 3, 4, 2, 6, 2, 5, 1}` 及滑动窗口的大小 3, 那么一共存在 6 个滑动窗口, 它们的最大值分别为 `{4, 4, 5, 5, 6, 6}`, 如下表所示

| 数组中的滑动窗口 | 滑动窗口中的最大值 |
| :--- | :--- |
| [2, 3, 4], 2, 6, 2, 5, 1 | 4 |
| 2, [3, 4, 2], 6, 2, 5, 1 | 4 |
| 2, 3, [4, 2, 6], 2, 5, 1 | 6 |
| 2, 3, 4, [2, 6, 2], 5, 1 | 6 |
| 2, 3, 4, 2, [6, 2, 5], 1 | 6 |
| 2, 3, 4, 2, 6, [2, 5, 1] | 5 |

##### 思路
维护一个两端开口的队列, 只将有可能成为滑动窗口的最大值的数组下标存入队列; 以上述数组为例:  
数组中第一个数字是 2, 把它存入队列, 第二个数字是 3, 由于它比前一个数字 2 大, 因此 2 不可能成为滑动窗口中的最大值, 所以把 2 从队列中删除; 接下来的数字 4 也是一样的操作, 此时滑动窗口中已经有 3 个数字, 而它的最大值 4 位于队列的队首  
接下来处理第四个数字 2, 虽然 2 比 4 小, 但当 4 滑出窗口时, 2 也可能成为最大值, 所以将 2 放入队列尾部, 此时的最大值 4 在队首; 第 5 个数字是 6, 那么 4 和 2 都不可能成为最大值了, 那么把 4 和 2 从队首移出, 将 6 存入队列的尾部, 此时的最大值仍然在队首  
接下来第六个数字 2, 同上存入队尾, 此时最大值 6 仍在队首; 接着是数字 5, 由于 2 已不可能是最大值, 所以移除 2 存入 5, 此时最大值 6 仍在队首  
最后一个数字是 1, 把 1 存入队列的尾部, 此时队列首部的 6 已经出了滑动窗口, 需要将其删除; 如何判断数字是否已经不再滑动窗口中, 即在队列中存放数字的下标而非数字, 当数字下标加减滑动窗口大小仍小于或大于当前数字下标, 那么就不在滑动窗口中  
以下是步骤示意图

| 步骤 | 插入数字 | 滑动窗口 | 队列中的下标 | 最大值 |
| :--- | :--- |:--- | :--- |:--- |
| 1 | 2 | 2 | 0(2) | - |
| 2 | 3 | 2, 3 | 1(3) | - |
| 3 | 4 | 2, 3, 4 | 2(4) | 4 |
| 4 | 2 | 3, 4, 2 | 2(4), 3(2) | 4 |
| 5 | 6 | 4, 2, 6 | 4(6) | 6 |
| 6 | 2 | 2, 6, 2 | 4(6), 5(2) | 6 |
| 7 | 5 | 6, 2, 5 | 4(6), 6(5) | 6 |
| 8 | 1 | 2, 5, 1 | 6(5), 7(1) | 5 |

##### 实现
```java
import java.util.*;

public class Solution {
    public static void main(String[] args) {
        int[] array = new int[]{2, 3, 4, 2, 6, 2, 5, 1};
        System.out.println(maxInSlidingWIndow(array, 3));
    }

    private static List<Integer> maxInSlidingWIndow(int[] array, int size) {
        List<Integer> maximums = new ArrayList<>();
        if (array == null || size < 1 || array.length < size) {
            return maximums;
        }

        Deque<Integer> deque = new LinkedList<>();
        for (int i = 0; i < array.length; i++) {
            if (size > i + 1) {
                // 尚未够滑动窗口大小时
                maintainPossibleMaxInWindow(deque, i, array, size);
            } else {
                maintainPossibleMaxInWindow(deque, i, array, size);
                maximums.add(array[deque.peek()]);
            }
        }
        return maximums;
    }

    private static void maintainPossibleMaxInWindow(Deque<Integer> deque, int index, int[] array, int size) {
        while (!deque.isEmpty() && array[deque.peekLast()] < array[index]) {
            deque.pollLast();
        }
        if (!deque.isEmpty() && deque.peekFirst() + size <= index) {
            deque.pollFirst();
        }
        deque.offerLast(index);
    }
}
```

#### 题目二: 队列的最大值
请定义一个队列并实现函数 max 得到队列里的最大值, 要求函数 max, push_back, pop_front 的时间复杂度都是 $O(1)$

##### 思路
TODO

##### 实现
TODO
