---
layout: post
title: "最长的不重复子字符串"
date: "2019-07-13"
description: "最长的不重复子字符串"
tag: [algorithm]
---

### 最长的不重复子字符串

#### 题目
请从字符串中找出一个最长的不包含重复字符的子字符串, 计算该最长子字符串的长度; 假设字符串中只包含 `a ~ z` 的字符; 例如: 在字符串 "arabcacfr" 中, 最长的不含重复字符的子字符串是 "acfr", 长度为 4

##### 思路
预置一个数组保存字符上次出现的下标, 默认值是 -1 表示尚未出现过; 遍历字符串的字符
- 如果当前字符的下标为 -1, 则当前子字符串长度加一
- 如果当前字符下标不为 -1, 表示此字符之前出现过, 先更新当前最长子字符串的长度
  - 如果此字符是否在当前子字符串中有出现, 有则需计算子字符串的长度

##### 实现
```Java
public class Solution {
    public static void main(String[] args) {
        String str = "arabcacfr";
        System.out.println(longestSubstringWithoutDup(str));
    }

    private static int longestSubstringWithoutDup(String str) {
        int maxLength = 0;
        if (str == null || str.length() == 0) {
            return maxLength;
        }

        // 26 个字符上一次出现的下标, 初始化为 -1 表示尚未出现过
        int[] positions = new int[26];
        for (int i = 0; i < positions.length; i++) {
            positions[i] = -1;
        }
        int currentLength = 0;
        char[] chars = str.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            int index = positions[chars[i] - 'a'];
            if (index == -1) {
                currentLength++;
            } else {
                // 更新最长子字符串的长度
                maxLength = currentLength > maxLength ? currentLength : maxLength;
                // 当前字符出现在当前最长子字符串中时, 重新计算当前最长子字符串的长度
                if (i < index + currentLength) {
                    currentLength = i - index;
                }
            }
            // 更新字符上一次出现的下标
            positions[chars[i] - 'a'] = i;
        }
        return currentLength > maxLength ? currentLength : maxLength;
    }
}
```
