---
layout: post
title: "调整数组顺序使奇数位于偶数前面"
date: "2019-06-16"
description: "调整数组顺序使奇数位于偶数前面"
tag: [algorithm]
---

### 调整数组顺序使奇数位于偶数前面

#### 题目一
输入一个整数数组, 实现一个函数来调用该数组中数字的顺序, 使得所有奇数位于数组的前半部分, 所有偶数位于数组的后半部分

##### 思路
如果使用蛮力解法, 那么只需要从头到尾遍历数组, 遇到一个偶数时, 将这个偶数后的所有元素向前移动一位, 并将这个偶数放置数组末尾; 但是这样的时间复杂度为 $O(n^2)$  
可以换一种思路, 通过交换来达到目的: 维护两个指针, 一个从头开始遍历, 遇到一个偶数时停下, 另一个从尾遍历, 遇到一个奇数是停下, 交换这两个元素, 循环这个过程直到两个指针相遇

##### 实现
```Java
public class ReorderArray {
    public static void main(String[] args) {
        int[] array = new int[]{1, 2, 3, 4, 5, 6, 7};
        reorderArray(array);
        for (int i = 0; i < array.length; i++) {
            System.out.print(array[i] + " ");
        }
    }

    private static void reorderArray(int[] array) {
        if (array == null || array.length == 0) {
            return;
        }

        int i = 0;
        int j = array.length - 1;

        while (i < j) {
            while ((array[i] & 0x1) != 0) {
                i++;
            }
            while ((array[j] & 0x1) == 0) {
                j--;
            }
            if (i < j) {
                int tmp = array[i];
                array[i] = array[j];
                array[j] = tmp;
            }
        }
    }
}
```
以上代码时间复杂度为 $O(1)$

#### 扩展
TODO
