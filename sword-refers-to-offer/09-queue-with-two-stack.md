---
layout: post
title: "用两个栈实现队列"
date: "2019-06-05"
description: "用两个栈实现队列"
tag: [algorithm]
---

### 用两个栈实现队列

#### 题目一
用两个栈实现一个队列, 队列的声明如下, 请实现它的两个函数 appendTail 和 deleteHead, 分别完成在队列尾部插入节点和在队列头部删除节点的功能; 队列声明如下
```
public class Queue<E> {
    Stack<E> stack1 = new Stack<E>();
    Stack<E> stack2 = new Stack<E>();

    public void push(E element) {}

    public E pop() {}
}
```

##### 思路
栈是先入后出的, 即最先被压入栈的元素会最后弹出; 队列是先入先出的, 即最先进入队列的元素会最先出队列; 使用栈实现队列的操作的难点, 主要在弹出元素时, 如何是最先进入的栈的元素最先出栈  
考虑示例: 有 `a, b, c` 三个元素先入 queue, 此时将三个元素依次压入 stack1 (即元素入队列, 将其压入 stack1 即可); 此时 `a` 出队列, 由于 `a` 元素在 stack1 栈底, 想出队列则需将 `a` 先放置栈顶后弹出; 将 stack1 中的元素依次弹出压入 stack2, 这时栈中的元素被反序, `a` 在 stack2 栈顶弹出即可; 当有新元素进入队列, 则将其压入 stack1, 有元素想出队列则弹出 stack2 栈顶的元素即可, 当 stack2 为空时则先将 stack1 的元素依次弹出压入 stack2, 再弹出 stack2 栈顶元素

##### 实现
```
public class QueueWithTwoStack {
    public static void main(String[] args) {
        Queue<Integer> queue = new Queue<>();
        for (int i = 0; i < 10; i++) {
            queue.push(i);
            if (i % 2 == 0|| i % 3 == 0){
                System.out.println(queue.pop());
            }
        }
        while (!queue.stack1.empty() || !queue.stack2.empty()) {
            System.out.println(queue.pop());
        }
    }

    private static class Queue<E> {
        Stack<E> stack1 = new Stack<>();
        Stack<E> stack2 = new Stack<>();

        public void push(E element) {
            stack1.push(element);
        }

        public E pop() {
            if (stack2.empty()) {
                while (!stack1.empty()) {
                    stack2.push(stack1.pop());
                }
            }
            return stack2.pop();
        }
    }
}
```

#### 题目二
用两个队列实现一个栈

##### 思路
考虑示例: 有 `a, b, c` 三个元素先入 stack, 此时将三个元素依次压入 queue1 (当两个队列都为空时选择 queue1, 否则选择非空队列); 此时 `c` 出栈, 由于 `c` 元素在 queue1 队尾, 想出栈则需将 `c` 先放置队首后弹出; 将 queue1 中的非队尾元素依次弹出压入 queue2, 队尾元素直接弹出; 当有新元素进入栈时, 则将其压入非空队列 (当两个队列都为空时选择 queue1), 栈顶元素要弹出, 则将非空队列中的非队尾元素依次进入空队列
, 范围非空队列中的队尾元素即可

##### 实现
```
public class StackWithTwoQueue {
    public static void main(String[] args) {
        Stack<Integer> stack = new Stack<>();
        for (int i = 0; i < 10; i++) {
            stack.push(i);
        }
        while (!stack.queue1.isEmpty() || !stack.queue2.isEmpty()) {
            System.out.println(queue.pop());
        }
    }

    private static class Stack<E> {
        Queue<E> queue1 = new LinkedList<>();
        Queue<E> queue2 = new LinkedList<>();

        public void push(E element) {
            if (queue2.isEmpty()) {
                queue1.offer(element);
            } else {
                queue2.offer(element);
            }

        }

        public E pop() {
            Queue<E> nonEmptyQueue = !queue1.isEmpty() ? queue1 : queue2;
            Queue<E> emptyQueue = queue1.isEmpty() ? queue1 : queue2;
            while (true) {
                E element = nonEmptyQueue.poll();
                if (nonEmptyQueue.isEmpty()) {
                    return element;
                }
                emptyQueue.offer(element);
            }
        }
    }
}
```
