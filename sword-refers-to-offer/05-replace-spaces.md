---
layout: post
title: "替换空格"
date: "2019-05-30"
description: "替换空格"
tag: [algorithm]
---

### 替换空格

#### 题目一
请实现一个函数, 把字符串中的每个空格替换成 "%20"; 例如, 输入 "We are happy.", 则输出 "We%20are%20happy." (不允许使用辅助数组, 原字符数组后续有足够的空间)

##### 思路
通常的思路是从头遍历字符, 找到匹配的字符后, 将后续的字符后移 m - 1 个位置 (其中 m 为新的字符数组的长度), 这样的蛮力算法时间复杂度为 $ O(n^2)$; 效率是很低下的, 因为当新字符数组的长度大于 1, 且旧字符出现次数大于 1 时, 有很多字符被后移了多次
为了能将需移动的字符一步到位后移, 可以考虑从数组尾部遍历匹配, 当匹配到旧字符时, 计算出需移动的字符要后移的位置, 这样可以将算法复杂度降低为 $ O(n)$

##### 实现
```
public class ReplaceSpaces {
    public static void main(String[] args) {
        StringBuffer buffer = new StringBuffer("We are happy.");
        System.out.println(replaceSpaces(buffer));
    }

    private static String replaceSpaces(StringBuffer buffer) {
        char space = ' ';
        char[] chars = new char[]{'%', '2', '0'};
        int count = 0;
        for (int i = 0; i < buffer.length(); i++) {
            if (buffer.charAt(i) == space) {
                count++;
            }
        }

        int leftIndex = buffer.length() - 1;
        buffer.append(new char[(chars.length - 1) * count]);
        int rightIndex = buffer.length() - 1;
        while (leftIndex > -1) {
            char c = buffer.charAt(leftIndex);
            if (c != space) {
                buffer.setCharAt(rightIndex--, c);
                leftIndex--;
            } else {
                for (int j = chars.length - 1; j > -1 ; j--) {
                    buffer.setCharAt(rightIndex--, chars[j]);
                }
                leftIndex--;
            }
        }
        return buffer.toString();
    }
}
```
