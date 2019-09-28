---
layout: post
title: "数据流中的中位数"
date: "2019-07-06"
description: "数据流中的中位数"
tag: [algorithm]
---

### 数据流中的中位数

#### 题目
如何得到一个数据流中的中位数? 如果从数据流中读出奇数个数值, 那么中位数就是所有数值排序之后位于中间的数值; 如果从数据流中读出偶数个数值, 那么中位数就是所有数值排序之后中间两个数的平均值

##### 思路
以下是使用不同数据结构的时间复杂度对比

| 数据结构 | 插入时间的复杂度 | 得到中位数的时间复杂度  |
| :--- | :--- | :--- |
| 没有排序的数组 | O(1) | O(n) |
| 排序的数组 | O(n) | O(1) |
| 排序的链表 | O(n) | O(1) |
| 二叉搜索树 | 平均 O(logn), 最差 O(n) | 平均 O(logn), 最差 O(n) |
| AVL 树 | O(logn) | O(1) |
| 最大堆和最小堆 | O(logn) | O(1) |

##### 实现
```
public class Solution {

    // 保存较大的元素,
    private TreeSet<Double> minSet = new TreeSet<>();
    // 保存较小的元素
    private TreeSet<Double> maxSet = new TreeSet<>((d1, d2) -> -(d1.compareTo(d2)));

    public void insert(Integer num) {
        // 放入 max
        if (minSet.size() > maxSet.size()) {
            if (!minSet.isEmpty() && num > minSet.first()) {
                maxSet.add(minSet.pollFirst());
                minSet.add((double)num);
            } else {
                maxSet.add((double)num);
            }
        } else {
            if (!maxSet.isEmpty() && num < maxSet.first()) {
                minSet.add(maxSet.pollFirst());
                maxSet.add((double)num);
            } else {
                minSet.add((double)num);
            }
        }
    }

    public Double getMedian() {
        if (minSet.isEmpty() && maxSet.isEmpty()) {
            return 0.0;
        }
        if (minSet.size() == maxSet.size()) {
            return (minSet.first() + maxSet.first()) / 2;
        }
        return minSet.size() > maxSet.size() ? minSet.first() : maxSet.first();
    }
}
```
