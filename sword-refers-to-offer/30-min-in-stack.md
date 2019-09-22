---
layout: post
title: "包含 min 函数的栈"
date: "2019-06-25"
description: "包含 min 函数的栈"
tag: [algorithm]
---

### 包含 min 函数的栈

#### 题目一
定义栈的数据结构, 请在该类型中实现一个能够得到栈的最小元素的 min 函数; 在该栈中, 调用 min, push, pop 的时间复杂度都是 $O(1)$

##### 思路
这里可以借助一个辅助栈, 这个辅助栈中的元素保存的是对应栈的元素之前的最小值; 如何做到这一点呢, 即在往栈中压入元素时, 比较当前元素与辅助栈栈顶元素的大小, 如果当前元素更小则在辅助栈中也压入该元素, 否则应该再往辅助栈中压入辅助栈栈顶元素, 弹出栈元素时同时也弹出辅助栈栈顶元素; 以下是一个实例示意图

| 步骤 | 操作 | 数据栈 | 辅助栈 | 最小值 |
| :--- | :--- | :--- | :--- | :--- |
| 1 | push(3) | 3 | 3 | 3 |
| 2 | push(4) | 3, 4 | 3, 3 | 3 |
| 3 | push(2) | 3, 4, 2 | 3, 3, 2 | 2 |
| 4 | push(1) | 3, 4, 2, 1 | 3, 3, 2, 1 | 1 |
| 5 | pop() | 3, 4, 2 | 3, 3, 2 | 2 |
| 6 | pop() | 3, 4 | 3, 3 | 3 |
| 7 | push(0) | 3, 4, 0 | 3, 3, 0 | 0 |

##### 实现
```
import java.util.Stack;

public class MinInStack {
    public static void main(String[] args) {
        MinInStack minInStack = new MinInStack();
        minInStack.push(3); System.out.println(minInStack.min());
        minInStack.push(4); System.out.println(minInStack.min());
        minInStack.push(2); System.out.println(minInStack.min());
        minInStack.push(1); System.out.println(minInStack.min());
        minInStack.pop(); System.out.println(minInStack.min());
        minInStack.pop(); System.out.println(minInStack.min());
        minInStack.push(0); System.out.println(minInStack.min());
    }

    private Stack<Integer> stack = new Stack();
    private Stack<Integer> minStack = new Stack();

    private void push(int value) {
        stack.push(value);
        if (minStack.empty()) {
            minStack.push(value);
        } else {
            minStack.push(minStack.peek() > value ? value : minStack.peek());
        }
    }

    private void pop() {
        stack.pop();
        minStack.pop();
    }

    private int top() {
        return stack.peek();
    }

    private int min() {
        return minStack.peek();
    }
}
```
