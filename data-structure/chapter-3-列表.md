### 列表
列表与向量同属序列结构的范畴, 其中的元素也构成一个线性逻辑次序, 但与向量极为不同的是, 元素的物理地址可以任意  
为保证对列表元素的可行性, 逻辑上互为前驱和后继的元素之间, 应维护某种索引关系, 这种索引关系可抽象的理解为索引元素的位置 (position), 故列表元素是 "循位置访问 (call-by-position)" 的: 也可形象来的理解为通往被索引元素的链接 (link), 故也称作 "循链接访问 (call-by-link)"

#### 从向量到列表
##### 从静态到动态
数据结构支持的操作, 通常无非静态和动态两类: 前者仅从中获取信息, 后者则会数据结构的局部甚至整体

##### 由秩到位置
改用动态存储策略后, 在提高动态操作效率的同时, 却又不得不舍弃原静态存储策略中循秩访问的方式, 从而造成静态操作性能的下降; 所以数据结构的访问方式, 应与其存储策略相一致

##### 列表
与向量一样, 列表也是由具有线性逻辑次序的一组元素构成的集合:
$$ L = \{a_0, a_1, ..., a_{n-1}\} $$
列表是链表结构的推广, 其中的元素称作节点 (node), 分别由特定的位置或链接指代

#### 接口
作为列表的基本组成单位, 列表节点除需保存数据项, 还应记录其前驱和后继的位置, 故需将这些信息即相关操作组成列表节点对象, 参与列表的构建

##### 列表节点
###### ADT 接口

| 操作接口 | 功能 |
| :--- | :--- |
| data() | 当前节点所存数据对象 |
| pred() | 当前节点前驱节点的位置 |
| succ() | 当前节点后继节点的位置 |
| insertAsPred(e) | 插入前驱节点, 存入被引用对象 e, 返回新节点对象 |
| insertAsSucc(e) | 插入后继节点, 存入被引用对象 e, 返回新节点对象 |

###### ListNode 模板类
TODO

##### 列表
###### ADT 接口

| 操作接口 | 功能 | 适用对象 |
| :--- | :--- | :--- |
| size() | 报告列表的当前规模 (节点总数) | 列表 |
| first, last() | 返回首, 末节点 | 列表 |
| insertAsFirst, insertAsLast(e) | 将 e 作为首, 末节点插入 | 列表 |
| insertA, insertB(p, e) | 将 e 当作节点 p 的直接后继, 前驱插入 | 列表 |
| remove(p) | 删除位置 p 处的节点, 返回其数值 | 列表 |
| disordered() | 判断所有节点是否已按非降序排序 | 列表 |
| sort() | 调整个节点位置, 使之按非降序排序 | 列表 |
| find(e) | 查找目标元素 e, 失败时返回 NULL | 列表 |
| search(e) | 查找目标元素 e, 返回不大于 e 且秩最大的节点 | 有序列表 |
| deduplicate() | 删除重复节点 | 列表 |
| uniquify() | 删除重复节点 | 有序列表 |
| traverse() | 遍历并统一处理所有节点, 处理方法由函数对象指定 | 列表 |

###### List 模板类
TODO

#### 列表
##### 头, 尾节点
List 对象的内部组成及逻辑结构如下, 其中私有的头节点 (header) 和尾节点 (trailer) 始终存在, 但对外并不可见; 对外部可见的数据节点如果存在, 则其中的第一个和最后一个节点分别称作首节点 (first node) 和末节点 (last node); 就颞部结构而言, 头节点紧邻与首节点之前, 尾节点紧邻于末节点之后, 这类经封装之后从外部不可见的节点, 称作哨兵节点 (sentinel node)
```
                      --------------------------------------------------------
---------- succ       |  ---------  succ      visble              --------   |  succ      ------------
| header | -------->  |  | first |  --------------------------->  | last |   |  ------->  | trailer |
| node   | <--------  |  | node  |  <---------------------------  | node |   |  <-------  | node     |
---------       pred  |  ---------            list          pred  --------   |      pred  ------------
                      --------------------------------------------------------
```

##### 默认构造方法
```
template <typename T> void List<T>::init() {
    header = new ListNode<T>; // 创建头哨兵节点
    trailer = new ListNode<T>; // 创建尾哨兵节点
    header->succ = trailer;
    header->pred = NULL;
    trailer->pred = header;
    trailer->succ = NULL;
    _size = 0; // 记录规模
}
```

