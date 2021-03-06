### 向量
数据结构是数据项的结构化集合, 其结构性表现为数据项之间的相互联系及作用, 也可以理解为定义于数据项之间的某种逻辑次序; 根据这种逻辑次序的复杂程度, 大致可以将各种数据结构划分为线性结构, 半线性结构, 非线性结构三大类; 在线性结构中, 各数据项按照一个线性次序构成一个整体; 最基本的线性结构统称为序列 (sequence), 根据其中数据项的逻辑次序与其物理存储地址的对应关系不同, 又可进一步的将序列区分为向量 (vector) 和列表 (list); 在向量中, 所有数据项的物理存放位置与其逻辑次序完全吻合, 此时的逻辑次序也称作秩 (rank); 而在列表中, 逻辑上相邻的数据项在物理上未必相邻, 而是采用间接定址的方式通过封装后的位置 (position) 相互引用

#### 从数组到向量

##### 数组
若集合 S 由 n 个元素组成, 且各元素之间具有一个线性次序, 则可将它们存放于起始地址 A, 物理位置连续的一段地址空间, 并统称作数组 (array), 可记作 $ A = \{a_0, a_1, ..., a_{n-1} \} $ 或者 $ A[0.n) = {A[0], A[1], ..., A[n-1]} $; 其中对于任何 $ 0 \leq i < j < n $, A[i] 都是 A[j] 的前驱 (predecessor), A[j] 都是 A[i] 的后继 (successor); 特别的, 对于任何 $ i \geq 1 $, A[i-1] 称作 A[i] 的直接前驱 (immediate predecessor), 对于任何 $ i \leq n -2 $, A[i+1] 称作 A[i] 的直接后继 (immediate successor); 任一元素的所有前驱构成其前缀 (prefix), 所有后继构成其后缀 (suffix); 另外, 若数组 A[] 存放空间的起始地址为 A, 且每个元素占用 s 个单位的空间, 则元素 A[i] 对应的物理位置为: $ A + i * s  $

##### 向量
按照面向对象思想中的数据抽象原则, 可以对以上数组结构做一般性推广, 使得其以上特性更具有普遍性; 向量 (vector) 就像线性数组的一种抽象和泛化, 也是具有线性次序的一组元素构成的集合 $ V = \{v_0, v_1, ..., v_{n-1} \} $, 其中元素分别由秩相互区分; 各元素的秩 (rank) 互异, 即可通过 r 来唯一确定 $ e = v_r $, 这是向量特有的元素访问方式, 称作 "循秩访问 (call-by-rank)"  
经过抽象后, 不再限定同一向量中的各元素都属于同一基本类型, 另外也不保证同时具有某一属性, 故不保证元素之间可以相互比较大小; 以下是定义的向量模板类

| 操作接口 | 功能 | 适用对象 |
| :--- | :--- | :--- |
| size() | 报告向量当前的规模 (元素总数) | 向量 |
| get(r) | 获取秩为 r 的元素 | 向量 |
| put(r, e) | 用 e 替换秩为 r 的元素 | 向量 |
| insert(r, e) | e 作为秩为 r 的元素插入, 原后继元素依次后移 | 向量 |
| remove(r) | 删除秩为 r 的元素, 返回该元素中存放的元素 | 向量 |
| disordered() | 判断所有元素是否已按非降序排序 | 向量 |
| sort() | 调整各元素的位置, 使之按非降序排序 | 向量 |
| find(e) | 查找等于 e 且秩最大的元素 | 向量 |
| search(e) | 查找目标元素 e, 返回不大于 e 且秩最大的元素 | 有序向量 |
| deduplicate() | 删除重复元素 | 向量 |
| uniquify() | 删除重复元素 | 有序向量 |
| traverse() | 遍历向量并统一处理所有元素, 处理方法由函数对象指定 | 向量 |

##### 操作实例
TODO

##### Vector 模板类
TODO

#### 构造与析构
向量结构在内部维护一个元素类型为 T 的私有数组 \_elem[]: 其容量由私有变量 \_capacity 表示, 有效元素的数量则由 \_size 表示

##### 默认构造方法
默认的构造方法是, 首先根据创建者指定的初始容量向系统申请空间, 以创建内部数组 \_elem[]; 若容量未指定, 则使用默认值 DEFAULT_CAPACITY; 忽略分配数组空间的时间, 共需常数时间

