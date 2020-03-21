---
layout: post
title: "丑数"
date: "2019-07-14"
description: "丑数"
tag: [algorithm]
---

### 丑数

#### 题目
我们把只包含因子 2, 3, 5 的数称作丑数 (ugly number); 求按从小到大的顺序的第 1500 个丑数; 例如: 6 和 8 都是丑数, 但 14 不是, 因为它包含因子 7; 习惯上我们把 1 当作第一个丑数

##### 思路
判断一个数是丑数的最直接的办法是连续除 2, 3, 5, 最后得到的是 1, 那么这个数就是丑数; 为了找到第 1500 个丑数, 则需从小到大依次判断每个数, 直到找到第 1500 个丑数; 但这样的方式效率不高, 因为每个非丑数也会被执行这判断  
其实可以反过来想, 每个丑数都是由一个更小的丑数乘以  2, 3, 5 得到的; 使用一个容器来保存丑数, 先将 1 加入容器, 将容器中乘以 2, 3, 5 得到的最小数字加入容器, 循环这个过程知道容器中有 1500 个丑数时停止; 在计算下一个丑数时, 并不是将容器中所有的数拿出与 2, 3, 5相乘再比较最小值, 容器中一定有一个数乘以 2 得到数是刚好大于容器中已有的丑数 (保存其坐标为 t2Index), 同理 3, 5 也有这样对应的一个数 (保存其坐标为 t3Index, t5Index); 一开始这个数对应的下标为 0 (即是 1), 再计算下一个丑数时, 比较 2, 3, 5 分别乘以坐标为 t2Index, t3Index, t5Index 的数, 将最小的丑数加入容器, 并将对应坐标加一

##### 实现
```Java
import java.util.ArrayList;
import java.util.List;

public class Solution {
    public static void main(String[] args) {
        System.out.println(uglyNumber(1500));
    }

    private static int uglyNumber(int index) {
        if (index <= 0) {
            return 0;
        }

        List<Integer> uglyNumbers = new ArrayList<Integer>(){{add(1);}};
        int t2Index = 0, t3Index = 0, t5Index = 0;
        while (uglyNumbers.size() < index) {
            int t2UglyNumber = uglyNumbers.get(t2Index) * 2;
            int t3UglyNumber = uglyNumbers.get(t3Index) * 3;
            int t5UglyNumber = uglyNumbers.get(t5Index) * 5;
            int uglyNumber = Math.min(Math.min(t2UglyNumber, t3UglyNumber), t5UglyNumber);
            if (uglyNumber == t2UglyNumber) {
                t2Index++;
            }
            if (uglyNumber == t3UglyNumber) {
                t3Index++;
            }
            if (uglyNumber == t5UglyNumber) {
                t5Index++;
            }
            uglyNumbers.add(uglyNumber);
        }
        return uglyNumbers.get(uglyNumbers.size() - 1);
    }

}
```
