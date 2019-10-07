---
layout: post
title: "数组中的逆序对"
date: "2019-07-16"
description: "数组中的逆序对"
tag: [algorithm]
---

### 数组中的逆序对

#### 题目
在数组中的两个数字, 如果前面一个数字大于后面的数字, 则这两个数字组成一个逆序对; 输入一个数组, 求出这个数组中的逆序对的总数; 例如, 在数组 `{7, 5, 6, 4}` 中, 一共存在 5 个逆序对, 分别是 (7, 6), (7, 5), (7, 4), (6, 4), (5, 4)

##### 思路
先把数组分隔为子数组, 统计出子数组内的逆序对个数, 然后在统计两个相邻子数组之间的逆序对个数; 在统计逆序对过程中, 需要对数组进行排序, 这个排序过程即为归并排序

##### 实现
```
public class Solution {
    public static void main(String[] args) {
        int[] array = new int[]{7, 5, 6, 4};
        System.out.println(inversePairs(array));
    }

    private static int inversePairs(int[] array) {
        if (array == null || array.length == 0) {
            return 0;
        }
        return mergeSort(array, 0, array.length - 1);
    }

    private static int mergeSort(int[] array, int start, int end) {
        if (start == end) {
            return 0;
        }

        int mid = (end + start) >> 1;
        int leftCount = mergeSort(array, start, mid);
        int rightCount = mergeSort(array, mid + 1, end);
        // i 初始化为前半段最后一个数字的下标, j 初始化为前半段最后一个数字的下标
        int i = mid, j = end;
        int[] copyArray = new int[end - start + 1];
        int index = copyArray.length - 1;
        int count = 0;
        while (i >= start && j >= mid + 1) {
            if (array[i] > array[j]) {
                copyArray[index--] = array[i--];
                count += j - mid;
            } else {
                copyArray[index--] = array[j--];
            }
        }
        while (i >= start) {
            copyArray[index--] = array[i--];
        }
        while (j >= mid + 1) {
            copyArray[index--] = array[j--];
        }
        for (int k = 0; k < copyArray.length; k++) {
            array[start++] = copyArray[k];
        }
        return leftCount + rightCount + count;
    }
}
```
