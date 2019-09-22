---
layout: post
title: "从上到下打印二叉树"
date: "2019-06-27"
description: "从上到下打印二叉树"
tag: [algorithm]
---

### 从上到下打印二叉树

#### 题目一: 不分行从上到下打印二叉树
从上到下打印出二叉树的每个节点, 同一层的节点按照从左到右的顺序打印; 例如, 输入下图的二叉树
```
   --- 8 ---   
   |       |   
 - 6 -   - 10 -
 |   |   |    |
 5   7   9   11
```
则依次打印出 `8, 6, 10, 5, 7, 9, 11`; 二叉树节点的定义如下
```
class BinaryTreeNode {
    int value = 0;
    BinaryTreeNode left;
    BinaryTreeNode right;

    public BinaryTreeNode(int value) {
        this.value = value;
    }
}
```

##### 思路
维护一个容器, 从根节点开始, 打印一个节点值后, 将其子节点从左到右压入队列中, 完成后从队列中取出节点循环以上过程, 直到队列中没有元素为止

##### 实现
```
mport java.util.LinkedList;
import java.util.Queue;

public class PrintTreeFromTopToBottom {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(8)
                .setLeft(new BinaryTreeNode(6)
                        .setLeft(new BinaryTreeNode(5))
                        .setRight(new BinaryTreeNode(7)))
                .setRight(new BinaryTreeNode(10)
                        .setLeft(new BinaryTreeNode(9))
                        .setRight(new BinaryTreeNode(11)));
        printTreeFromTopToBottom(root);
    }

    private static void printTreeFromTopToBottom(BinaryTreeNode root) {
        if (root == null) {
            return;
        }

        Queue<BinaryTreeNode> queue = new LinkedList() {{add(root);}};
        while (!queue.isEmpty()) {
            BinaryTreeNode node = queue.poll();
            System.out.print(node.value + " ");
            if (node.left != null) {
                queue.add(node.left);
            }
            if (node.right != null) {
                queue.add(node.right);
            }
        }
    }

    private static class BinaryTreeNode {
        int value = 0;
        BinaryTreeNode left;
        BinaryTreeNode right;

        public BinaryTreeNode(int value) {
            this.value = value;
        }

        public BinaryTreeNode setLeft(BinaryTreeNode left) {
            this.left = left;
            return this;
        }

        public BinaryTreeNode setRight(BinaryTreeNode right) {
            this.right = right;
            return this;
        }
    }
}
```

#### 题目二: 分行从上到下打印二叉树
从上到下按层打印二叉树, 同一层的节点按从左到右的顺序打印, 每一层打印到一行; 例如, 打印下图中二叉树的结果
```
   --- 8 ---   
   |       |   
 - 6 -   - 10 -
 |   |   |    |
 5   7   9   11

8
6    10
5    7    9    11
```

##### 思路
仍然使用队列来辅助, 但新增了两个变量, 一个变量记录当前行剩余节点数, 一个变量记录下一行节点数

##### 实现
```
import java.util.LinkedList;
import java.util.Queue;

public class PrintTreeFromInLines {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(8)
                .setLeft(new BinaryTreeNode(6)
                        .setLeft(new BinaryTreeNode(5))
                        .setRight(new BinaryTreeNode(7)))
                .setRight(new BinaryTreeNode(10)
                        .setLeft(new BinaryTreeNode(9))
                        .setRight(new BinaryTreeNode(11)));
        printTreeFromInLines(root);
    }

    private static void printTreeFromInLines(BinaryTreeNode root) {
        if (root == null) {
            return;
        }

        Queue<BinaryTreeNode> queue = new LinkedList() {{add(root);}};
        int current = 1;
        int next = 0;
        while (!queue.isEmpty()) {
            BinaryTreeNode node = queue.poll();
            System.out.print(node.value + " ");
            if (node.left != null) {
                queue.add(node.left);
                next++;
            }
            if (node.right != null) {
                queue.add(node.right);
                next++;
            }
            if (--current == 0) {
                System.out.println();
                current = next;
                next = 0;
            }
        }
    }

    private static class BinaryTreeNode {
        int value = 0;
        BinaryTreeNode left;
        BinaryTreeNode right;

        public BinaryTreeNode(int value) {
            this.value = value;
        }

        public BinaryTreeNode setLeft(BinaryTreeNode left) {
            this.left = left;
            return this;
        }

        public BinaryTreeNode setRight(BinaryTreeNode right) {
            this.right = right;
            return this;
        }
    }
}
```