##### 基于复制的构造方法
```
template <typename T> void Vector<T>:: copyFrom(T const* A, Rank lo, Rank hi) {  // 以数组区间 A[lo, hi) 为蓝本复制向量
    _elem = new T[_capacity = 2 * (hi - lo)];  // 分配空间
    _size = 0;  // 规模清零
    while (lo < hi) {  // A[lo, hi) 内元素逐一复制至 _elem[0, hi - lo)
          _elem[_size++] = A[lo++];
    }
}
```
忽略开辟新空间所需时间, 运行时间正比于区间宽度, 即 $ O(hi - li) = o(\_size) $

#### 析构方法
TODO

#### 动态空间管理
内部数组占物理空间在向量生命周期内不允许调整, 则称为静态空间管理策略; 但由于容量固定, 则会存在某一时刻无法插入新的元素, 即所谓的上溢 (overflow); 向量实际规模与其内部数组容量的比值 (即 \_size / \_capacity), 称为装填因子 (laod factor), 是衡量空间利用率的重要指标

##### 可扩充向量
当数组 A[] 填充满后, 另外申请一个容量更大的数组, 并将原数组中的成员集体搬迁至新的空间, 此后可顺利的插入新元素而不导致溢出, 最后将原数组所占空间释放

##### 扩容
```
template <typename T> void Vector<T>::expand() {  // 向量空间不足时扩容
    if (_size < _capacity) return; // 尚未满员不必扩容
    if (_capacity < DEFAULT_CAPACITY) _capacity = DEFAULT_CAPACITY; // 不低于最小容量
    T* oldElem = _elem;
    _elem = new T[_capacity <<= 1]; // 容量加倍
    for (int i = 0; i < _size; i++)
        _elem[i] = oldElem[i]; // 复制原向量内容 (T 为基本类型, 或已重载复制操作符 "=")
    delete [] oldElem; // 释放原空间
}
```

##### 分摊分析
- 时间代价
每次由 n 到 2n 的扩容, 都需要花费 $ O(n) $ 的时间
- 分摊复杂度
将对可扩充向量的足够多次连续操作, 并将期间所消耗的时间, 分摊至所有的操作时间; 如此分摊平均至单次操作的时间成本, 称为分摊运行时间 (amoritized running time)
- O(1) 分摊时间
假定数组的初始容量为某一常数 N, 向量的初始规模也为 N --- 即将溢出; 假设此后需要连续进行 n 此 insert() 操作, n >> N; 定义如下函数
```
size(n) = 连续插入 n 个元素后向量的规模
capacity(n) = 连续插入 n 个元素后数组的容量
T(n) = 为连续插入 n 个元素而花费于扩容的时间
```
由于始终有 $ size(n) \leq capacity(n) < 2 * size(n) $, 即 $ capacity(n) = \Theta(size(n)) = \Theta(n) $; 容量以 2 为比例按指数增长, 在容量到达 capacity(n) 之前, 共做过 $ \Theta(log_2n) $ 次扩容, 每次扩容所需时间正比与当时容量, 且同样以 2 为比例按指数增长, 因此消耗于扩容的时间累计不超过: $ T(n) = 2N + 4N + 8N + ... + capacity(n) < 2 * capacity(n) = \Theta(n) $, 将其分摊到期间的连续 n 次操作, 单次操作所需的分摊运行时间为 $ O(1) $
- 其它扩容策略
TODO

##### 缩容
导致低效率的另一种情况是, 向量的实际规模远远小于内部数组的容量, 当装填因子低于某一阈值时, 称发生了下溢 (underflow); 尽管下溢不是必须要解决的问题, 但是在格外关注空间利用率的场景, 有必要考虑缩容
```
template <typename T> void Vector<T>::shrink() {  // 装填因子过小时压缩向量所占空间
    if (_capacity < DEFAULT_CAPACITY << 1) return; // 不致收缩到 DEFAULT_CAPACITY 以下
    if (_size << 2 > _capacity) retuen; // 以 25% 为界
    T* oldElem = _elem;
    _elem = new T[_capacity >>= 1]; // 容量减半
    for (int i = 0; i < _size; i++)  // 复制原向量内容
        _elem[i] = oldElem[i];
    delete [] oldElem; // 释放原空间
}
```
在实际应用中, 为了避免频繁的交替扩容或者缩容, 可以设置更低的阈值或者是 0 (即禁止缩容); 就单次扩容与缩容的操作而言, 所需时间高达 $ \Omega(n) $, 因此在对单次操作的执行速度极其敏感的应用场景, 以上策略并不适用, 其中缩容可以完全不予考虑

