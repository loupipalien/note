---
layout: post
title: "旋转数组的最小数字"
date: "2019-06-07"
description: "旋转数组的最小数字"
tag: [algorithm]
---

### 旋转数组的最小数字

#### 题目一
把一个数字最开始的若干个元素搬到数组的末尾, 我们称之为数组的旋转; 输入一个递增排序的数组的一个旋转, 输出旋转数组的最小元素; 例如, 数组 `{3, 4, 5, 1, 2}` 为 `{1, 2, , 3, 4, 5}` 的一个旋转, 该数组的最小值为 1

##### 思路
最暴力的解法是从头到尾遍历数组, 找到最小值的元素, 但这样的算法复杂度是 $O(n)$, 并不理想, 也没有利用到旋转数组的特性  
可以看到有序数组旋转后分为了两个有序的子数组, 而且前面子数组的元素都大于等于后面子数组的元素; 最小值的元素正好是两个数组的分界线, 在排序的数组中可以使用二分查找法实现 $O(logn)$ 的查找:
- 找到数组的中间的元素, 如果该元素位于前面的子数组, 那么它应该大于等于第一个指针指向的元素, 此时数组中的最小值的元素应该位于此元素之后, 因此把第一个指针指向该中间元素以缩小查找范围
- 如果该元素位于后面的子数组, 那么它应该小于等于第二个指针指向的元素, 此时数组中的最小值的元素应该位于此元素之前, 因此把第二个指针指向该中间元素以缩小查找范围

按照以上操作, 第一个指针总是指向前面子数组的元素, 第二个指针总是指向后面子数组的元素; 最终第一个指针将指向前面子数组的最后一个元素, 第二个指针会指向后面子数组的第一个元素; 也就是说它们最终会指向两个相邻的元素, 并且第二个指针正好指向最小值的元素

##### 实现
```
public class MinNumberInRotateArray {
    public static void main(String[] args) {
        int[] array = {3, 4, 5, 1, 2};
        System.out.println(minNumberInRotateArray(array));
    }

    public static int minNumberInRotateArray(int [] array) {
        if (array == null || array.length == 0) {
            throw new IllegalArgumentException("Invalid parameter.");
        }
        if (array.length == 1) {
            return array[0];
        }

        int before = 0;
        int after = array.length - 1;
        int mid = before;
        while (array[before] >= array[after]) {
            if (after - before == 1) {
                mid = after;
                break;
            }

            mid = (before + after) / 2;
            if (array[mid] >= array[before]) {
                before = mid;
            } else if (array[mid] <= array[after]) {
                after = mid;
            }
        }
        return array[mid];
    }
}
```
此算法时间复杂度为 $O(logn)$

##### 问题
以上算法是存在 bug 的, 例如当 `mid`, `before`, `after` 三个指针指向的元素值都相等时, 数组 `{0, 1, 1, 1, 1}` 的旋转数组 `{1, 0, 1, 1, 1}` 和 `{1, 1, 1, 0, 1}`, 以上算法将会判断错误旋转数组 `{1, 0, 1, 1, 1}` 的最小值元素
出现以上 bug 的原因是, 当 `mid`, `before`, `after` 三个指针指向的元素值都相等时, 是不能判定 `mid` 指向的元素是属于前面子数组还是后面子数组的, 此时不得不采用顺序查找法; 最终代码如下
```
public class MinNumberInRotateArray {
    public static void main(String[] args) {
        int[] array = {1, 0, 1, 1, 1};
        System.out.println(minNumberInRotateArray(array));
    }

    public static int minNumberInRotateArray(int [] array) {
        if (array == null || array.length == 0) {
            throw new IllegalArgumentException("Invalid parameter.");
        }
        if (array.length == 1) {
            return array[0];
        }

        int before = 0;
        int after = array.length - 1;
        int mid = before;
        // 考虑特例: 旋转 0 个元素
        while (array[before] >= array[after]) {
            if (after - before == 1) {
                mid = after;
                break;
            }

            mid = (before + after) / 2;
            // 考虑特例: array[mid] 不能判定为属于前面子数组还是后面子数组
            if (array[before] == array[mid] && array[mid] == array[after]) {
                return minNumberInArray(Arrays.copyOfRange(array, before, after + 1));
            }
            if (array[mid] >= array[before]) {
                before = mid;
            } else if (array[mid] <= array[after]) {
                after = mid;
            }
        }
        return array[mid];
    }

    private static int minNumberInArray(int[] array) {
        int min = array[0];
        for (int i = 1; i < array.length; i++) {
            if (array[i] < min) {
                min = array[i];
            }
        }
        return min;
    }
}
```
