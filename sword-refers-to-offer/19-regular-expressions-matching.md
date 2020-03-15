---
layout: post
title: "正则表达式匹配"
date: "2019-06-14"
description: "正则表达式匹配"
tag: [algorithm]
---

### 正则表达式匹配

#### 题目一
请实现一个函数用来匹配包含 `'.'` 和 `'*'` 的正则表达式; 模式中的字符 `'.'` 表示任意一个字符, 而 `'*'` 表示它前面的字符可以出现任意次(含 0 次); 在本题中, 匹配是字符串的所有字符匹配整个模式; 例如, 字符串 "aaa" 与模式 "a.a" 和 "ab*ac*a" 匹配, 但与 "aa.a" 和 "ab*a" 均不匹配

##### 思路
当模式中的第二个字符不是 `'*'` 时
- 如果字符串中的第一个字符和模式中的第一个字符匹配, 那么在字符串和模式上都向后移动一个字符, 然后匹配剩余字符串和模式
- 如果字符串中的第一个字符和模式中的第一个字符不匹配, 则直接返回 false  
当模式中的第二个字符是 `'*'` 时
- 如果字符串中的第一个字符和模式中的第一个字符匹配时, 有多种不同的移动方式
  - 字符串向后移动一个字符, 模式可以保持模式不变向, 也可以后移动两个字符
- 如果字符串中的第一个字符和模式中的第一个字符不匹配
  - 模式可以向后移动两个字符, 表示模式的第一个字符和 `'*'` 被忽略

##### 实现
```Java
import java.util.Arrays;

public class RegularExpressionsMatching {
    public static void main(String[] args) {
        char[] str = new char[]{'a', 'a', 'a'};
        char[] pattern1 = new char[]{'a', '.', 'a'};
        char[] pattern2 = new char[]{'a', 'b', '*', 'a', 'c', '*', 'a'};
        char[] pattern3 = new char[]{'a', 'a', '.', 'a'};
        char[] pattern4 = new char[]{'a', 'b', '*', 'a'};

        System.out.println(regularExpressionsMatching(str, pattern1));
        System.out.println(regularExpressionsMatching(str, pattern2));
        System.out.println(regularExpressionsMatching(str, pattern3));
        System.out.println(regularExpressionsMatching(str, pattern4));
    }

    private static boolean regularExpressionsMatching(String str, String pattern) {
        if (str == null || pattern == null) return false;
        if (pattern.isEmpty()) return str.isEmpty();

        boolean match = (!str.isEmpty() && (str.charAt(0) == pattern.charAt(0) || pattern.charAt(0) == '.'));
        if (pattern.length() > 1 && pattern.charAt(1) == '*') {
            /**
             * 当 pattern 的第二个字符为 * 时, str.charAt(0) 与 pattern.charAt(0) 匹配与不匹配都可以将 pattern 向后移动两个字符
             * 当 pattern 的第二个字符为 * 时, str.charAt(0) 与 pattern.charAt(0) 时, 将 str 向后移一个字符
             */
            return isMatch(str, pattern.substring(2)) || (match && isMatch(str.substring(1), pattern));
        } else {
            /**
             * 当 pattern 的长度等于 1 时, 或者 pattern 的第二个字符不等于 * 时
             */
            return match && isMatch(str.substring(1), pattern.substring(1));
        }
    }
}
```