#### 常规向量
##### 直接引用元素
TODO

##### 置乱器
###### 置乱算法
```
template <typename T> void permute(Vector<T>& V) { // 随机置乱向量, 使各元素等概率出现在各位置上
    for (int i = V.size(); i > 0; i--)  // 自后向前
        swap(V[i-1], V[rand() % i]);  // V[i-1] 与 V[0,i) 中某一随机元素交换
}
```
###### 区间置乱接口
TODO

##### 判等器与比较器
从算法的角度来看, "判断两个对象是否相等" 与 "判断两个对象的相对大小" 都是至关重要的操作; 为了区别这两种操作, 在后续的叙述中把前者称之为 "比对" 操作, 后者称之为 "比较" 操作

##### 无序查找
###### 判等器
只支持比对, 不支持比较的向量, 称为无序向量
###### 顺序查找
在无序向量中查找任意元素 e 时, 由于没有更多的信息, 只有遍历所有的元素后才能得出查找的结论; 一次逐个比对的查找方式称作为顺序查找 (sequential search)
###### 实现
```
template <typename T> // 无序向量的顺序查找, 返回最后一个元素 e 的位置, 失败时返回 -1
Rank Vector<T>::find(T const& e, Rank lo, Rank hi) const {
    while ((lo < hi--) && (e != _elem[hi]));  // 从后向前顺序查找
    return hi;  // 若 hi < lo, 则意味着失败, 否则 hi 即命中元素的秩
}
```
###### 复杂度
最坏的情况下, 查找终止于首元素 \_elem[lo], 运行时间为 $ O(hi -lo) = O(n) $; 最好的情况下, 查找命中于末元素 \elem[hi -1], 仅需时间 $ O(1) $; 对于规模相同, 内部组成不同的输入, 渐进运行时间却有本质区别, 故此雷算法也称为输入敏感的 (input sensitive) 算法

##### 插入
###### 实现
```
template <typename T> Rank Vector<T>::insert(Rank r, T const& e) { // 将 e 作为秩为 r 元素插入
    expand(); // 若有必要需扩容
    for (int i = _size; i > r; i--)
        _elem[i] = _elem[i-1]; // 自后向前
    _elem[r] = e; // 插入元素
    _size++; // 更新容量
    return r; // 返回秩
}
```
###### 复杂度
时间主要消耗于后继元素的后移, 线性正比于后缀的长度, 故总体为 $ O(\_size - r + 1) = O(n) $

##### 删除
###### 区间删除: remove(lo, hi)
```
template <typename T> int Vector<T>::remove(Rank lo, Rank hi) { // 删除区间 [lo, hi]
    if (lo == hi)  return 0;  // 处于效率考虑, 单独做退化情况的处理
    while (hi < _size)
        _elem[lo++] = _elem[hi++]; // [hi, _size) 顺次向前移 hi - lo 个单元
    _size = lo; // 更新规模, 直接丢弃尾部区间 [lo, _size = hi) 区间
    shrink(); // 如有必要则缩容
    return hi - lo; // 返回被删除元素的数目
}
```
###### 单元素删除: remove(r)
```
template <typename T> int Vector<T>::remove(Rank r) {  // 删除向量中国秩为 r 的元素, 0 < r < size
    T e = _elem[r]; // 备份被删除的元素
    remove(r, r + 1); // 调用区间删除的算法, 等效于对区间 [r, r + 1) 的删除
    return e; // 返回被删除元素
}  
```
###### 复杂度
时间主要消耗于后继元素的前移, 线性正比于后缀的长度, 故总体为 $ O(\_size - hi + 1) = O(n) $
###### 错误以及意外处理
上述接口对输入有一定的限制和约定, 其中指定的待删除区间必须落在合法的范围内, 为此输入参数必须满足 $ 0 \leq lo \leq hi \leq \_size $  
一般的, 输入参数超出接口所约定合法范围的此类问题, 都属于典型的错误 (error) 或者意外 (excepion); 除了以注释的形式加以说明外, 还应该尽可能的对此类情况做更为周全的处理