#### 题目三: 之字形打印二叉树
请实现一个函数按照之字形顺序打印二叉树, 即第一行按照从左到右的顺序打印, 第二层按照从右到左的顺序打印, 第三行再按照从左到右的顺序打印, 其他行以此类推; 例如, 按之字形顺序打印下图中的二叉树的结果为
```
      ------- 1 -------      
      |               |      
  --- 2 ---       --- 3 ---
  |       |       |       |    
- 4 -   - 5 -   - 6 -   - 7 -
|   |   |   |   |   |   |   |
8   9   10  11  12  13  14  15

1
3    2
4    5    6    7
15   14   13   12    11    10    9    8
```

##### 思路
这里使用两个栈 (当然也可以使用其他容器) 来保存当前行的节点和下一行的节点, 可以看到以上二叉树的奇数层是从左向有打印的, 而偶数行是从右向左打印的, 所以在保存时需要注意顺序; 以下是一个示例

| 步骤 | 操作 | stack1 | stack2 |
| :--- | :--- | :--- | :--- |
| 1 |  print(1) | 2, 3 | - |
| 2 |  print(3) | 2 | 7, 6 |
| 3 |  print(2) | - | 7, 6, 5, 4 |
| 4 |  print(4) | 8, 9 | 7, 6, 5 |
| 5 |  print(5) | 8, 9 ,10, 11 | 7, 6 |
| 6 |  print(6) | 8, 9 ,10, 11, 12, 13 | 7 |
| 7 |  print(7) | 8, 9 ,10, 11, 12, 13, 14, 15  | - |

##### 实现
```
import java.util.Stack;

public class Solution {
    public static void main(String[] args) {
        BinaryTreeNode root = new BinaryTreeNode(1)
                .setLeft(new BinaryTreeNode(2)
                        .setLeft(new BinaryTreeNode(4)
                                .setLeft(new BinaryTreeNode(8))
                                .setRight(new BinaryTreeNode(9)))
                        .setRight(new BinaryTreeNode(5)
                                .setLeft(new BinaryTreeNode(10))
                                .setRight(new BinaryTreeNode(11))))
                .setRight(new BinaryTreeNode(3)
                        .setLeft(new BinaryTreeNode(6)
                                .setLeft(new BinaryTreeNode(12))
                                .setRight(new BinaryTreeNode(13)))
                        .setRight(new BinaryTreeNode(7)
                                .setLeft(new BinaryTreeNode(14))
                                .setRight(new BinaryTreeNode(15))));
        printTreeInZigzag(root);
    }

    private static void printTreeInZigzag(BinaryTreeNode root) {
        if (root == null) {
            return;
        }

        Stack<BinaryTreeNode> oddStack = new Stack();
        Stack<BinaryTreeNode> evenStack = new Stack();
        // 奇数行
        boolean isOddLine = true;
        oddStack.push(root);
        while (!oddStack.isEmpty() || !evenStack.isEmpty()) {
            if (isOddLine) {
                // 奇数行节点, 先保存左孩子再保存右孩子
                while (!oddStack.isEmpty()) {
                    BinaryTreeNode node = oddStack.pop();
                    System.out.print(node.value + " ");
                    if (node.left != null) {
                        evenStack.push(node.left);
                    }
                    if (node.right != null) {
                        evenStack.push(node.right);
                    }
                }
                isOddLine = false;
            } else {
                // 偶数行节点, 先保存右孩子再保存左孩子
                while (!evenStack.isEmpty()) {
                    BinaryTreeNode node = evenStack.pop();
                    System.out.print(node.value + " ");
                    if (node.right != null) {
                        oddStack.push(node.right);
                    }
                    if (node.left != null) {
                        oddStack.push(node.left);
                    }
                }
                isOddLine = true;
            }
            System.out.println();
        }
    }

    private static class BinaryTreeNode {
        int value = 0;
        BinaryTreeNode left;
        BinaryTreeNode right;

        public BinaryTreeNode(int value) {
            this.value = value;
        }

        public BinaryTreeNode setLeft(BinaryTreeNode left) {
            this.left = left;
            return this;
        }

        public BinaryTreeNode setRight(BinaryTreeNode right) {
            this.right = right;
            return this;
        }
    }
}
```

#### 扩展
TODO
