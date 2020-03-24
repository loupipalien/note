### 排序算法

#### 引言
时间复杂度为 $O(n^2)$ 的排序算法
- 冒泡排序
- 选择排序
- 插入排序
- 希尔排序 (性能略优于 $O(n^2)$, 但又比不上 $O(nlogn)$, z暂时归入本类)

时间复杂度为 $O(nlogn)$ 的排序算法
- 快速排序
- 归并排序
- 堆排序

时间复杂度为线性的排序算法
- 计数排序
- 桶排序
- 基数排序

#### 什么是冒泡排序
##### 初识冒泡排序
冒泡排序的思想: 把相邻的元素两两比较, 当一个元素大于右侧相邻元素时, 交换它们的位置; 当一个元素小于等于右侧相邻元素时, 位置不变  
冒泡排序是一种稳定排序, 值相等的元素并不会被打乱原本的顺序; 由于该排序算法的每一轮都要遍历所有元素, 总共遍历 (n - 1) 轮, 所以平均时间复杂度为 $O(n^2)$

```Java
public static void bubbleSort(int[] array) {
    // 数组中无序部分的边界
    int unsortedBorder = array.length - 1;

    for (int i = 0; i < array.length - 1; i++) {
        // 有序标记
        boolean isSorted = true;
        // 最后依次交换的位置
        int lastExchangeIndex = 0;
        for (int j = 0; j < unsortedBorder; j++) {
            if (array[j] > array[j + 1]) {
                swap(array, j, j + 1);
                isSorted = false;
                lastExchangeIndex = j;
            }
        }
        // 如果已有序则直接跳出
        if (isSorted) break;
        // 更新无序部分的边界
        unsortedBorder = lastExchangeIndex;
    }
}

private static void swap(int[] array, int x, int y) {
    int tmp = array[x];
    array[x] = array[y];
    array[y] = tmp;
}
```

##### 鸡尾酒排序
原理是双向冒泡排序, 可以在特定的条件下减少排序的回合数

#### 什么是快速排序
##### 初识快速排序
快速排序的思想: 在每一轮挑选一个基准元素, 并让其他比它大的元素移动到数组的一边, 比它小的元素移动到数组的另一边, 从而把数组拆分成两个部分   
快排算法的核心是分区函数, 分区函数可实现为双边循环和单边循环, 单边循环较为简单并且选择随机位置作为枢轴更容易

```Java
public static void quickSort(int[] array) {
    if (array == null || array.length < 2) {
        return;
    }

    quickSort(array, 0, array.length - 1);
}

private static void quickSort(int[] array, int left, int right) {
    if (array == null || array.length < 2 || right - left < 1) {
        return;
    }

    if (left < right) {
        int pivot = partition(array, left, right);
        quickSort(array, left, pivot - 1);
        quickSort(array, pivot + 1, right);
    }
}

private static int partition(int[] array, int left, int right) {
    // pivot 表示枢轴
    int pivot = array[left];

    int index = left + 1;
    for (int i = index; i <= right; i++) {
        if (array[i] < pivot) {
            swap(array, i, index++);
        }
    }
    swap(array, left, --index);
    return index;
}

private static void swap(int[] array, int x, int y) {
    int tmp = array[x];
    array[x] = array[y];
    array[y] = tmp;
}
```

#### 什么是堆排序
##### 传说中的堆排序
堆排序的思想: 把无序数组构建成二叉堆, 需要从小到大排序则构建成最大堆, 需要从大到校排序则构建成最小堆; 然后循环删除堆顶元素, 替换到二叉堆的末尾, 调整产生新的堆顶