##### 唯一化
视向量是否有序, 该功能有两种实现方式, 以下是无序向量唯一化的算法
###### 实现
```
template <typename T> int Vector<T>::deduplicate() {  // 删除无须向量中重复元素
    int oldSize = _size; // 记录原规模
    Rank i = 1; // 从 _elem[1] 开始
    while (i < _size) // 自前像后寻找预知雷同者 (至多一个)
        (find (_elem[i], 0 , i) < 0) ? i++ : remove(i); // 若无雷同者则继续考差其后继, 否则删除雷同者
    return oldSize - _size; // 向量规模变化量
}
```
###### 正确性
TODO
###### 复杂度
随着循环的进行, 当前元素的后继持续的严格减少, 经过 n - 2 步迭代后该算法必然终止, 这里所需的时间主要消耗于 find() 和 remove() 两个接口, 因此每步迭代所需时间为 $ O(n) $, 总体复杂度为 $ O(n^2) $

##### 遍历
###### 功能
为变量专门设置一个遍历接口 traverse(), 以修改或访问每个元素
###### 实现
```
template <typename T> void Vector<T>::traverse(void (* visit) (T&)) { // 借助函数指针机制
    for (int i = 0; i < _size; i++)
        visit(_elem[i]);
}

template <typename T> template <template VST> void Vector<T>::traverse(VST& visit) {  // 借助函数对象机制
    for (int i = 0; i < _size; i++)
        visit(_elem[i]);
}
```
###### 实例
TODO
###### 复杂度
遍历一次的总体时间复杂度为 $ O(n) $

#### 有序向量
若向量 S[0, n) 中所有元素不仅按线性次序存放, 而且其数值大小也按此序列单调分布, 则称为有序向量 (sorted vector); 与通常的向量一样, 有序向量依然不要求元素互异, 故通常约定其中的元素自前向后构成一个非降序序列, 即对任意 $ 0 \leq i < j < n $ 都有 $ S[i] \leq S[j] $

##### 比较器
除了与无序向量一样需要支持元素之间的 "判等" 操作, 有序向量的定义中实际还隐含了另一更强的先决条件: 即各元素之间必须能够比较大小

##### 有序性甄别
作为无序向量的特例, 有序向量可以沿用无序向量的查找算法, 然而得益于元素之间的有序性, 有序向量的查找, 唯一化等操作都可以更快的完成; 因此在实施此类操作前, 都有必须要判断当前向量是否已经有序
```
template <typename T> int Vector<T>::disordered() const { // 返回向量中逆序相邻元素对的总数
    int n = 0; // 计数器
    for (int i = 1; i < _size; i++) { // 逐一检查 _size - 1 对相邻元素
        if (_elem[i-1] > _elem[i])
           i++; // 逆序对计数
    }
    return n; // 当且仅当 n = 0 时向量有序
}
```

##### 唯一化
出于效率考虑, 为清除无序向量中的重复元素, 一般做法是首先将其转化为有序向量
###### 低效版
```
template <typename T> int Vector<T>::uniquify() { // 有序向量重复元素剔除算法 (低效版)
    int oldSize = _size;
    int i = 1;
    while (i < _size) // 从前向后逐一对比各相邻元素
        _elem[i - 1] == _elem[i] ? remove(i) : i++; // 若雷同则剔除后者, 否则转至后一元素
    return oldSize - _size; // 向量规模变换量
}
```
这里的运行时间主要消耗于 while 循环和 remove 操作, 总体的时间复杂度为 $ O(n) $
###### 改进思路
以上唯一化过程复杂度过高的根源是, 在对 remove() 接口的各次调用中, 同一元素可能作为后继元素向前移动多次, 且每次仅移动一个元素; 在有序向量中, 重复的元素必然前后紧邻, 可以批量的删除重读元素
###### 高效版
```
template <typename T> int Vector<T>::uniquify() { // 有序向量重复元素剔除算法 (高效版)
    Rank i = 0, j = 0; // 各对互异 "相邻" 元素的秩
    while (++j < _size) { // 注意扫描直至末元素
        if (_elem[i] != _elem[j])
            _elem[++i] = _elem[j]; // 发现不同元素时, 向前移至紧邻于前者右侧
    }
    _size = ++i; // 截除尾部多于元素
    shrink();
    retuin j - i; // 向量规模变化量
}
```
###### 复杂度
while 循环的每一步迭代, 仅需对元素数值做一次比较, 向后移动一到两个位置指针, 并至多向前复制一个元素, 故只需常数时间; 所以时间复杂度为 $ O(n) $

