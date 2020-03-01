### 面试中的算法

#### 如何判断链表中有环
快慢指针破循环

#### 最小栈实现
双栈实现, 其中一个栈保存已入栈元素的最小值

##### 如何求出最大公约数
- 辗转相除法
两个正整数 a 和 b (a > b), 它们的最大公约数等于 a 除以 b 的余数 c 与 b 之间的最大公约数  
- 更相减损术
两个正整数 a 和 b (a > b), 它们的最大公约数等于 a 减去 b 的差 c 与 b 之间的最大公约数

辗转相除法有一个问题是当两个数较大时做 a % b 的运算性能交叉式; 更相减损术的问题是运算次数较多; 以下结合更相减损术和移位运算给出一个更优解
- 当 a 和 b 均为偶数时, gcd(a, b) = 2 * gcd(a >> 1, b >> 1)
- 当 a 为偶数, b 为奇数时, gcd(a, b) = gcd(a >> 1, b)
- 当 a 为奇数, b 为偶数时, gcd(a, b) = gcd(a, b >> 1)
- 当 a 和 b 均为奇数时, 先利用更相减损术运算一次, gcd(a, b) = gcd(a - b, b), 此时 a - b 必然是偶数, 然后又可以进行移位运算

```Java
public static int gcd(int a, int b) {
    if (a == b) return a;

    if ((a & 1) == 0 && (b & 1) == 0) {
        return gcd(a >> 1, b >> 1) << 1;
    } else if ((a & 1) == 0 && (b & 1) != 0) {
        return  gcd(a >> 1, b);
    } else if ((a & 1) != 0 && (b & 1) == 0) {
        return  gcd(a, b >> 1);
    } else {
        return gcd(Math.abs(a - b), Math.min(a, b));
    }
}
```

#### 如何判断一个树是否是 2 的整数次幂
n & (n - 1)


##### 无序数组排序后的最大相邻差
利用计数排序和桶排序的思想

```Java
public static int getMaxSortedDistance(int[] array) {
    // 得到数组的极值
    int min = array[0], max = array[0];
    for (int i = 1; i < array.length; i++) {
        if (array[i] < min) min = array[i];
        if (array[i] > max) max = array[i];
    }

    int range = max - min;
    // 如果 max 与 min 相差不大于 1 则直接返回 max 与 min 的差值
    if (range < 2) return range;

    // 初始化桶
    int bucketNumber = array.length;
    Bucket[] buckets = new Bucket[bucketNumber];
    for (int i = 0; i < bucketNumber; i++) {
        buckets[i] = new Bucket();
    }

    // 遍历原始数组, 确定每个桶的最大最小值
    for (int i = 0; i < array.length; i++) {
        // 确定数组元素所归属的桶下标
        int number = ((array[i] - min) * (bucketNumber - 1) / range);
        if (buckets[number].getMin() == null || buckets[number].getMin() > array[i]) {
            buckets[number].setMin(array[i]);
        }
        if (buckets[number].getMax() == null || buckets[number].getMax() < array[i]) {
            buckets[number].setMax(array[i]);
        }
    }

    // 遍历桶找到最大差值
    int maxDistance = 0;
    int leftMax = buckets[0].getMax();
    for (int i = 1; i < buckets.length; i++) {
        if (buckets[i].min == null) continue;
        maxDistance = Math.max(maxDistance, buckets[i].getMin() - leftMax);
        leftMax = buckets[i].getMax();
    }
    return maxDistance;
}

public static class Bucket {
    private Integer min;
    private Integer max;

    public Integer getMin() {
        return min;
    }

    public void setMin(Integer min) {
        this.min = min;
    }

    public Integer getMax() {
        return max;
    }

    public void setMax(Integer max) {
        this.max = max;
    }
}
```

#### 如何用栈实现队列
双栈实现

#### 寻找全排列的下一个数
给出一个正整数, 找出这个正整数所有数字全排列的下一个数

思路如下 (字典序算法)
- 从后向前查看逆序区域, 找到逆序区域的前一位, 也就是数字置换的边界
- 让逆序区域的前一位和逆序区域中大于它的最小的数字交换位置
- 把原来的逆序区域转为顺序区域

