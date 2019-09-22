---
layout: post
title: "栈的压入弹出序列"
date: "2019-06-26"
description: "栈的压入弹出序列"
tag: [algorithm]
---

### 栈的压入弹出序列

#### 题目一
输入两个整数序列, 第一个序列表示栈的压入顺序, 请判断第二个序列是否为该栈的弹出顺序; 假设压入栈的所有数字均不相等; 例如, 序列 `{1, 2, 3, 4, 5}` 是某栈的压栈序列, 序列 `{4, 5, 3, 2, 1}` 是压栈序列对应的一个弹出序列, 但 `{4, 3, 5, 1, 2}` 就不可能是该栈序列的弹出序列

##### 思路
最直观的思路就是建立一个辅助栈, 把输入的第一个序列中的数字依次压入该辅助栈中, 并按照第二个序列的顺序依次从该栈中弹出数字  
正确序列: `{1, 2, 3, 4, 5}` 和 `{4, 5, 3, 2, 1}` 作为示例

| 步骤 | 操作 | 栈 | 弹出数字 | 步骤 | 操作 | 栈 | 弹出数字 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | push(1) | 1 | - | 6 | push(5) | 1, 2 , 3, 5 | - |
| 2 | push(2) | 1, 2 | - | 7 | pop() | 1, 2, 3 | 5 |
| 3 | push(3) | 1, 2, 3 | - | 8 | pop() | 1, 2 | 3 |
| 4 | push(4) | 1, 2, 3, 4 | - | 9 | pop() | 1 | 2 |
| 5 | pop() | 1, 2, 3 | 4 | 10 | pop() | - | 1 |

错误序列: `{1, 2, 3, 4, 5}` 和 `{4, 3, 5, 1, 2}` 作为示例

| 步骤 | 操作 | 栈 | 弹出数字 | 步骤 | 操作 | 栈 | 弹出数字 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 1 | push(1) | 1 | - | 6 | pop() | 1, 2 | 3 |
| 2 | push(2) | 1, 2 | - | 7 | push(5) | 1, 2, 5 | - |
| 3 | push(3) | 1, 2, 3 | - | 8 | pop() | 1, 2 | 5 |
| 4 | push(4) | 1, 2, 3, 4 | - | - | - | - | - |
| 5 | pop() | 1, 2, 3 | 4 | - | - | - | - |

这里出错是因为下一个弹出的是 1, 但 1 不在栈顶, 压栈序列的数字都已入栈; 操作无法继续

##### 实现
```
import java.util.Stack;

public class IsPopOrder {
    public static void main(String[] args) {
        int[] push = new int[]{1, 2, 3, 4, 5};
        int[] pop = new int[]{4, 5, 3, 2, 1};
        System.out.println(isPopOrder(push, pop));
        pop = new int[]{4, 3, 5, 1, 2};
        System.out.println(isPopOrder(push, pop));
    }

    private static boolean isPopOrder(int[] push, int[] pop) {
        if (push == null || pop == null || push.length != pop.length) {
            return false;
        }

        int i = 0, j = 0;
        Stack<Integer> stack = new Stack();
        while (j < pop.length) {
            if (!stack.isEmpty() && stack.peek() == pop[j]) {
                stack.pop();
                j++;
            } else {
                if (i < push.length) {
                    stack.push(push[i++]);
                } else {
                    return false;
                }
            }
        }
        return true;
    }
}
```
