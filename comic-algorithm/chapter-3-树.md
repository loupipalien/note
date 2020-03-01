### 树

#### 树和二叉树
##### 什么是树
在数据结构中, 树的定义如下  
树 (Tree) 是 n (n >= 0) 个节点的有限集, 当 n = 0 时, 称为空树, 在任意一个非空树中, 有如下特点
- 有且只有一个特定的称为根的节点
- 当 n > 1 时, 其余节点可分为 m (m > 0) 个互不相交的有限集, 每一个结合本身又是一个树, 并称为根的子树

#### 什么是二叉树
二叉树是树的一种特殊形式, 即每个节点最多有两个孩子节点; 二叉树有两种特殊的形式, 一个叫做满二叉树, 另一个叫做完全二叉树
- 满二叉树
一个二叉树的所有非叶子节点都存在左右孩子, 并且所有叶子节点都在同一层级上, 那么这个树就是满二叉树
- 完全二叉树
对一个有 n 个节点的二叉树, 按层级顺序编号, 则所有节点的编号为从 1 到 n; 如果这个树所有节点和同样深度的满二叉树的编号为从 1 到 n 的节点位置相同, 则这个二叉树为完全二叉树

##### 二叉树的应用
二叉树最主要的应用还在于进行查找操作和维持相对顺序两个方面
###### 查找
二叉查找树
- 如果左子树不为空, 则左子树上所有节点的值均小于根节点的值
- 如果右子树不为空, 则右子树上所有节点的值均大于根节点的值
- 左右子树也都是二叉查找树
##### 维持相对顺序
二叉查找树另外一个名字叫二叉排序树

#### 二叉树的遍历
##### 深度优先遍历
- 前序遍历
- 中序遍历
- 后序遍历

##### 广度优先遍历
TODO

#### 什么是二叉堆
二叉堆本质上是一种完全二叉树, 分为两个类型
- 最大堆
最大堆的任何一个父节点的值都大于或等于它左右孩子节点的值
- 最小堆
最小堆的任何一个父节点的值都小于或等于它左右孩子节点的值

##### 二叉堆的代码实现
```Java
public class Solution {

    public static void main(String[] args) {
        int[] array = new int[]{1, 3, 2, 6, 5, 7, 8, 9 , 10, 0};
        upAdjust(array);
        System.out.println(Arrays.toString(array));

        array = new int[]{7, 1, 3, 10, 5, 2, 8, 9, 6};
        buildHeap(array);
        System.out.println(Arrays.toString(array));
    }

    /**
     * 上浮调整
     * @param array
     */
    public static void upAdjust(int[] array) {
        int child = array.length - 1;
        int parent = (child - 1) >> 1;
        // 保存插入的叶子节点的值, 用于最后赋值
        int tmp = array[child];
        while (child > 0 && tmp < array[parent]) {
            // 单向赋值即可
            array[child] = array[parent];
            child = parent;
            parent = (child - 1) / 2;
        }
        array[child] = tmp;
    }

    /**
     * 下沉调整
     * @param array
     * @param parent
     */
     public static void downAdjust(int[] array, int parent) {
         int tmp = array[parent];
         int child = 2 * parent + 1;
         // 当 parent 为非叶子节点时
         while (child < array.length) {
             // 如果有右孩子, 找到较小值的孩子
             if (child + 1 < array.length && array[child] > array[child + 1]) child++;
             // 如果父节点小于孩子的值则直接跳出
             if (tmp <= array[child]) break;
             // 单向赋值即可
             array[parent] = array[child];
             parent = child;
             child = 2 * child + 1;
         }
         array[parent] = tmp;
     }

    /**
     * 构建堆
     * @param array
     */
    public static void buildHeap(int[] array) {
        // 从最后一个非叶子节点开始, 依次做下沉调整
        for (int i = (array.length - 2) / 2; i >= 0; i--) {
            downAdjust(array, i);
        }
    }
}
```

#### 什么是优先队列
##### 优先队列的特点
- 最大优先队列, 无论入队顺序如何, 都是当前最大的元素优先出队
- 最小优先队列, 无论入队顺序如何, 都是当前最小的元素优先出队

##### 优先队列的实现
```Java
class PriorityQueue {
    private int[] array;
    private int size;

    public PriorityQueue() {
        this(32);
    }

    public PriorityQueue(int capacity) {
        this.array = new int[capacity];
    }

    /**
     * 入队元素
     */
    public void enQueue(int element) {
        while (size >= array.length) {
            expandSize();
        }
        array[size++] = element;
        upAdjust();
    }

    /**
     * 出队元素
     * @return
     */
    public int deQueue() {
        if (size <= 0) {
            throw new IllegalStateException("The queue is empty!");
        }
        int head = array[0];
        array[0] = array[--size];
        downAdjust();
        return head;
    }

    /**
     * 扩展大小
     */
    private void expandSize() {
        array = Arrays.copyOf(array, size * 2);
    }

    /**
     * 上浮调整
     */
    private void upAdjust() {
        int child = size - 1;
        int parent = (child - 1) / 2;
        // 保存插入的叶子节点的值, 用于最后赋值
        int tmp = array[child];
        while (child > 0 && tmp < array[parent]) {
            // 单向赋值即可
            array[child] = array[parent];
            child = parent;
            parent = (child - 1) / 2;
        }
        array[child] = tmp;
    }

    /**
     * 下沉调整
     */
    private void downAdjust() {
        int parent = 0, child = 1;
        int tmp = array[parent];
        // 当 parent 为非叶子节点时
        while (child < size) {
            // 如果有右孩子, 找到较小值的孩子
            if (child + 1 < size && array[child] > array[child + 1]) child++;
            // 如果父节点小于孩子的值则直接跳出
            if (tmp <= array[child]) break;;
            // 单向赋值即可
            array[parent] = array[child];
            parent = child;
            child = 2 * child + 1;
        }
        array[parent] = tmp;
    }
}
```
