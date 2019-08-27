### 数组的 partition 调整

#### 题目
给定一个有序数组 array, 调整 array 使得这个数组的左半部分没有重复元素且升序, 而不用保证右半部分是否有序; 例如, array = [1, 2, 2, 2, 3, 3, 4, 5, 6, 6, 7, 7, 8, 8, 8, 9], 调整之后 array = [1, 2, 3, 4, 5, 6, 7, 8, 9,...]

#### 补充题目
给定一个数组 array, 其中只可能含有 0, 1, 2 三个值, 请实现 array 的排序  
另一种问法为: 有一个数组, 其中只有红球, 蓝球, 黄球, 请实现红球全放数组的左边, 篮球放在中间, 黄球放在右边  
另一种问法为: 有一个数组, 再给定一个值 k, 请实现比 k 小的数都放在数组的左边, 等于 k 的数都放在数组的中间, 比 k 大的数都放在数组的右边

#### 要求
- 所有题目实现的实现复杂度为 O(N)
- 所有题目实现的额外空间复杂度为 O(1)

#### 难度
:star:

#### 思路
- 原问题：遍历数组, 记录已排序且无重复元素的最后一个下标, 记为 current; 当遍历的元素与当前值不同, 遍历元素下标等于 current + 1 则 current 加一继续, 遍历元素下标大于 current + 1 则交换当前元素与 current + 1 的值后 current 加一继续
- 补充问题: 遍历数组, 将 0 值交换到前面, 2 交换到后面

#### 实现
- 原问题
```Java
public class Solution {

    public static void main(String[] args) {
        int[] array = new int[]{1, 2, 2, 2, 3, 3, 4, 5, 6, 6, 7, 7, 8, 8, 8, 9};
        leftUnique(array);
        for (int i : array) {
            System.out.print(i + " ");
        }
    }

    public static void leftUnique(int[] array) {
        if (array == null || array.length < 2) {
            return;
        }

        int current = 0;
        for (int i = 1; i < array.length; i++) {
            if (array[i] > array[current]) {
                if (current + 1 < i) {
                    int temp = array[i];
                    array[i] = array[current + 1];
                    array[current + 1] = temp;
                }
                current++;
            }
        }
    }
}
```
- 补充问题
```Java
public class Solution {

    public static void main(String[] args) {
        int[] array = new int[]{0, 1, 0, 2, 2, 1, 1, 0, 2};
        sort(array);
        for (int i : array) {
            System.out.print(i + " ");
        }
    }

    public static void sort(int[] array) {
        if (array == null || array.length < 2) {
            return;
        }

        int left = -1, right = array.length, current = 0;
        while (current < right) {
            if (array[current] == 0) {
                swap(array, ++left, current++);
            } else if (array[current] == 2) {
                swap(array, current, --right);
            } else {
                current++;
            }
        }
    }

    private static void swap(int[] array, int indexA, int indexB) {
        int temp = array[indexA];
        array[indexA] = array[indexB];
        array[indexB] = temp;
    }
}
```
