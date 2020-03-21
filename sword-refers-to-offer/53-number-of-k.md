---
layout: post
title: "在排序数组中查找数字"
date: "2019-07-18"
description: "在排序数组中查找数字"
tag: [algorithm]
---

### 在排序数组中查找数字

#### 题目一
数字在排序数组中出现的次数;
统计一个数字在排序数组中出现的次数; 例如: 输入排序数组 `{1, 2, 3, 3, 3, 3, 4, 5}` 和数字 3, 由于 3 在这个数组中出现了 4 次, 因此输出 4

##### 思路
在排序数组中找到某个数可以使用二分查找法, 这里可以先使用二分查找找到一个指定的数, 如果这个数存在, 那么以这个数的下标为起点向前向后找到这个数的起始和结尾, 因为数组长度为 n, 所以算法复杂度为 $O(n)$; 这样与从头到尾遍历数组的时间复杂度相同  
可以利用二分查找法在排序数组中找到第一次出现指定数出现的下标, 同理也找到最后出现的下标, 这样的时间复杂度为 $O(logn)$

##### 实现
```Java
public class Solution {
    public static void main(String[] args) {
        int[] array = {1, 2, 3, 3, 3, 3, 4, 5};
        System.out.println(numberOfK(array, 3));
    }

    private static int numberOfK(int[] array, int k) {
        if (array == null || array.length == 0) {
            return 0;
        }

        int firstK = indexOfFirstK(array, k, 0, array.length - 1);
        if (firstK > -1) {
            int lastK = indexOfLastK(array, k, 0, array.length - 1);
            return lastK - firstK + 1;
        }
        return 0;
    }

    private static int indexOfFirstK(int[] array, int k, int start, int end) {
        if (array == null || array.length == 0 || start > end || end >= array.length) {
            return -1;
        }

        int middle = (start + end) >> 1;
        while (start <= end) {
            if (array[middle] < k) {
                start = middle + 1;
            } else if (array[middle] > k) {
                end = middle - 1;
            } else {
                // 当前元素为数组的第一个元素 或者前一个元素小于 k, middle 为第一个 k 的下标
                if (middle == 0 || array[middle - 1] < k) {
                    return middle;
                }
                end = middle - 1;
            }
            middle = (start + end) >> 1;
        }
        return -1;
    }

    private static int indexOfLastK(int[] array, int k, int start, int end) {
        if (array == null || array.length == 0 || start > end || end >= array.length) {
            return -1;
        }

        int middle = (start + end) >> 1;
        while (start <= end) {
            if (array[middle] < k) {
                start = middle + 1;
            } else if (array[middle] > k) {
                end = middle - 1;
            } else {
                // 当前元素为数组的最后一个元素, 或者后一个元素大于 k, middle 为最后一个 k 的下标
                if (middle == array.length - 1 || array[middle + 1] > k) {
                    return middle;
                }
                start = middle + 1;
            }
            middle = (start + end) >> 1;
        }
        return -1;
    }
}
```

#### 题目二
0 ~ n -1 中缺失的数字;  
一个长度为 n - 1 的递增排序数组中的所有数字都是唯一的, 并且每个数字都在范围 0 ~ n - 1 之内; 在范围 0 ~ n - 1 内的 n 个数字有且只有一个数字不再该数组中, 请找出这个数字

##### 思路
TODO

##### 实现
```Java
public class Solution {
    public static void main(String[] args) {
        int[] array = {0, 1, 2, 3, 4, 5, 6};
        System.out.println(missingNumber(array));
    }

    private static int missingNumber(int[] array) {
        if (array == null || array.length == 0) {
            return -1;
        }

        int low = 0;
        int high = array.length - 1;
        int middle = (low + high) >> 1;
        while (low <= high) {
            if (middle == array[middle]) {
                low = middle + 1;
            } else {
                // 这里 middle < array[middle]
                if (middle == 0 || array[middle - 1] == middle - 1) {
                    return middle;
                }
                high = middle - 1;
            }
            middle = (low + high) >> 1;
        }
        if (low == array.length) {
            return low;
        }
        throw new IllegalArgumentException("Invalid parameter.");
    }
}
```

#### 题目三
数组中数值和下标相等的元素;  
假设一个单调递增的数组里的每个元素都是整数并且是唯一的; 请编程实现一个函数, 找出数组中任意一个数值等于其下标的元素; 例如, 在数组 `{-3, -1, 1, 3, 5}` 中, 数字 3 和它的下标相等

##### 思路
TODO

##### 实现
```Java
public class Solution {
    public static void main(String[] args) {
        int[] array = {-3, -1, 1, 3, 5};
        System.out.println(integerIdenticalToIndex(array));
    }

    private static int integerIdenticalToIndex(int[] array) {
        if (array == null || array.length == 0) {
            return -1;
        }

        int low = 0;
        int high = array.length - 1;
        int middle = (low + high) >> 1;
        while (low <= high) {
            if (middle > array[middle]) {
                low = middle + 1;
            } else if (middle < array[middle]) {
                high = middle - 1;
            } else {
                return middle;
            }
            middle = (low + high) >> 1;
        }
        return -1;
    }
}
```
