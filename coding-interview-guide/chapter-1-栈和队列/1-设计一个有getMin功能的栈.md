### 设计一个有 getMin 功能的栈
#### 题目
实现一个特殊的栈, 在实现栈的基本功能的基础上, 再实现返回栈中最小元素的操作

#### 要求
- pop, push, getMin 操作的时间复杂度都是 O(1)
- 设计的栈类型可以使用现成的栈结构

#### 难度
:star:

#### 思路
使用两个栈, 一个栈用来保存当前栈中的元素, 另一个栈用于保存对应的最小值

#### 实现
```java
import java.util.Stack;

public class MinStack {
    private Stack<Integer> data;
    private Stack<Integer> mins;

    public MyStack(Stack<Integer> data, Stack<Integer> mins) {
        this.data = data;
        this.mins = mins;
    }

    public void push(int number) {
        if (mins.isEmpty()) {
            mins.push(number);
        } else if (mins.pop() > number) {
            mins.push(number);
        } else {
            mins.push(mins.peek());
        }
        data.push(number);
    }

    public int pop() {
        if (!mins.isEmpty()) {
            mins.pop();
        }
        return data.pop();
    }

    public int getMin() {
        return mins.peek();
    }
}
```