##### 由秩到位置的转换
```
// 重载下标操作符, 以通过秩直接访问列表节点 (虽方便, 但效率低, 慎用)
template <typename T> T& List<T>::operator[](Rank r) const {
    ListNodePosi(T) p = first(); // 从首节点出发
    while(0 < r--) p = p->succ; // 顺数第 r 个节点
    return p->data; // 目标节点, 返回其中所存元素
}
```

##### 查找
###### 实现
对于整体和区间查找, 重载了操作接口 find(e) 和 find(e, n, p), 前者作为特例可以直接调用后者
```
// 在无序列表内节点 p (可能是 trailer) 的 n 个真前驱中, 找到等于 e 的最后者
template <typename T> ListNodePosi(T) List<T>::find(T const& e, int n, ListNodePosi(T) p) const {
    while (0 < n--) // (0 <= n <= rank(p) < _size) 对于 p 的最近的 n 个前驱, 从右至左
        if (e = (p = p->pred)->data) return p; // 逐个比对, 直至命中或范围越界
    return NULL; // p 越出左边界, 意味着区间内不含 e, 查找失败
} // 失败时返回 NULL
```

###### 复杂度
线性正比于查找区间的宽度, 时间复杂度为 $ O(n) $

##### 插入
###### 接口
```
template <typename T> ListNodePosi(T) List<T>::insertAsFirst(T const& e) {
    _size++;
    return header->insertAsSucc(e); // e 当作首节点插入
}

template <typename T> ListNodePosi(T) List<T>::insertAsLast(T const& e) {
    _size++;
    return trailer->insertAsPred(e); // e 当作末节点插入
}

template <typename T> ListNodePosi(T) List<T>::insertA(ListNodePosi(T) p, T const& e) {
    _size++;
    return p->insertAsSucc(e); // e 当作 p 的后继插入 (After)
}

template <typename T> ListNodePosi(T) List<T>::insertB(ListNodePosi(T) p, T const& e) {
    _size++;
    return p->insertAsPred(e); // e 当作 p 的前驱插入 (Before)
}
```

###### 前插入
```
// 将 e 紧靠当前节点之前插入于当前节点所属列表 (设有哨兵头节点 header)
template <typename T> ListNodePosi(T) ListNode<T>::insertAsPred(T const& e) {
    ListNodePosi(T) x = new ListNode(e, pred, this); // 创建新节点
    pred->succ = x;
    pred = x; // 设置正向链接
    return x; // 返回新节点的位置
}
```

###### 后插入
```
// 将 e 紧靠当前节点之后插入于当前节点所属列表 (设有哨兵头节点 trailer)
template <typename T> ListNodePosi(T) ListNode<T>::insertAsPred(T const& e) {
    ListNodePosi(T) x = new ListNode(e, this, succ); // 创建新节点
    succ->pred = x;
    succ = x; // 设置逆向链接
    return x; // 返回新节点的位置
}
```

###### 复杂度
前插入和后插入都可以在常数时间内完成, 故时间复杂度为 $ O(1) $

##### 基于复制的构造
###### copyNodes()
尽管提供了多种形式以允许对原列表的整体或局部复制, 但实质大同小异
```
// 列表内部方法: 复制列表中自 p 位置起的 n 项
template <typename T> void List<T>::copyNodes(ListNodePosi(T) p, int n) { // p 合法, 且至少有 n - 1 个后继
    init(); // 创建头尾节点并初始化
    while (n--) {
        insertAsLast(p->data);
        p = p->succ; // 将起自 p 的 n 项依次作为末节点插入
    }
}
```

###### 基于复制的构造
基于上述 copyNodes() 方法可以实现多种接口, 通过复制已有列表的区间或整体, 构造出新列表
```
// 复制列表中自位置 p 起的 n 项
template <typename T> List<T>::List(ListNodePosi(T) p, int n) {
    copyNodes(p, n);
}

// 整体复制列表 L
template <typename T> List<T>::List(List<T> const& L) {
    copyNodes(L.first, L._size);
}

// 复制 L 中自第 r 项起的 n 项
template <typename T> List<T>::List(List<T> const& L, int r， int n) {
    copyNodes(L[r], n);
}
```

