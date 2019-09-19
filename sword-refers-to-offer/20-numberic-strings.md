---
layout: post
title: "表示字符串的数字"
date: "2019-06-15"
description: "表示字符串的数字"
tag: [algorithm]
---

### 表示字符串的数字

#### 题目一
请实现一个函数用来判断字符串是否表示数值 (包括整数和小数); 例如: 字符串 `+100`, `5e2`, `-123`, `3.1416`, `-1E-16` 都表示数值, 但 `12e`, `1a3.14`, `1.2.3`, `+-5`, `12e+5.4` 都不是

##### 思路
表示数值的字符串遵循模式 `A[.[B][e|EC]` 或 `.B[e|EC]`, 其中 A 为数值的整数部分, B 为数值的小数部分, C 为数值的指数部分; 其中 A, C 都可能是以 `+\-` 开头的 0 ~ 9 的数位串, B 也是 0 ~ 9 的数位串, 但是不能以正负开头
- 先扫描整数部分
- 如果包含小数部分则接下来是 `.`, 再继续扫描小数部分
- 如果包含指数部分则接下来是 `e` 或者 `E`, 再继续扫描指数部分

##### 实现
```
import java.util.Arrays;

public class NumbericStrings {
    public static void main(String[] args) {
        // true
        System.out.println(numbericStrings("+100".toCharArray()));
        System.out.println(numbericStrings("5e2".toCharArray()));
        System.out.println(numbericStrings("-123".toCharArray()));
        System.out.println(numbericStrings("3.1416".toCharArray()));
        System.out.println(numbericStrings("-1E-16".toCharArray()));
        // false
        System.out.println(numbericStrings("12e".toCharArray()));
        System.out.println(numbericStrings("1a3.14".toCharArray()));
        System.out.println(numbericStrings("1.2.3".toCharArray()));
        System.out.println(numbericStrings("+-5".toCharArray()));
        System.out.println(numbericStrings("12e+5.4".toCharArray()));
    }

    private static boolean numbericStrings(char[] str) {
        if (str ==  null || str.length == 0) {
            return false;
        }

        int scanCount = scanInteger(str);
        // 如果出现 '.', 接下来是数字的小数部分
        if (scanCount < str.length && str[scanCount] == '.') {
            /*
             * 1. 小数可以没有整数部分, 如 .123 等于 0.123
             * 2. 小数点后可以没有数字, 如 233. 等于 233.0
             * 3. 小数点前后都可以有数字
             */
            if (scanCount + 1 < str.length) {
                char[] afterPoint = Arrays.copyOfRange(str, scanCount + 1, str.length);
                scanCount = scanCount + 1 + scanUnsignedInteger(afterPoint);
            }
        }
        // 如果出现 'e' 或者 'E', 接下来是数字的指数部分
        if (scanCount < str.length && (str[scanCount] == 'e' || str[scanCount] == 'E' )) {
            /*
             * 1. 当 e 或者 E 前面没有数字时, 整个字符串不能表示数字, 如 .e1, e1
             * 2. 当 e 或者 E 后面没有整数时, 整个字符串也不能表示数字, 如 12e, 12e+5.4
             */
            if (scanCount > 0 && scanCount + 1 < str.length) {
                char[] afterExponent = Arrays.copyOfRange(str, scanCount + 1, str.length);
                scanCount = scanCount + 1 + scanInteger(afterExponent);
            }
        }
        return scanCount == str.length;
    }

    private static int scanInteger(char[] str) {
        // +/-
        boolean hasSign = str.length > 0 && (str[0] == '+' || str[0] == '-');
        return hasSign ? str.length > 1
                ? 1 + scanUnsignedInteger(Arrays.copyOfRange(str, 1, str.length)) : 1
                : scanUnsignedInteger(str);
    }

    private static int scanUnsignedInteger(char[] str) {
        // 0~9
        for (int i = 0; i < str.length; i++) {
            if (str[i] < '0' || str[i] > '9') {
                return i;
            }
        }
        return str.length;
    }
}
```
