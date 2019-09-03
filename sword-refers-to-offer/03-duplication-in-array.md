---
layout: post
title: "数组中的重复数字"
date: "2019-05-28"
description: "数组中的重复数字"
tag: [algorithm]
---

### 数组中重复的数字

#### 题目一: 找出数组中重复的数字
在一个长度为 n 的数组里的所有数字都在 0 ~ n-1 的范围里; 数组中某些数字是重复的, 但不知道有几个数字重复了, 也不知道每个数字重复了几次; 请找出数组中任意一个重复的数字; 例如, 如果输入长度为 7 的数组 `{2, 3, 1, 0, 2, 5, 3}`, 那么对应的输出是重复的数字 2 或 3

##### 思路
通用的解决思路有以下两种, 基本可以应用于所有整数数组
- 将给定数组排序, 然后在遍历比较; 时间复杂度为 $ O(nlogn) + O(n) = O(nlogn) $, 空间复杂度为 $ O(1) $
- 遍历数组将其放置哈希表中, 数字作为 key, 添加时先判断是否已存在; 时间复杂度为 $ O(n) $, 空间复杂度为 $ O(n) $

但本题题目中还给定了 `一个长度为 n 的数组里的所有数字都在 0 ~ n-1 的范围` 的条件, 可以利用这个条件重排数组, 发现重复数字  
从头遍历数组, 当遍历到下标为 `i` 的数字时, 首先比较这个数字 (用 `m` 表示) 是不是 `i`; 如果是, 则接着遍历下一个数字, 如果不是, 则再拿它和第 `m` 个数字进行比较; 如果它和第 `m` 个数字不相等, 就把第 `i` 个数字和第 `m` 个数字交换, 即把 `m` 放到第 `m` 个位置; 接下来重复做这个比较, 交换的过程, 直到遍历完整个数组

##### 实现
```
public class DuplicationInArray {
    public static void main(String[] args) {
        int[] array = new int[]{2, 3, 1, 0, 2, 5, 3};
        System.out.println(findDuplicationInArray(array));
    }

    private static int duplicationInArray(int[] array) {
        int noDuplication = -1;
        // check array
        if (array == null || array.length < 1) {
            return noDuplication;
        }
        // check number
        for (int i = 0; i < array.length; i++) {
            if (array[i] < 0 || array[i] > array.length - 1){
                return noDuplication;
            }
        }

        for (int i = 0; i < array.length; i++) {
            while (i != array[i]) {
                int temp = array[i];
                if (temp != array[temp]) {
                    array[i] = array[temp];
                    array[temp] = temp;
                } else {
                    return temp;
                }
            }
        }
        return noDuplication;
    }
}
```
这个方法的时间复杂度为 $ O(n) $, 空间复杂度为 $ O(1) $


#### 题目二: 不修改数组找出重复的数字
在一个长度为 n+1 的数组里的所有数字都在 1 ~ n 的范围内, 所以数组中至少有一个数字是重复的; 请找出数组中任意一个重复的数字, 但不能修改输入的数组; 例如, 如果输入长度为 8 的数组 `{2, 3, 5, 4, 3, 2, 6, 7}`, 那么对应的输出是重复的数字 2 或数字 3

##### 思路
此题与上一题类似, 只是不能修改数组; 那么只要将原数组复制一个辅助数组, 就可以使用与上题一样的思路; 这里有一种避免使用 $ O(n) $ 的辅助空间的方法, 来找到数组中任意重复的元素  
由于 `一个长度为 n+1 的数组里的所有数字都在 1 ~ n 的范围`, 那么必然有一个重复的数字; 由于不能修改数组, 可以考虑使用二分法将重复数字的范围缩小直至找到 (此方法不能找出所有重复的数字); 即将 `1 ~ n` 的区间二分, 然后统计数组中出现两个子区间的数字的频次是否大于对应子区间的范围, 如果大于那么必有一个重复数字, 再在子区间重复这一过程直到子区间个数为 1

##### 实现
```
public class DuplicationInArrayNoEdit {
    public static void main(String[] args) {
        int[] array = new int[]{2, 3, 1, 0, 2, 5, 3};
        System.out.println(findDuplicationInArrayNoEdit(array));
    }

    private static int duplicationInArrayNoEdit(int[] array) {
        int noDuplication = -1;
        // check array
        if (array == null || array.length < 1) {
            return noDuplication;
        }
        // check number
        for (int i = 0; i < array.length; i++) {
            if (array[i] < 0 || array[i] > array.length - 1){
                return noDuplication;
            }
        }

        int start = 1;
        int end = array.length - 1;
        while (start <= end) {
            int middle = (start + end) / 2;
            int count = countRange(array, start, middle);
            if (start == end) {
                if (count > 1) {
                    return start;
                }
                break;
            }
            if (count > (middle - start + 1)) {
                end = middle;
            } else {
                start = middle + 1;
            }
        }
        return noDuplication;
    }

    private static int countRange(int[] array, int start, int end) {
        int count = 0;
        for (int i = 0; i < array.length; i++) {
            if (start <= array[i] && array[i] <= end) {
                count++;
            }
        }
        return count;
    }
}
```
这个方法的时间复杂度为 $ O(nlogn) $, 空间复杂度为 $ O(1) $; 与使用辅助空间的方法相比, 相当于以时间换空间  
这种算法不能保证找出所有重复的数字, 例如: `{2, 3, 5, 6, 3, 2, 6, 7}` 中的重复数字 2, 因为在 1 ~ 2 的范围里有 1 和 2 两个数字, 这个范围里的数字也出先了两次, 此时使用该算法不能确定每个数字各出现一次还是某个数字出现了两次