##### 查找
有序向量 S 中的元素已不再随机分布, 秩 r 是 S[r] 在 S 中按大小的相对位次, 位于 S[r] 前 (后) 的元素均不至于更大 (小); 当所有元素互异时, r 即使 S[r] 中小于 S[r] 的元素数; 一般的, 若小于, 等于 S[r] 的元素各有 i, k 个, 则该元素及其雷同元素应集中分布在 S[i, i + k]  
基于上述性质, 有序向量的查找操作可以更加高效的完成; 尽管在最坏的情况下, 无序向量的查找操作需要线性时间, 但有序向量的这一效率可以提高至 $ O(log_2n) $
```
template <typename T> Rank Vector<T>::search(T const& e, Rank lo, Rank hi) const { // 在有序向量的区间 [lo， hi) 内, 确定不大于 e 的最后一个节点的秩
    return (rand() % 2) ? binSearch(_elem, e, lo, hi) : fibSearch(_elem, e, lo, hi); // 随机选择使用二分查找或者 finbonacci 查找
}
```

##### 二分查找 (版本 A)
###### 减而治之
循秩访问的特点加上有序性, 使得有序向量的查找可将 "减而治之" 的策略运用于有序向量的查找; 假定任一元素 S[mi] = x 为界, 都可将区间分为三部分, 且根据此时的有序性必有: $ S[lo, mi) \leq S[mi] \leq S[mi, hi)$, 进一步处理如下
- 若 e < x, 则目标元素 e 若存在, 比属于左侧子区间 S[lo, mi), 故可深入其中继续查找
- 若 x < e, 若 e 存在, 必属于有侧子区间 S[mi, hi), 故也可深入其中继续查找
- 若 e = x, 则意味着已经在此命中, 故查找随即终止

也就是说, 每经过至多两次比较操作, 或者已经找到目标元素, 或者可以将查找问题简化为一个规模更小的新问题
###### 实现
```
// 二分查找算法 (版本 A): 在有序向量的区间 [lo, hi) 内查找元素 e, 0 <= lo <= hi <= _size
template <typename T> static Rank binSearch(T *t, T const& e, Rank lo, Rank hi) {
    while (lo < hi) { // 每步迭代可能要做两次判断, 故有三个分支
        Rank mi = (lo + hi) >> 1; // 以中心为轴点
        if (e < A[mi]) hi = mi; // 深入前半段 [lo, mi) 查找
        else if (A[mi] < e) lo = mi + 1; // 深入后半段 (mi, hi) 查找
        else return mi; // 在 mi 命中
    }
    return -1; // 查找失败
} // 有多个命中元素时, 不能保证返回秩最大者; 查找失败时, 简单的返回 -1, 不能指示失败的位置
```
###### 实例
TODO
###### 复杂度
随着迭代的不断深入, 有效的查找区间宽度将按 1/2 的比例以几何级数的速度递减, 经过至多 $ log_2(hi - lo) $ 步迭代后, 算法必然终止; 鉴于每步迭代仅需常数时间, 故总体复杂度不超过 $ O(log_2(hi - lo)) = O(log_2n) $

###### 查找长度
查找算法的整体效率也更主要取决于其中所执行的元素大小的比较操作, 即所谓的查找长度 (search length)

###### 成功查找长度
假定查找的目标按等概率分布, 则平均查找长度为: $ (4 + 3 + 5 + 2 + 5 + 4 + 6) = 29/7 = 4.14 $; 为了估计出一般情况下的成功查找长度, 不失一般性的, 仍在等概率条件下考查长度为 $ n = 2^k - 1 $ 的有序向量, 并将其对应的平均成功查找长度记作 $ c_{average}(k)$, 将所有元素对应的查找长度总和记作 $ C(k) = c_{average}(k) * (2^k - 1)$  
特别的, 当 k = 1 时向量长度 n = 1, 成功查找仅有一种情况, 故有边界条件:
$$ c_{average}(1) = C(1) = 2 $$  
采用递推分析法; 对于长度 $ n = 2^k - 1$  的有序向量, 每步迭代都有三种可能的分支: 经过 1 次成功的比较后, 转化为一个规模为 $ 2^{k-1} - 1 $ 的新问题; 经过 2 次失败的比较后, 终止于向量中的某一元素, 并确认在此处成功命中; 经过 1 次失败的比较另加 1 次成功的比较后, 转化为另一规模为 $ 2^{k-1} - 1 $ 的新问题; 根据以上递推分析的结论, 可得递推式如下:
$$
\begin{align}
C(k) & = [C(k - 1) + (2^{k-1} - 1)] + 2 + [C(k - 1) + 2 * (2^{k-1} - 1)] \\
     & = 2 * C(k - 1) + 3 * 2^{k-1} - 1
