### 构造数组的 MaxTree

#### 题目
定义二叉树的节点如下:
```java
public class Node {
    public int value;
    public Node left;
    public Node right;

    public Node(int value) {
        this.value = value;
    }
}
```
一个数组的 MaxTree 定义如下
- 数组必须没有重复元素
- MxaTree 是一棵二叉树, 数组的每一个值对应一个二叉树节点
- 包括 MaxTree 树在内且在其中的每一棵子树上, 值最大的节点都是树的根节点

给定一个没有重复元素的数组 array, 写出生成这个数组的 MaxTree 的函数, 要求如果数组长度为 N, 则时间复杂度为 O(N), 额外空间复杂度为 O(N)

#### 难度
:star::star::star:

#### 思路
