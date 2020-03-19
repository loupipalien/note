---
layout: post
title: "数组中出现次数超过一半的数字"
date: "2019-07-04"
description: "数组中出现次数超过一半的数字"
tag: [algorithm]
---

### 数组中出现次数超过一半的数字

#### 题目
数组中有一个数字出现次数超过数组长度的一半, 请找出这个数字; 例如, 输入一个长度为 9 的数组 `{1, 2, 3, 2, 2, 2, 5, 4, 2}`, 由于数字在数组中出现了 5 次, 超过了数组长度的一半, 因此输出 2

##### 思路
如果数组是有序的, 很容易就找出出现次数最多的数字, 而题目中给出的数字是乱序的, 如果将其排序, 那么排序的时间复杂度最小是 $O(nlogn)$; 这里有两种时间复杂度为 $O(n)$ 的算法
- 基于 partition 函数的时间复杂度为 $O(n)$ 的算法
TODO
- 根据数组特点找出时间复杂度为 $O(n)$ 的算法
从另一个角度来看次数超过一半的数字, 即它出现的次数比其他所有数字出现的次数都多, 因此可以在遍历数组是保存两个值, 一个是数组中的一个数字, 一个是次数; 当遍历到下一个数字的时候, 如果下一个数字和保存的数字相同则次数加一, 如果不同次数减一, 如果次数为零, 那么需要保存下一个数字, 并把次数设置为 1; 由于要找的数字出现的次数比其他所有数字出现的次数之和还要多, 那么要找的数字肯定是最后一次把次数设置为 1 时对应的数字

##### 实现
- 基于 Partition 函数的时间复杂度为 $O(n)$ 的算法
TODO

- 根据数组特点找出时间复杂度为 $O(n)$ 的算法
```Java
public class MoreThanHalfNumber {
    public static void main(String[] args) {
        System.out.println(moreThanHalfNumber(new int[]{1, 2, 3, 2, 2, 2, 5, 4, 2}));
    }

    private static int moreThanHalfNumber(int[] array) {
        if (array == null || array.length != 0) {
            int number = array[0];
            int count = 1;
            for (int i = 1; i < array.length; i++) {
                if (count == 0) {
                    number = array[i];
                    count++;
                } else {
                    count = number == array[i] ? count + 1 : count - 1;
                }
            }

            if (count > 0) {
                return checkMoreThanHalf(array, number) ? number : 0;
            }
        }
        return 0;
    }

    private static boolean checkMoreThanHalf(int[] array, int number) {
        int count = 0;
        for (int i = 0; i < array.length; i++) {
            if (array[i] == number) {
                count++;
            }
        }
        return (count << 1) > array.length ? true : false;
    }

}
```