\end{align}
$$
若令: $ F(k) = C(k) - 3k * 2^{k-1} - 1 $  
则有:
$$
\begin{align}
F(1) & = -2 \\  
F(K) & = 2 * F(k - 1) = 2^2 * F(k - 2) = 2^3 * * F(k - 3) = ... \\
     & = 2^{k-1} * F(1) = -2^k
\end{align}
$$
于是:
$$
\begin{align}
C(k) & = F(k) + 3k * 2^{k-1} + 1 \\
     & = -2^k + 3k * 2^{k-1} + 1 \\
     & = (3k/2 - 1) * (2^k - 1) + 3k/2
\end{align}
$$
进而:
$$
\begin{align}
c_{average}(k) & = C(k) / (2^k - 1) \\
               & = 3k/2 - 1 + 3k/2/(2^k - 1) \\
               & = 3k/2 - 1 + O(\epsilon)
\end{align}
$$
也就是说, 如忽略末尾趋于收敛的波动项, 平均查找长度为: $ O(1.5k) = O(1.5 * log_2n) $

###### 失败查找长度
TODO

###### 不足
尽管二分查找算法 (版本 A) 即便在最坏的情况下也可保证 $ O(log_2n) $ 的渐进时间复杂度, 但就其常系数 1.5 而言仍然有改进的余地; 原因在于每一步迭代中确定左右分支, 分别需要做 1 次或者 2 次元素比较, 从而造成不同情况所对应的查找长度的不均衡;

##### Finbonacci 查找
###### 递推方程
二分法版本 A 最终求解所得的平均复杂度主要取决于 $ (2^{k-1} - 1) 和 2 * (2^{k-1} - 1) $ 这两项, 其中 $ 2^{k-1} $ 为子向量的宽度, 而系数 1 和 2 为算法深入前, 后子向量所需比较的次数; 二分法版本 A 存在的均衡缺陷, 根源在于这两项的大小不匹配; 基于这一点, 解决的思路不外乎以下两种
- 调整前后区域的宽度, 适当的加长 (缩短) 前 (后) 子向量
- 统一沿两个方法所需要执行的比较次数, 比如统一为一次

###### 黄金分割比
减治策略本身并不要求切分点 mi 必须居中, 不妨按黄金分割比来确定 mi; 为简化起见, 不妨设向量长度 n =  fib(k) - 1

###### 实现
```
// Finbonacci 查找算法 (版本 A): 在有序向量的区间 [lo, hi) 内查找元素 e, 0 <= lo <= hi <= _size
template <typename T> static Rank fibSearch(T* A, T const&e, Rank lo, Rank hi) {
    Fib fib(hi - lo); // 用 O(log_phi(n = hi - lo)) 时间创建 Fib 序列
    while (lo < hi) {
        while (hi - lo < fig.get()) fib.prev(); // 通过向前顺序查找
        Rank mi = lo + fib.get() - 1; // 确定轴点
        if (e < A[mi]) hi = mi; // 深入前半段 [lo, mi) 查找
        else if (A[mi] < e) lo = mi + 1; // 深入后半段 (mi, hi) 查找
        else return mi; // 在 mi 命中
    }
    return -1; // 查找失败
} // 有多个命中元素时, 不能保证返回秩最大者; 查找失败时, 简单的返回 -1, 不能指示失败的位置
```

###### 定性比较
TODO

###### 定量分析
将长度 $ n = fib(k) - 1 $ 的有序向量的平均成功查找长度记作 $c_{average}(k)$, 查找长度总和记作 $ C(k) = c_{average}(k) * (fib(k) - 1) $; $ c_{average}(2) = C(2) = 0, c_{average}(3) = C(3) = 2 $
同理, 可得递推公式如下:
$$
\begin{align}
C(k) & = [C(k - 1) + (fib(k-1) - 1)] + 2 + [C(k - 2) + 2 * (fib(k-2) - 1)] \\
     & = C(k - 2) + C(k - 1) + fib(k-2) + fib(k) - 1
