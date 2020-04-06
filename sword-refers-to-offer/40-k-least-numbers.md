---
layout: post
title: "最小的 k 个数"
date: "2019-07-05"
description: "最小的 k 个数"
tag: [algorithm]
---

### 最小的 k 个数

#### 题目
输入 n 个整数, 找出其中最小的 k 个数; 例如, 输入 `4, 5, 1, 6, 2, 7, 3, 8` 这 8 个数字, 则最小的 4 个数字是 `1, 2, 3, 4`

##### 思路
最直接的思路是将数组排序, 但这样的算法复杂度最小为 $O(nlogn)$, 这里有时间复杂度更小的算法
- 基于 partition 函数的时间复杂度为 $O(n)$ 的算法
如果基于数组的第 k 的数字用 partiton 算法来调整, 那么调整过后使得比 k 小的所有数字都位于数组的左边, 比第 k 大的所有数字都位于数组的右边
- 时间复杂度为 $O(nlogk)$ 的算法
TODO

##### 实现
- 基于 partition 函数的时间复杂度为 $O(n)$ 的算法
```Java
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class KLeastNumbers {
    public static void main(String[] args) {
        System.out.println(kLeastNumbers(new int[]{4, 5, 1, 6, 2, 7, 3, 8}, 0));
    }

    private static List<Integer> kLeastNumbers(int[] data, int k) {
        List<Integer> list = new ArrayList<>();
        if (data == null || data.length < k || k <= 0) {
            return list;
        }

        int start = 0;
        int end = data.length - 1;
        int index = partition(data, start, end);
        while (index != k - 1) {
            if (index < k - 1) {
                start = index + 1;
                index = partition(data, start, end);
            } else {
                end = index - 1;
                index = partition(data, start, end);
            }
        }
        for (int i = 0; i < k; i++) {
            list.add(data[i]);
        }
        return list;
    }

    private static int partition(int[] data, int start, int end) {
        if (data == null || data.length == 0 || start < 0 || end >= data.length) {
            throw new IllegalArgumentException("Invalid Parameter.");
        }
        if (start == end) {
            return start;
        }

        int index = new Random().nextInt(end - start) + start;
        swap(data, index, end);
        int small = start - 1;
        for (int i = start; i < end; i++) {
            if (data[i] < data[end]) {
                small++;
                if (small != i) {
                    swap(data, small, i);
                }
            }
        }
        swap(data, ++small, end);
        return small;
    }

    private static void swap(int[] data, int indexM, int indexN) {
        int tmp = data[indexM];
        data[indexM] = data[indexN];
        data[indexN] = tmp;
    }

}
```
- 时间复杂度为 $O(nlogk)$ 的算法
TODO
