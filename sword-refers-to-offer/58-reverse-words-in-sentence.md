---
layout: post
title: "反转字符串"
date: "2019-07-23"
description: "反转字符串"
tag: [algorithm]
---

### 翻转字符串

#### 题目一: 翻转单词顺序
输入一个英文句子, 翻转句子中单词的顺序, 但单词内字符的顺序不变; 为简单起见, 标点符号和普通字母一样处理; 例如输入字符串 "I am a student.", 则输出 "student. a am I"

##### 思路
TODO

##### 实现
```
import java.util.Arrays;
import java.util.Collections;
import java.util.List;

public class Solution {

    public static void main(String[] args) {
        String str = "student. a am I";
        System.out.println(reverseWordsInSentence(str));
    }

    public static String reverseWordsInSentence(String str) {
        if (str == null || str.length() == 0) {
            return "";
        }

        List<String> list = Arrays.asList(str.split(" "));
        Collections.reverse(list);
        return String.join(" ", list);
    }
}
```

#### 题目二: 左旋转字符串
字符串的左旋转操作是把字符串前面的若干个字符转移到字符串的尾部; 定义一个函数实现字符串左旋转操作的功能; 比如: 输入字符串 "abcdefg" 和数字 2, 该函数将返回左旋转两位得到结果 "cdefgab"

##### 思路
TODO

##### 实现
```
public class Solution {

    public static void main(String[] args) {
        String str = "abcdefg";
        System.out.println(leftRotateString(str, 2));
    }

    public static String leftRotateString(String str,int n) {
        if (str == null || str.length() == 0) {
            return "";
        }
        n = n % str.length();
        return str.substring(n) + str.substring(0, n);
    }
}
```