```Java
public static int[] findNearestNumber(int[] numbers) {
    // 从后向前查找逆序边界
    int index = findTransferPoint(numbers);
    // 返回 0 表示无更大的整数
    if (index == 0) return null;

    // 将逆序区域前一位和逆序区域中刚刚大于它的数字交换位置
    exchange(numbers, index);
    // 将原来逆序区域反序
    reverse(numbers, index);
    return numbers;
}

private static int findTransferPoint(int[] numbers) {
    for (int i = numbers.length - 1; i > 0; i--) {
        if (numbers[i] > numbers[i - 1]) return i;
    }
    return 0;
}

private static void exchange(int[] numbers, int index) {
    int previous = numbers[index - 1];
    for (int i = numbers.length - 1; i >= index ; i--) {
        if (previous < numbers[i]) {
            numbers[index - 1] = numbers[i];
            numbers[i] = previous;
            break;
        }
    }
}

private static void reverse(int[] numbers, int index) {
    int left = index, right = numbers.length - 1;
    while (left < right) {
        swap(numbers, left++, right--);
    }
}

private static void swap(int[] array, int x, int y) {
    int tmp = array[x];
    array[x] = array[y];
    array[y] = tmp;
}
```

#### 删去 k 个数字后的最小值
给出一个整数, 从该整数中去掉 k 个数字, 要求剩下的数字形成的新整数尽可能小, 应该如何选取被去掉的数字?   

思路: 原整数的所有数字从左到右进行比较, 如果发现某一位数字大于它右面的数字, 那么在删除该数字后, 必然会使该数位值降低

```Java
public static String removeKDigits(String number, int k) {
    // 新整数的长度
    int length = number.length() - k;
    // 使用数组接收数字
    char[] chars = new char[number.length()];
    // 移除 k 次第一个右边大于左边的数字
    int index = 0;
    for (int i = 0; i < number.length(); i++) {
        char c = number.charAt(i);
        // 当栈顶数字大于遍历的当前数字时, 移除此数字
        while (index > 0 && chars[index - 1] > c && k > 0) {
            index--;
            k--;
        }
        chars[index++] = c;
    }
    // 找到第一个非零的下标
    int offset = 0;
    while (offset < length && chars[offset] == '0') offset++;
    return offset != length ? new String(chars, offset, length - offset) : "0";
}
```

#### 如何实现大整数相加
依次相加即可

#### 如何求解金矿问题
##### 题目
在很久很久以前, 有一位国王拥有 5 座金矿, 每座金矿的黄金储量是不同的, 需要参与挖掘的工人人数也不同, 例如有的金矿储量是 500KG 黄金, 需要 5 个工人来挖掘, 有的金矿储量是 200KG 黄金, 需要 3 个工人来挖掘  
如果参与挖矿的工人的总数是 10, 有五座金矿, 每座金矿的黄金储量和需要工人数为 `400KG/5, 500KG/5, 200KG/3, 300KG/4, 350KG/3`, 每座金矿要么全挖, 要么不挖, 不能排除一半人挖取一半的金矿, 要求用程序求出要想得到尽可能多的黄金应该选择挖取哪几座金矿

##### 思路
对于每座金矿都存在着 "挖" 和 "不挖" 两种选择, 这两种情况, 被称为全局问题的两个最优子结构
- 假设最后一座金矿不挖, 那么问题则被简化为 10 个工人在前 4 个金矿中做出最优选择
- 假设最后一座金矿挖, 那么问题则被简化为 7 个工人在前 4 个金矿中做出最优选择