```Java
public static void heapSort(int[] array) {
    // 将无序数组构建为最大堆
    for (int i = (array.length - 2) / 2; i >= 0; i--) {
        downAdjust(array, i, array.length);
    }

    // 循环删除堆顶元素, 移到集合尾部, 调整堆产生的新的堆顶
    for (int i = array.length - 1; i >= 0; i--) {
        swap(array, 0, i);
        // 重新调整堆
        downAdjust(array, 0, i);
    }
}

private static void downAdjust(int[] array, int parent, int length) {
    int tmp = array[parent];
    int child = 2 * parent + 1;
    // 当 parent 为非叶子节点时
    while (child < length) {
        // 如果有右孩子, 找到较小值的孩子
        if (child + 1 < length && array[child] < array[child + 1]) child++;
        // 如果父节点小于孩子的值则直接跳出
        if (tmp >= array[child]) break;
        // 单向赋值即可
        array[parent] = array[child];
        parent = child;
        child = 2 * child + 1;
    }
    array[parent] = tmp;
}

private static void swap(int[] array, int x, int y) {
    int tmp = array[x];
    array[x] = array[y];
    array[y] = tmp;
}
```

#### 计数排序和桶排序
##### 初识计数排序
计数排序适用于一定取值范围不大的整数排序, 其思想为统计整数出现的次数, 将整个范围内的整数按频次输出; 其时间复杂度为 $O(n)$

```Java
public static int[] countSort(int[] array) {
    // 得到数组的极值
    int min = array[0], max = array[0];
    for (int i = 1; i < array.length; i++) {
        if (array[i] < min) min = array[i];
        if (array[i] > max) max = array[i];
    }

    int size = max - min + 1;
    int[] counts = new int[size];
    // 统计元素的次数
    for (int i = 0; i < array.length; i++) {
        counts[array[i] - min]++;
    }

    // 构造结果
    int index = 0;
    int[] sorted = new int[array.length];
    for (int i = 0; i < counts.length; i++) {
        int count = counts[i];
        while (count-- > 0) {
            sorted[index++] = min + i;
        }
    }
    return sorted;
}
```

##### 什么是桶排序
桶排序适用于在某个范围内均匀分布的整数排序, 其思想为将范围均匀分段称之为桶, 并将数组中的值分配到各个桶中, 桶内先排序后, 依次输出各个桶中元素即可; 当元素分布均匀时其时间复杂度为 $O(n)$, 分布极其不均匀时降为 $O(nlogn)$

```Java
public static int[] bucketSort(int[] array) {
    // 得到数组的极值
    int min = array[0], max = array[0];
    for (int i = 1; i < array.length; i++) {
        if (array[i] < min) min = array[i];
        if (array[i] > max) max = array[i];
    }

    // 初始化桶, 桶个数自定义
    int bucketNumber = array.length;
    ArrayList<LinkedList<Integer>> buckets = new ArrayList<>(bucketNumber);
    for (int i = 0; i < bucketNumber; i++) buckets.add(new LinkedList<>());

    // 遍历数组将元素放入桶中
    int range = max - min;
    for (int i = 0; i < array.length; i++) {
        int number = ((array[i] - min) * (bucketNumber - 1) / range);
        buckets.get(number).add(array[i]);
    }

    // 桶内排序
    for (int i = 0; i < bucketNumber; i++) {
        Collections.sort(buckets.get(i));
    }

    // 输出全部元素
    int index = 0;
    int[] sorted = new int[array.length];
    for (LinkedList<Integer> bucket : buckets) {
        for (Integer element : bucket) {
            sorted[index++] = element;
        }
    }

    return sorted;
}
```

#### 小结
| 排序算法 | 平均时间复杂度 | 最坏时间复杂度 | 空间复杂度 | 是否稳定排序 |
| :--- | :--- | :--- | :--- | :--- |
| 冒泡排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定 |
| 鸡尾酒排序 | $O(n^2)$ | $O(n^2)$ | $O(1)$ | 稳定 |
| 快速排序 | $O(nlogn)$ | $O(n^2)$ | $O(logn)$ | 稳定 |
| 堆排序排序 | $O(nlogn)$ | $O(nlogn)$ | $O(1)$ | 稳定 |
| 计数排序 | $O(n + m)$ | $O(n + m)$ | $O(m)$ | 稳定 |
| 桶排序排序 | $O(n)$ | $O(nlogn)$ | $O(n)$ | 稳定 |
