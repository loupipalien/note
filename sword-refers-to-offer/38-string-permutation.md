---
layout: post
title: "字符串的排列"
date: "2019-07-03"
description: "字符串的排列"
tag: [algorithm]
---

### 字符串的排列

#### 题目
输入一个字符串, 打印出该字符串中字符的所有排列; 例如, 输入字符串 `abc`, 则打印出有字符 `a, b, c` 所能排列处理啊的所有字符串 `abc, acb, bac, bca, cab, cba`

##### 思路
这里把一个字符串看作两个部分: 第一部分是它的第一个字符, 第二部分是后面的所有字符; 为了求整个字符串的排列, 第一步求所有可能出现在第一个位置的字符, 第二步是固定一个字符求后面所有字符的排列组合

##### 实现
```Java
import java.util.*;

public class Solution {
    public static void main(String[] args) {
        System.out.println(stringPermutaion("abc"));
    }

    private static List<String> stringPermutaion(String str) {
        if (str == null || str.isEmpty()) {
            return new ArrayList<>();
        }

        List<String> list = Arrays.asList(str.split(""));
        return permutation(list);
    }

    private static List<String> permutation(List<String> list) {
        Set<String> result = new TreeSet<>();
        if (list == null || list.isEmpty()) {
            return new ArrayList<>(result);
        }
        if (list.size() == 1) {
            result.addAll(list);
            return new ArrayList<>(result);
        }
        if (list.size() == 2) {
            result.add(toString(list));
            result.add(toReverseString(list));
            return new ArrayList<>(result);
        }

        for (int i = 0; i < list.size(); i++) {
            String first = list.get(i);
            List<String> second = cloneThenRemove(list, i);
            List<String> subResult = permutation(second);
            for (String s : subResult) {
                result.add(first.concat(s));
            }
        }
        return new ArrayList<>(result);
    }

    private static String toString(List<String> list) {
        StringBuilder builder = new StringBuilder("");
        for (String s : list) {
            builder.append(s);
        }
        return builder.toString();
    }

    private static String toReverseString(List<String> list) {
        StringBuilder builder = new StringBuilder("");
        for (int i = list.size() - 1; i > -1 ; i--) {
            builder.append(list.get(i));
        }
        return builder.toString();
    }

    private static <T> List<T> cloneThenRemove(List<T> list, int index) {
        if (list == null || list.size() <= index) {
            throw new IllegalArgumentException("Invalid Parameter");
        }

        List<T> clonedList = new ArrayList<>();
        clonedList.addAll(list);
        clonedList.remove(index);
        return clonedList;
    }
}
```

#### 相关题目
TODO