##### 动态规划与状态转移方程
就这样把问题一分为二, 二分为四, 一直把问题简化成在 0 个金矿或 0 个工人时的最优选择, 这个收益结果显然是 0, 也就是问题的边界; 这也是动态规划的要点: **确定全局最优解和最优子结构之间的关系, 以及问题的边界**; 这种关系用数学公式来表达的话, 就叫做状态转移方程  
我们把金矿数量设为 n, 工人数量设为 w, 金矿的含金量设为数组 g[], 金矿所需开采人数设为数组 p[], 设 F(n, w) 为 n 个金矿 w 个工人的最优收益函数  
$$
        F(n, w) =
        \begin{cases}
        0 (n = 0 || w = 0),  & \text{问题边界, 金矿数为 0 或工人数为 0的情况} \\
        F(n - 1, w) (n >= 1, w < p[n - 1]), & \text{当所剩工人不够挖掘当前金矿时, 只有一种最优子结构} \\
        max(F(n - 1, w), F(n - 1, w - p[n - 1] + g[n - 1])), & \text{在常规情况下, 具有两种最优子结构, 即挖当前金矿或不挖当前金矿}
        \end{cases}
$$

##### 代码实现
```Java
public static int getBestGoldMining(int w, int n, int[] p, int[] g) {
    if (w == 0 || n == 0) return 0;
    if (w < p[n - 1]) {
        return getBestGoldMining(w, n - 1, p, g);
    } else {
        return Math.max(getBestGoldMining(w, n - 1, p, g),
                getBestGoldMining(w - p[n - 1], n - 1, p, g) + g[n - 1]);
    }
}
```
以上代码实现相当简单, 但由于部分子问题被重复求解, 算法的时间复杂度为 $O(2^n)$

##### 优化时间复杂度
为了避免重复求解, 需要将实现从自顶向下的求解过程改为自底向上的求解过程, 并在过程中保存中间结果避免重复调用; 这里我们使用二维数组保存中间数据, 并利用状态转移方程填充值

| - | 1 人 | 2 人 | 3 人 | 4 人 | 5 人 | 6 人 | 7 人 | 8 人 | 9 人 | 10 人 |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 400KG/5| 0 | 0 | 0 | 0 | 400 | 400 | 400 | 400 | 400 | 400 |
| 500KG/5| 0 | 0 | 0 | 0 | 500 | 500 | 500 | 500 | 500 | 900 |
| 200KG/3| 0 | 0 | 200 | 200 | 500 | 500 | 500 | 700 | 700 | 900 |
| 300KG/4| 0 | 0 | 200 | 300 | 500 | 500 | 500 | 700 | 800 | 900 |
| 350KG/3| 0 | 0 | 350 | 350 | 500 | 550 | 650 | 850 | 850 | 900 |

```Java
public static int getBestGoldMining(int w, int[] p, int[] g) {
    int[][] table = new int[g.length + 1][w + 1];
    // 填充表格
    for (int i = 1; i <= g.length; i++) {
      for (int j = 1; j <= w; j++) {
          if (j < p[i - 1]) {
              table[i][j] = table[i - 1][j];
          } else {
              table[i][j] = Math.max(table[i - 1][j],
                      table[i - 1][j - p[i - 1]] + g[i - 1]);
          }
      }
    }
    // 返回最后一个格子的值
    return table[g.length][w];
}
```

##### 空间复杂度优化
在中间结果中, 除了第一行数据之外, 每一行的结果都是由上一行数据推导出来的, 例如 `4 个金矿/9 个工人` 的最优结果, 是由它的两个最优子结构也就是 `3 个金矿/5 个工人` 和 `3 个金矿/9 个工人`, 这两个最优子结构都位于它的上一行  
所以程序中并不需要保存整个表格, 无论金矿有多少座, 只用保存一行数据即可; 再计算下一行时, 需要从右向左计算 (因为人数多的情况依赖人数少的情况), 把旧数据一个一个替换掉

```Java
public static int getBestGoldMining(int w, int[] p, int[] g) {
    int[] result = new int[w + 1];
    // 填充
    for (int i = 1; i <= g.length; i++) {
        for (int j = w; j >= 1; j--) {
            if (j >= p[i - 1]) {
                result[j] = Math.max(result[j], result[j - p[i - 1]] + g[i - 1]);
            }
        }
    }
    // 返回最后一个格子的值
    return result[w];
}
```

#### 寻找缺失的整数
在一个无序数组中有 99 个不重复的正整数, 范围是 1 ~ 100, 唯独缺少 1 个 1 ~ 100 之间的整数, 如何找出这个缺失的整数?

两种思路
- 等差数列求和减去出现的数字
- 1 ~ 100 异或出现的数字
