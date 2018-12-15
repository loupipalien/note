## 串
### 目录
- [定义与术语](#定义与术语)
- [ADT](ADT)
- [串匹配](#串匹配)
- [算法评测](#算法评测)
- [BF (Brute Force) 算法](#bf-brute-force-算法)
- [KMP (Knuth-Morris-Pratt) 算法](#kmp-knuth-morris-pratt-算法)
- [BM (Boyer-Moore) 算法](#bm-boyer-moore-算法)
- [各种算法性能比较(BF, KMP, BM)](#各种算法性能比较-bf-kmp-bm)
- [KR (Karp-Rabin) 算法](#kr-karp-rabin-算法)

### 定义与术语
定义: 由来自字母表 $\Sigma$ 的字符所组成的**有限序列**
$$ S = a_0, a_1, a_2, ... , a_{n-1} \in \Sigma^\ast $$

术语:
- 相等: S[0, n) = T[0, m), 长度相等 (n = m), 且对应的字符均相同 (S[i] = T[i])
- 子串: S.substr(i, k) = S[i, i + k), 0 <= i <= n, 0 <= k; 即从 S[i] 起的连续 k 个字符
- 前缀: S.prefix(k) = S.substr(0, k) = S[0, k), 0 <= k <= n; 即 S 中最靠前的 k 个字符
- 后缀: S.suffix(k) = S.substr(n - k, k) = S[n - k, n), 0 <= k <= n; 即 S 中最靠后的 k 个字符
- 联系: S.substr(i, k) = S.prefix(i + k).suffix(k)
- 空串: S[0, n = 0), 也是任何串的子串, 前缀, 后缀

### ADT
|接口|说明|
|-|-|
|length()|返回串的长度|
|charAt(i)|返回串中第 i 个字符|
|substr(i, k)|返回串中第 i 个字符到第 k 个字符的子串, i 闭 k 开|
|prefix(k)|返回串中前 k 个字符的子串|
|suffix(k)|返回串中后 k 个字符的子串|
|concat(T)|将串 T 连接在原串之后|
|equal(T)|判定 T 是否与原串相等|
|indexOf(P)|返回原串中第一个子串 P 的起始下标|

### 串匹配
记文本串为 T, 模式串为 P; n = |T| 和 m = |P|, 通常有 n >> m >> 2
模式匹配(Pattern Matching):
- detection: P 是否出现
- location: 首次在哪里出现
- counting: 共出现几次
- enumeration: 各出现在哪里

### 算法评测
- 随机 T + 随机 P ? 不妥!
以 $\Sigma = \lbrace 0, 1 \rbrace^\ast$ 为例, | {长度为 m 的 P} | = $2^m$, | {长度为 m 且在 T 中出现的 P} | = n - m + 1 < n
匹配成功的概率 = n / $2^m$ << 100000 / $2^100$ < $2^{-25}$,如此无法对算法做充分的测试
- 随机 T, 对成功或失败的匹配分别测试
成功: 在 T 中, 随机取出长度为 m 的子串作为 P, 分析平均复杂度
失败: 采用随机的 P, 统计平均复杂度

### BF (Brute-Force) 算法
思想: 自左向右以字符为单位, 依次移动模式串, 指导在某个位置发现匹配  

#### BF 实现
  - 版本一
  ```
  int match(char * P, char * T) {
      int n = (int)strlen(T), i = 0; //文本串指针
      int m = (int)strlen(P), j = 0; //模式串指针
      while(j < m && i < n) {
        if(T[i] == P[j]) {
          i++; j++; //若匹配则携手共进
        } else {
          j = 0; i = i - j + 1; // 不匹配则P复位, T回退
        }
      }
      return i - j;// 当i = n, j < m 时返回值大于n - m则意味着匹配失败
  }
  ```
  - 版本二
  ```
  int match(char * P, char * T) {
      int n = (int)strlen(T), i = 0; // T[i]与P[0]对齐
      int m = (int)strlen(P), j; //T[i + j]与P[j]对齐
      for(i = 0; i < n - m + 1; i++) {
        for(j = 0; j < m; j++) {
          if(T[i + j] != P[j]) break; // 若失配则P整体右移一个字符
        }
        if(m <= j) break; // 找到匹配子串
      }
      return i; // 返回值大于n - m则意味着匹配失败
  }
  ```

#### 时间复杂度:
- 最好情况: 只经过依次比对即可确定匹配; 比对 m 次, 复杂度为 O(m)
- 最坏情况: 每轮比对至 P 的末字符, 且反复如此; 比对 m * (n - m + 1), 因为一般地有 m << n, 故复杂度为 O(n * m)
注意: $| \Sigma |$ 越小, 最坏情况出现的概率越高; m 越大, 最坏情况的后果更加严重; 当 $| \Sigma |$ 越大, 匹配复杂度越可能降至 O(n)

### KMP (Knuth-Morris-Pratt) 算法
- 蛮力算法低效原因: 当 T[i] 与 P[j] 失配时, T 回退 P复位之后, 此前 T[i] 中比对匹配的字符将再次参与比对  
- 失配时但有局部匹配成功: 当 T[i] 与 P[j] 失配时, 已经掌握了 T[i - j, i) 的信息, 即 T[i - j, i) = P[0, j)  
- 失配时的局部匹配可为后续匹配利用: 当 T[i] 与 P[j] 失配时, 找到 P[0, j) 子串中最大自匹配的真前缀和真后缀, 记为 P[0,t) = P[j-t,j), 匹配串 P 向前滑动 j - t 个字符, 重置 j = t 并开始下一轮比对, 继续从比较 T[i] 和 P[j]  
- 当 T[i] 与 P[j] 失配时如何找到 t 值: 因为 t 为 P[0, j) 串中最大自匹配的真前缀和真后缀的长度, 即 t 是关于 j 为自变量的函数, t 的值仅与匹配串 P 有关, 那么在匹配之前即可计算出 t = next(j)  
- 最长自匹配(快速右移 + 避免回溯): 对于任意 j, N(P, j) = {0 <= t < j | P[0, t) = P[j-t, j)}, 即 t = next(j) = max(N(P, j)); 当 j = 0, 即 N(P, j) = $\emptyset$, t = next(j) = -1;  

#### NEXT 递推
已知 next[j] 如何得到 next[j + 1]:  
next[j + 1] <= next[j] + 1, 当且仅当 P[j] == P[next[j]]  
next[j + 1] 的候选者: 1 + next[j] -> 1 + next[next[j]] -> ... -> 1 + next[0] = 0  

##### 构造 next 表
```
int * buildNext(char * P) {
    int m = (int)strlen(P), j = 0; // 主串指针
    int * next = new int[m]; // next 表
    int t = next[0] = -1; // 模式串指针(P[-1] 通配符)
    while(j < m - 1) {
      if(0 > t || P[j] == P[t]) {
        next[++j] = ++t; // j++; t++; next[j] = t;
      } else {
        t = next[t];
      }
    }
    retunr next; // 返回 next 表
}
```

#### KMP 实现:  
```
int match(char * P, char * T) {
    int * next = buildNext(P); //根据匹配串构造next表
    int n = (int)strlen(T), i = 0; //文本串指针
    int m = (int)strlen(P), j = 0; //模式串指针
    while(j < m && i < n) {
      if(0 > j || T[i] == P[j]) {
        i++; j++; //若匹配则携手共进
      } else {
        j = next[j]; // 不匹配则P右移, T不回退
      }
    }
    delete [] next; // 释放next表
    return i - j;
}
```

#### 性能分析(分摊精准分析)
- 当匹配时: i++; j++; i, j 同时加 1, 故 k 恰好加 1
- 当失配时: j = next[j]; i 不变, j 至少减 1, 故 k 至少加 1

i 可看作时比对成功的次数, i - j 可看作比对失败次数的上界, 则令 k = 2 * i - j <= 2(n - 1) - (-1) = 2n - 1 = O(n), 即是迭代步数的上界  
buildNext的时间复杂度是匹配串长度 O(m - 1) = O(m), 即时间复杂度为 O(n) + O(m) = O(n + m)  

#### 不足与改进
- 不足: 只利用了自相似的经验, 未利用不相同的教训
T: 0 0 0 1 0 0 0 0 1
P: 0 0 0 0 1
N: -1 0 1 2 3

- 改进: 利用不相同的教训
```
int * buildNext(char * P) {
    int m = (int)strlen(P), j = 0; // 主串指针
    int * next = new int[m]; // next 表
    int t = next[0] = -1; // 模式串指针(P[-1] 通配符)
    while(j < m - 1) {
      if(0 > t || P[j] == P[t]) {
        j++; t++; N[j] = P[j] != P[t] ? t : N[t]; // 如果获取的 P[j] = P[next[j]] 则跳过
      } else {
        t = N[t];
      }
    }
    retunr next; // 返回 next 表
}
```

### BM (Boyer-Moore) 算法
- 判定失配(只要有一个字符不等)的概率 VS 判定匹配(所有字符都相等)的概率
  - 失配时: 局部多次成功一次失败
  - 匹配时: 整体多次失败一次成功
- 加速失败: 尽可能的快速的低成本的排除掉无效的对齐位置(回想KMP)
- 以终为始: 自后向前, 自右向左比对, 因为越靠后的失配越能排除更多的不匹配, 从而带来更多的价值

#### BC (Bad-Character)
- 发现一个坏字符时, 如何移动匹配串  
  - 一般情况(在 P[0, j) 中存在一个字符 X, 且 P(j,m) 中不存在字符 X): shift = j - bc[X] = j - rank[X]
  - 特殊情况(在 P[0, j) 中存在多个字符 X, 且 P(j,m) 中不存在字符 X): shift = j - bc[X] = j - max(rank[X])
  - 特殊情况(在 P[0, j) 中不存在字符 X, 且 P(j,m) 中不存在字符 X): shift = j + 1
  - 特殊情况(在 P[0, j) 中存在字符 X, 且 P(j,m) 中也存在字符 X): shift = 1

位移量仅取决于失配位置 j, 以及失配字符 X 在 模式串 P 中的秩, 而与 T 和 i 无关

##### 构造 bc 表
```
int * buildBC(char * P) {
    int * bc = new int[256]; // bc表与字母表等长
    for(int j = 0; j < 256; j++) {
        bc[j] = -1; // 初始化(统一指向通配符)
    }
    for(int m = (int)strlen(P), j = 0; j < m; j++) {
      bc[P[j]] = j; // 自左向右扫描刷新 P[j] 出现的位置记录(画家算法: 后来覆盖以往)
    }
    return bc;
}
```
空间 =  | bc[] | = O(| $\Sigma$ |) = O(S); 时间 = O(| $\Sigma$ | + m) = O(S + m), 可改进至 O(m)

#### 时间复杂度
- 最好情况: O(n / m), 单次比对失败概率越大则越适用, 即 | $\Sigma$| 越大, 常用于 Ctrl + F 的实现
T: x x x x 1 x x x x 1 ... x x x x 1
P: 0 0 0 0 0
- 最差情况: O(n * m), 退化为蛮力算法, 因为只借鉴教训, 没有利用经验
T: 0 0 0 0 0 0 0 0 0 0 ... 0 0 0 0 0
P: 1 0 0 0 0

#### GC (Good-Suffix)
当 T [i + j] 与 P[j] 失配, 有 U = P(j, m) = T(i + j, n), 常意味着有一个好后缀,
- 当出现一个坏字符时, 如何移动匹配串
- 一般情况(P[0, j) 中存在一个子串 V(k, k + m -j) = U, P[k] 不等于 P[j]): shift = j - k = gs[j]
- 特殊情况(P[0, j) 中存在多个子串 V(k, k + m -j) = U, P[k] 不等于 P[j]): shift = j - max(k) = gs[j]
- 特殊情况(P[0, j) 中不存在 V(k, k + m -j) = U, 或 P[k] 等于 P[j]): 移动会越过 T[i + j], 且在 P 中找是否有 T[i + m -j - t, t) = P[0, t), 则 shift = m - t = gs[j]

位移量仅取决于 j 和 P 本身, 而与 T 和 i 无关
##### 构造 gc 表
MS[] -> ss[] -> gs[]
MS[j]: P[0,j) 的所有后缀中, 与 P 的某一后缀匹配的最长者
ss[j] = |MS[j]| = max{0 <= s < j + 1 | P(j-s, j] = P[m-s, m)}
- 若 ss[j] = j + 1, 则对于任一字符 P[i] (i < m - j - 1), m - j - 1 必是 gs[i] 的一个候选
- 若 ss[j] <= j, 则对于任一字符 P[i] (i < m - ss[j] - 1), m - j - 1 必是 gs[m = ss[j] - 1] 的一个候选

### 各种算法性能比较 (BF, KMP, BM)
|#|最好情况|最差情况|影响因素|
|-|-|-|-|
|BF|O(n + m)|O(n * m)| $\Sigma$ 的规模大小 |
|KMP|O(n + m)|O(n + m)| - |
|BF|O(n + m)|O(n / m)| - |

### KR (Karp-Rabin) 算法
逻辑系统的符号, 表达式, 公式, 命题, 定理, 公理等均可以不同自然数(素数序列)标识  
素数序列: p(k) = 第 k 个素数: 2, 3, 5, 7, 11, ...    
每个有限维的自然数向量, 唯一对应于一个自然数  
$$ <a_1, a_2, ... , a_n> ~ p(1)^{1 + a_1} * p(2)^{1 + a_2} * ... * p(n)^{1 + a_n} $$  


串亦是数 -> 数位溢出 -> 散列压缩 -> 解决冲突 -> 快速计算指纹