\end{align}
$$
若令 $ F(k) = -C(k) + k * fib(k) + 1 $
则有:
$$
\begin{align}
F(0) & = 1 \\
F(1) & = 2 \\
F(k) & = F(k-1) + F(k-2) = fib(k+2)
\end{align}
$$
结合以上边界条件可解得
$$
\begin{align}
C(k) & = k * fib(k) - fib(k + 2) + 1 \\
     & = (k - \Phi^2) * fib(k) + 1 + O(\epsilon)
\end{align}
$$
其中, $ \Phi = (\sqrt{5} + 1) / 2 = 1.618 $, 于是有:
$$
\begin{align}
c_{average} & = C(k) / (fib(k) - 1) \\
            & = k - \Phi^2 + 1 + (k - \Phi^2) / (fib(k) - 1) + O(\epsilon) \\
            & = k - \Phi^2 + 1 + O(\epsilon)
\end{align}
$$
也就是说, 如忽略末尾趋于收敛的波动项, 平均查找长度为: $ O(k) = O(log_{\Phi}n) = O(log_{\Phi}n * log_2n) = O(1.44 * log_2n) $

##### 二分查找 (版本 B)
###### 从三分支到两分支
Finbonacci 查找算法通过采用黄金分隔点, 在一定程度程度上降低了时间复杂度的常系数; 实际还有另外一种更直接的方法, 即令左右分支的两项常系数同时为 1; 与二分查找算法的版本 A 基本类似, 不同之处是在每个切分点 A[mi] 处, 仅做一次元素比较; 具体的, 若目标元素小于 A[mi], 则深入前端子向量 A[lo, mi), 否则深入后端子向量 A[mi, hi) 继续查找

###### 查找
```
// 二分查找算法 (版本 B): 在有序向量的区间 [lo, hi) 内查找元素 e, 0 <= lo <= hi <= _size
template <typename T> static Rank binSearch(T *t, T const& e, Rank lo, Rank hi) {
    while (lo < hi) { // 每步迭代仅需做一次判断, 有两个分支, 成功查找不能提前终止
        Rank mi = (lo + hi) >> 1; // 以中心为轴点
        (e < A[mi]) ? hi = mi : lo = mi; // 经过比较后确定深入 [lo, mi) 或 [mi, hi)
    }
    return (e == A[lo]) ? lo : -1; // 查找成功时返回对应的秩, 否则统一返回 -1
} // 有多个命中元素时, 不能保证返回秩最大者; 查找失败时, 简单的返回 -1, 不能指示失败的位置
```

###### 性能
这一版本中, 只有在向量有效区间宽度为 1 个单元时算法才会终止, 而不能如版本 A 那样, 一旦命中就能直接返回; 因此最好情况下的效率有所倒退, 但是作为补偿, 最坏的情况下的效率相应的有所提高; 实际上无论是成功查找还是失败查找, 版本 B 各分支的查找长度更加接近, 故整体性能更加趋于稳定

###### 进一步要求
TODO

##### 二分查找 (版本 C)
###### 实现
```
// 二分查找算法 (版本 C): 在有序向量的区间 [lo, hi) 内查找元素 e, 0 <= lo <= hi <= _size
template <typename T> static Rank binSearch(T *t, T const& e, Rank lo, Rank hi) {
    while (lo < hi) { // 每步迭代仅需做一次判断, 有两个分支, 成功查找不能提前终止
        Rank mi = (lo + hi) >> 1; // 以中心为轴点
        (e < A[mi]) ? hi = mi : lo = mi + 1; // 经过比较后确定深入 [lo, mi) 或 (mi, hi)
    }
    return --lo; // 循环结束时, lo 为大于 e 元素的最小秩, 故 --lo 为不大于 e 元素的最大秩
} // 有多个命中元素时, 总能保证返回秩最大者; 查找失败时, 能够返回失败的位置
```

###### 正确性
TODO

#### 排序与下界
##### 有序性
从数据处理的角度看, 有序性在很多场合都能够极大的提高计算的效率

##### 排序及其分类
- 根据其处理数据的规模和存储的特点不同, 可以分为内部排序算法和外部排序算法
- 根据输入形式的不同, 排序算法也可以分为离线算法和在线算法
- 依赖的体系结构不同, 又可以分为串行和并行排序算法
- 根据排序算法是否采用随机策略, 又可以分为确定式和随即式