##### 删除
###### 实现
```
template <typename T> T List<T>::remove(ListNodePosi(T) p) { // 删除合法节点 p, 放回其数值
    T e = p.date; //  备份待删除节点的数值 (假定 T 类型可以直接赋值)
    p->pred->succ = p->succ; // 后继
    p->succ->pred = p->pred; // 前驱
    delete p; // 释放节点
    _size--; // 更新规模
    return e; // 返回备份的数值
}
```

###### 复杂度
删除操作在常数时间内完成, 故时间复杂度为 $ O(1) $

##### 析构
###### 释放资源及清除节点
```
template <typename T> List<T>::~List() { // 列表析构器
    clear(); // 清空列表
    delete header; // 释放头
    delete trailer; // 释放尾
}

template <typename T> int List::clear() { // 清空列表
    int oldSize = _size;
    while (9 < _size) remove(header-succ); // 反复清空首节点, 直至列表变空
    return oldSize;
}
```
###### 复杂度
析构方法的时间消耗主要在于 clear() 方法, 时间复杂度为 $ O(n) $, 线性正比于列表原先的规模

##### 唯一化
###### 实现
```
template <typename T> int List<T>::deduplicate() { // 剔除无序列表中的重复节点
    if (_size < 2) return; // 平凡列表自然无重复
    int oldSize = _size; // 记录原规模
    ListNodePosi(T) p = header; // p 从首节点开始
    Rank r = 0;
    while (trailer != (p = p->succ)) { // 依次到末节点
        ListNodePosi(T) q = find(p->data, r, p); // 在 p 的 r 个前驱中查找雷同者
        q ? remove(q) : r++; // 若存在则删除, 否则秩加一
    }
    return oldSize - _size; // 列表规模变化量, 即被删除元素总数
}
```
###### 复杂度
与无序向量的去重算法一致, 该算法需做 $ O(n) $ 步迭代, 每一步迭代中的 find() 操作所需时间正比于查找区间宽度, 因此总体执行时间为 $ 1 + 2 + 3 + ... + n = n * (n - 1) / 2  = O(n^2) $

##### 遍历
```
template <typename T> void List<T>::traverse(void (*visit) (T&)) { // 借助函数指针机制遍历
    for(ListNodePosi(T) p = header->succ; p != trailer; p = p ->succ)
        visit(p->data);
}

template <typename T> template <typename VST> void List<T>::traverse(VST& visit) { // 借助函数对象机制遍历
    for(ListNodePosi(T) p = header->succ; p != trailer; p = p ->succ)
        visit(p->data);
}
```

#### 有序列表
若列表中所有节点的逻辑次序与其大小次序完全一致, 则称作有序列表 (sorted list)

##### 唯一化
```
template <typename T> int List<T>::uniquify() {  // 成批剔除重复元素, 效率更高
    if (_size < 2) return 0; // 平凡列表自然无重复
    int oldSize = _size; // 记录原规模
    ListNodePosi(T) p = first(); // p 为个区段起点
    ListNodePosi(T) q; // q 为 p 的后继
    while(trailer != (q = p->succ)) { // 反复考查紧邻节点对 (p, q)
        if (p->data != q->data)
            q = p; // 互异则转向下一区段
        else
            remove(q); // 雷同则删除后者
    }
    return oldSize - _size; // 列表规模变化量, 即被删除元素总数
}
```
整个过程的运行时间为 $ O(\_size) = O(n) $, 线性正比于列表的原先规模

##### 查找
###### 实现
```
// 在有序列表内节点 p 的 n 个前驱中, 找到不大于 e 的最后者
template <typename T> ListNodePosi(T) List<T>::search(T const& e, int n, ListNodePosi(T) p) const {
    while (0 <= n--) // 对于 p 的最近的 n 个前驱, 从右向左逐个比较
        if (()(p = p->pred)->data) >= e) // 直至命中, 数值越界, 或范围越界
            break;
    return p; // 返回查找终止的位置
} // 失败时, 返回区间左边界的前驱 (可能是 header), 调用者可通过 valid() 判断成功与否
```
###### 顺序查找
尽管有序列表中的节点已在逻辑上按次序单调排序, 但在动态存储策略中, 节点的物理地址与逻辑次序毫无关系, 故无法像有序向量那样自如地应用减治策略, 从而不得不沿用无序列表的顺序查找策略
###### 复杂度
线性正比于查找区间的宽度, 平均运行时间为 $ O(n) $

#### 排序器
