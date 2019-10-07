---
layout: post
title: "第一个只出现一次的字符"
date: "2019-07-15"
description: "第一个只出现一次的字符"
tag: [algorithm]
---

### 第一个只出现一次的字符

#### 题目一
字符串中第一个只出现一次的字符; 在字符串中找出第一个只出现一次的字符; 如输入 "abaccdeff", 则输出 'b'

##### 思路
最直观的解法是从头遍历每个字符, 将其与后续的字符比较, 如果没有重复的字符则返回; 但这样的解法复杂度为 $O(n^2)$  
在遍历字符串时, 可以使用一个 Map 按序来统计字符出现的次数, 最后返回第一个次数为 1 的字符; 这是以空间换时间的解法, 时间复杂度为 $O(n)$

##### 实现
```
import java.util.LinkedHashMap;
import java.util.Map;

public class Solution {
    public static void main(String[] args) {
        String str = "abaccdeff";
        System.out.println(firstNotRepeatingChar(str));
    }

    private static Character firstNotRepeatingChar(String str) {
        if (str == null || str.length() == 0) {
            return null;
        }

        Map<Character,Integer> map = new LinkedHashMap<>(16);
        char[] chars = str.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            map.compute(chars[i], (k, v) -> map.get(k) == null ? 1 : map.get(k) + 1);
        }
        for (Map.Entry<Character, Integer> entry : map.entrySet()) {
            if (entry.getValue() == 1) {
                return entry.getKey();
            }
        }
        return null;
    }
}
```

#### 相关题目
TODO

#### 题目二
字符流中第一个值出现一次的字符  
请实现一个函数, 用来找出字符流中第一个值出现一次的字符; 例如, 当从字符流中只读出前两个字符 "go" 时, 第一个只出现一次的字符是 'g'; 当从该字符流中读出前 6 个字符 "google" 时, 第一个值出现一次的字符是 'l'

##### 思路
TODO

##### 实现
TODO