以下讨论排序算法为确定式的串行离线的内部排序算法

##### 下界
TODO

##### 比较树
TODO

##### 估计下界
TODO


#### 排序器

##### 统一入口
```
template <typename T> void Vector<T>::sort(Rank lo, Rank hi) { // 向量区间 [lo, hi) 排序
    switch(rand() % 5) { // 随机选取排序算法
         case 1: bubbleSort(lo, hi); break; // 起泡排序
         case 2: selectionSort(lo, hi); break; // 选择排序
         case 3: mergeSort(lo, hi); break; // 归并排序
         case 4: heapSort(lo, hi); break; // 堆排序
         default: quickSort(lo, hi); break; // 快速排序
    }
}
```

###### 起泡排序
```
template <typename T> void Vector<T>::bubbleSort(Rank lo, Rank hi) {
    while (!bubble(lo, hi--)); // 逐躺做扫描交换, 直至全序
}
```
###### 扫描交换
```
template <typename T> bool Vector<T>::bubble(Rank lo, Rank hi) { // 一趟扫描交换
    bool sorted = true; // 整体有序标志
    while (++ lo < hi) {
        if (_elem[lo - 1] > _elem[lo]) { // 若逆序
            sorted = false;  // 意味着整体未全序
            swap(_elem[lo - 1], _elem[lo]); // 通过交换使局部有序
        }
    }
    return sorted; // 返回有序标志
}
```

###### 重复性和稳定性
稳定算法的特征是: 重复元素之间的相对次序在排序前后保持一致; 即在将 A 向量转换为有序向量 S 之后, 设 A[i] 对应于 $ S[k_i] $, 对于 A 中每一对重复元素 A[i] = A[j] (相应地 $ S[k_i] = S[k_j] $), 都有 i < j 当且仅当 $ k_i < k_j $, 称该排序算法是稳定算法 (stable algorithm)

##### 归并排序
###### 历史与发展
归并排序是第一个可以在最坏的情况下依然保持 $ O(nlogn) $ 运行时间的确定性排序算法

###### 有序向量的二路归并
与起泡排序类似, 归并排序可以理解为通过反复调用所谓的二路归并 (2-way merge) 算法实现的; 所谓的二路归并, 就是将两个有序序列合并成为一个有序序列; 二路归并属于迭代式算法, 在每步迭代中, 只需比较两个待归并向量的首元素, 将小者取出追加到输出向量的末尾, 该元素在原向量中的后继则成为新的首元素; 如此反复, 直到某一向量为空, 最后将另一非空的向量整体接至输出向量的末尾

###### 分治策略
归并排序的主体结构属于典型的分治策略, 递归实现如下
```
template <typename T> void Vector<T>::mergeSort(Rank lo, Rank hi) { // 向量归并排序
    if (hi - lo < 2) return; // 单元素区间自然有序
    int mi = (lo + hi) / 2; // 以中点为界
    mergeSort(lo, mi);
    mergeSort(mi, hi);
    merge(lo, mi, hi); // 归并
}
```

###### 实例
TODO

###### 二路归并接口的实现
```
template <typename T> void Vector<T>::merge(Rank lo, Rank mi, Rank hi) { // 有序向量的归并
    T* A = _elem + lo; // 合并后的向量 A[0, hi -lo) = _elem[lo, hi)
    int lb = mi - lo;
    T* B = new T[lb]; // 前子向量 B[0, lb) = _elem[lo, mi)
    for (Rank i = 0; i < lb; B[i] = A[i++]); // 复制前子向量
    int lc = hi - mi;
    T* C = _elem + mi; // 后子向量 C[0, lc) = _elem[mi, hi)
    for(Rank i = 0, j = 0, k = 0; (j < lb) || (k < lc);) { // B[j] 和 C[k] 中的小者续至
        if ((j < lb) && (!(k < lc) || (B[j] <= C[k]))) A[i++] = B[j++];
        if ((k < lc) && (!(j < lb) || (C[k] < B[j]))) A[i++] = C[k++];
    }
    delete [] B; // 释放临时空间 B
} // 归并后得到完整的有序向量 [lo, hi)
```

###### 归并时间
TODO

###### 推广
TODO

###### 排序时间
TODO
