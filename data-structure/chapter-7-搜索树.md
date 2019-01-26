### 搜索树

#### 查找
##### 循关键码访问
所谓的查找或者搜索, 指从一组数据对象中找出符合特定条件者, 这是构建算法的一种基本而重要的操作; 其中的数据对象, 统一的表示和实现为词条 (entry) 的形式, 不同的词条之间依照各自的关键码 (key) 彼此区分; 这一新的访问方式, 与数据对象的物理位置与逻辑次序均无关, 查找的过程与结果仅仅取决于目标对象的关键码, 这种方式也称作循关键码访问 (call-by-key)

##### 词条
```
template <typename K, typename V> struct Entry { // 词条模板类
    K key; V value; // 关键码, 数据
    Entry(K k = K(), V v = V()): key(k), value(v) {}; // 基于克隆的构造器
    Entry(Entry<K,V> const& e): key(e.key), value(e.value) {}; // 基于克隆的构造函数
    bool operator< (Entry<K,V> const& e) {return key < e.key;} // 比较器: 小于
    bool operator> (Entry<K,V> const& e) {return key > e.key;} // 比较器: 大于
    bool operator== (Entry<K,V> const& e) {return key == e.key;} // 比较器: 等于
    bool operator!= (Entry<K,V> const& e) {return key != e.key;} // 比较器: 不等于
}
```
词条对象拥有成员变量 key 和 value, 前者作为特征, 是词条之间比对和比较的依据, 后者为实际的数据

##### 序与比较器
关键码之间能够相互比较, 这里有一个隐含条件: 所有词条构成一个全序关系, 可以相互比对和比较; 但是这一条件并不是总满足, 优势需要付出有限的代价来维护一个偏序关系

#### 二叉搜索树
##### 顺序性
若二叉树中各节点所对应的词条之间支持大小比较, 则在不致歧义的情况下, 可以不必严格区分树中的节点, 节点所对应的词条以及词条内部所对应的关键码  
在二叉搜索树 (binary search tree) 中, 处处都满足顺序性: ** 任一节点 r 的左 (右) 子树中, 所有节点 (若存在) 均不大于 (不小于) r **
```
        --------- r ---------
        |                   |
       r_L                 r_R
  L-subtree(<=r)       L-subtree(r<=)
```
##### 中序遍历序列
```
                               |
                     --------- 16 ---------
                     |                    |
            --- 10 ---             ------ 25 ------
           |         |             |              |
       --- 5 ----    11 ---    --- 19 ---    --- 28 ---
       |        |         |    |        |    |        |
       2 ---    8     --- 15   17       22   27   --- 37
           |          |                           |
           4          13                          33
二叉搜索树的中序遍历: 2, 4, 5, 8, 10, 11, 13, 15, 16, 17, 19, 22, 25, 17, 18, 33, 37
```
任何一颗二叉树是二叉搜索树, 当且仅当其中序遍历序列单调非降

##### BST 模板类
二叉搜索树是二叉树的特例, 故可以基于 BinTree 类派生出 BST 模板类
```
#include "../BinTree/BinTree.h" // 引入 BinTree

template <typename T> class BST : public BinTree<T> { // 由 BinTree 派生 BST 模板类
    protected:
        BinNodePosi(T) _hot; // "命中" 节点的父亲
        BinNodePosi(T) connect34 ( // 按照 3 + 4 结构, 联接 3 个节点 4 个子树
            BinNodePosi(T), BinNodePosi(T), BinNodePosi(T),
            BinNodePosi(T), BinNodePosi(T), BinNodePosi(T), BinNodePosi(T))
        BinNodePosi(T) rotateAt(BinNodePosi(T) x); // 对 x 及其父亲, 祖父做统一旋转调整
    public: // 基本接口: 以 virtual 修饰, 强制要求所有派生类根据各各自的规则对其重写
        virtual BinNodePosi(T) & search(const T& e); // 查找
        virtual BinNodePosi(T) insert(const T& e); // 插入
        virtual bool remove(const T& e); // 删除
}
```
这些接口的语义均涉及词条的大小和相等关系, 故这里也假定基本元素类型 T 或者直接支持比较和判等操作, 或者已经重载过对应的操作符
##### 查找算法及其实现
###### 算法
二叉搜索树查找算法, 采用减而治之的思路和策略, 其执行过程可描述为: ** 从树根出发, 逐步地缩小范围, 直到发现目标 (成功) 或者缩小至空树 (失败) **  
一般的, 在查找过程中, 一旦发现当前节点为 NULL, 即说明查找范围已经缩小至空, 查找失败; 否则视关键码比较结果, 向左 (更小) 或向右 (更大) 深入, 或者报告成功 (相等)  
对照中序遍历可见, 整个过程与有序向量的二分查找过程等效, 可视为后者的推广
###### searchIn() 算法和 search() 接口
一般的, 在子树 v 中查找关键码 e 的过程, 可实现为如下的 searchIn()
```
// 在以 v 为根的 (AVL, SPLAY, rbTree 等) BST 子树中查找关键码 e
template <typename T> static BinNodePosi(T) & searchIn(BinNodePosi(T) & v, const T& e, BinNodePosi(T) & hot) {
    if (!v || e == v->data) return v; // 递归基, 在节点 v (或假想的通配节点) 处命中
    hot = v; // 一般情况: 先记下当前节点
    return searchIn(((e < v->data) ? v->lc : v->rc), e, hot); // 深入一层, 递归查找
} // 返回时, 返回值指向命中节点, hot 指向器父节点 (退化时为初始值 NULL)
```
通过调用 searchIn() 算法, 可实现二叉搜索树的标准接口 search()
```
// 在 BST 中查找关键码 e
template <typename T> BinNodePosi(T) & BST<T>::search(const T& e) {
    return searchIn(_root, e, _hot = NULL);
} // 返回节点位置, 以便后续插入和删除操作
```
