### 二叉树
线性结构中的元素之间都存在着一个自然的线性约束; 树则不然, 其中的元素并不存在天然的直接后继或直接前驱关系, 但只要附加某种约束, 也可以在树中元素之间确定某种线性次序

#### 二叉树及其表示
##### 树
###### 有根树
从图论的角度来看, 树等价于连通无环图; 与一般图相同, 树也由一组顶点 (vertex) 以及联接与其间的若干条边 (edge) 组成; 往往会在此基础上指定一个特定顶点, 并称之为 (root); 在指定根节点后也称之为有根树 (rooted tree)
###### 深度与层次
由于树的连通性, 每一节点与根之间都有一条路径相联; 而根据树的无环性, 由根通往每个节点的路径必然唯一; 沿每个节点 v 到根 r 的唯一通路所经过的边的数目, 称作 v 的深度 (depth), 记作 depth(v); 约定根节点的深度 depth(r) = 0, 故属于第 0 层
###### 祖先, 后代, 子树
任一节点 v 在通往树根沿途所经过的每个节点都是其祖先 (ancestor), v 是它们的后代 (descendant); 特别的, v 的祖先/后代包括其本身, 而 v 本身以外的祖先/后代称作真祖先 (proper ancestor) / 真后代 (proper descendant)  
节点 v 在每一层次上, v 的祖先至多一个; 若节点 u 是 v 的祖先且恰好比 v 高出一层, 则称 u 是 v 的父亲 (parent), v 是 u 的孩子 (child)  
v 的孩子总数, 称作其度数或度 (degree), 记作 deg(v); 无孩子的节点称作叶节点 (leaf), 包括根在内的其余节点皆为内部节点 (internal node)  
v 所有的后代及其之间的联边称作子树 (subtree), 记作 subtree(v)
###### 高度
树 T 中所有节点深度的最大值称作该树的高度 (height), 记作 height(T); 树的高度总是由其某一叶节点的深度确定的, 特别的, 仅含单个节点的树高度为 0, 空树的高度为 -1; 推而广之, 任一节点 v 所对应的子树 subtree(v) 的高度, 亦称作该节点的高度, 记作 height(v); 特别的, 全树的高度亦即其根节点 r 的高度, height(T) = height(r)

##### 二叉树
二叉树 (binary tree) 中的每个节点的度数均不超过 2, 因此在二叉树中的孩子节点都可以用左右区分, 亦称作有序二叉树 (ordered binary tree); 特别的, 不含一度节点的二叉树称作真二叉树 (proper binary tree)

##### 多叉树
一般的, 树中各节点的孩子数目并不确定, 每个节点的孩子均不超过 k 个的有根树, 称作 k 叉树 (k-ary tree)
###### 父节点
在多叉树中, 根节点以外的任一节点有且仅有一个父节点
```
        |
  ------R------
  |     |     |
--A--   B     C
|   |         |
D   E       --F--
            | | |
            G H K
```
将多叉树的各节点组织为向量或者列表, 其中每个元素除了保存节点本身的信息 (data) 外, 还需要保存父节点 (parent) 的秩或者位置, 可为树根指定一个虚构的父节点 -1 或者 NULL, 以便做统一的判断; 如此在所有向量或列表所占的空间为 O(n), 线性正比于节点总数 n; 时间方面, 确定任一节点的父节点仅需常数时间, 但孩子节点的查找却需要花费 O(n) 的时间
```
   | data   parent
---|--------------
0  |  R      -1
1  |  A       0
2  |  B       0
3  |  C       0
4  |  D       1
5  |  E       1
6  |  F       3
7  |  G       6
8  |  H       6
9  |  K       6
```
###### 孩子节点
若注重孩子节点的快速定位, 则令各节点将其所有的盖子组织为一个向量或列表; 这样对于拥有 r 个孩子的节点, 可以在 O(r + 1) 时间内列举出所有的孩子
```
   | data   children
---|----------------
0  |  A       * ---> [3 | *] ---> [5 | ^]
1  |  B       ^
2  |  C       * ---> [6 | ^]
3  |  D       ^
4  |  R       * ---> [0 | *] ---> [1 | *] ---> [2 | ^]
5  |  E       ^
6  |  F       * ---> [7 | *] ---> [8 | *] ---> [9 | ^]
7  |  G       ^
8  |  H       ^
9  |  K       ^
```
###### 父节点 + 孩子节点
结合父节点表示法和孩子节点表示法, 可令各节点既记录父节点, 同时也维护一个序列以保存所有孩子
```
| data   parent   children
---|----------------------
0  |  A    4         * ---> [3 | *] ---> [5 | ^]
1  |  B    4         ^
2  |  C    4         * ---> [6 | ^]
3  |  D    0         ^
4  |  R    -1        * ---> [0 | *] ---> [1 | *] ---> [2 | ^]
5  |  E    0         ^
6  |  F    2         * ---> [7 | *] ---> [8 | *] ---> [9 | ^]
7  |  G    6         ^
8  |  H    6         ^
9  |  K    6         ^
```
尽管可以如此高效的兼顾父节点和孩子节点的定位, 但在节点插入与删除操作频繁的场合, 为动态维护和更新拓扑结构, 不得不反复遍历和调整一些节点所对应的孩子序列; 然而向量和列表等线性结构都需要耗费大量时间, 势必影响到整体的效率
###### 有序多叉树 = 二叉树
为了保证作为多叉树特例的二叉树有足够的能力表示任何一颗多叉树, 只需要给树增加一项约束条件: 同一节点的所有孩子之间必须具有某一线性次序; 凡符合这一条件的多叉树也称作有序树 (ordered tree)
###### 长子 + 兄弟
有序多叉树中任一非叶节点都有唯一的 "长子", 而且从该 "长子" 出发, 可按照预先约定或指定的次序遍历所有孩子节点; 为每个节点设置两个指针, 分别指向其 "长子" 和下一 "兄弟"
```
|
R
|
A ---> B ---> C
|             |
D ---> E      F
              |
              G ---> H ---> K
```
若将这两个指针分别与二叉树节点的左右孩子指针统一对应起来, 则进一步将具有有序多叉树转换为常规的二叉树
```
                        |
       -----------------R
       |
   --- A ---
   |       |
   D ---   B -------
       |           |
       E       --- C
               |
           --- F
           |
           G ---
               |
               H ---
                   |
                   K
```

#### 编码树
通信中的一个基本问题是: 如何在尽可能低的情况下, 以尽可能高的速度, 尽可能忠实的实现信息在空间和时间上的复制与转移; 信息的传递在信道上大多以二进制比特的形式表示和存在, 而每一个具体的编码方案都对应于一颗二叉编码树

##### 二进制编码
在加载到信道撒谎功能之前, 信息被转换为二进制形式的过程称为编码 (encoding); 反之, 经信道抵达目标后再由二进制编码恢复原始信息的过程称为解码 (decoding)

##### 生成编码表
原始信息的基本组成单位称作字符, 它们都来自于某一特定的有限集合 $ \Sigma $, 也称作字符集 (alphabet); 而以二进制形式承载的信息, 都可表示为来自编码表 $ \Gamma $ = {0, 1}* 的某一特定二进制串; 从这个角度讲, 每一编码表都是从字符集 $ \Sigma $ 到编码表  $ \Gamma $ 的一个单射, 编码就是对信息文本中各字符逐个实施这一映射的过程, 而解码则是逆向映射的过程; 编码表一旦确定, 信息的发送方与接收方之间就建立起了一个约定与默契, 从而使得独立的编码与解码成为可能

| 字符 | A | E | G | M | S |
| :--- | :--- | :--- | :--- | :--- |:--- |
| 编码 | 00 | 01 | 10 | 110 | 111 |

###### 二进制编码
所谓编码就是对任意给定的文本, 通过查阅编码表逐一将其中的字符转译为二进制编码; 例如待编码文本为 "MESSAGE"

| 二进制编码 | 当前匹配字符 | 解出原文 |
| :--- | :--- | :--- |
| 11001111111001001 | M | M |
| 01111111001001 | E | ME |
| 111111001001 | S | MES |
| 111001001 | S | MESS |
| 001001 | A | MESSA |
| 1001| G | MESSAG |
| 01 | E | MESSAGE |

###### 二进制解码
接收方依次扫描各比特位, 并经匹配逐一转译出各字符, 从而最终恢复出原始的文本

###### 解码歧义
在编码方案确定后, 尽管编码结果必然确定, 但解码过程和结果却不见得唯一; 例如若将字符 "M" 的编码由 "110" 改为 "11", 则原始文本 "MESSAGE" 经编码得到二进制码的解法则有至少两种

| 二进制编码 | 当前匹配字符 | 解出原文 | 二进制编码 | 当前匹配字符 | 解出原文
| :--- | :--- | :--- | :--- | :--- |:--- |
| 11001111111001001 | M | M | 11001111111001001 | M | M |
| 01111111001001 | E | ME | 01111111001001 | E | ME |
| 111111001001 | S | MES | 111111001001 | S | MEM |
| 111001001 | S | MESS | 1111001001 | S | MEMM |
| 001001 | A | MESSA | 11001001 | A | MEMMM |
| 1001| G | MESSAG | 001001| G | MEMMMA |
| 01 | E | MESSAGE | 1001 | E | MEMMMAG |
| - | - | - | 01 | E | MEMMMAGE |

###### 前缀无歧义编码
解码过程中出现歧义甚至错误, 根源在于编码表制订不当; 为了消除匹配歧义, 任何两个原始字符所对应的二进制编码串, 相互都不得是前缀; 反过来, 只要各字符的编码串互不为前缀, 即便出现无法解码的错误, 但绝不会导致歧义, 这类编码方法即所谓的 "前缀无歧义编码 (prefix-free code)", 简称 PFC 编码

##### 二叉编码树
###### 根通路与节点编码
任一编码方案都可描述为一颗二叉树: 从根节点出发, 每次向左 (右) 都对应一个 0 (1) 的比特位; 从根节点到每个节点的唯一通路, 可以为各节点 v 赋予一个互异的二进制串, 称作根通路串 (root path string), 记作 rps(v), |rps(v)|  = depth(v) 就是 v 的深度  
若将 $ \Sigma $ 中的字符分别映射至二叉树的节点, 则字符 x 的二进制编码串即可取做 rps(v(x)); 以下在不致引起混淆的前提下, 不再区分字符 x 和与之对应的节点 v(x), 于是 rps(v(x) 可简记为 rps(x), depth(v(x)) 简记为 depth(x)

###### PFC 编码树
只要所有字符都对应与叶节点, 歧义现象自然消除, 这也是实现 PFC 编码的简明策略

###### 基于 PFC 编码树的解码
依据 PFC 编码树也可以便捷的完成编码串的解码; 以 "11001111111001001" 为例, 从前向后扫描该串, 同时在树中相应移动, 起始时从树根出发, 视各比特位的取值相应的向左或者向右深入下一层, 直达叶节点, 匹配字符后重新在回到树根重复这一过程; 这一解码过程可以在二进制编码串的接受过程中实时进行, 而不必等到所有比特位都到达后才开始

###### PFC 编码树的构造
PFC 编码方案可由 PFC 编码树来描述, 但如何构造编码树?

#### 二叉树的实现
作为图的特殊形式, 二叉树的基本组成单元是节点和边; 作为数据结构, 其基本的组成实体是二叉树节点 (binary tree node), 而边则对应于节点之间的相互作用

##### 二叉树节点
###### BinNode 模板类
```
#define BinNodePosi(T) BinNode<T>* //节点位置
#define stature(p) ((p) ? (p)->height : -1) //节点高度（与“空树高度为-1”的约定相统一）
typedef enum { RB_RED, RB_BLACK} RBColor; //节点颜色

template <typename T> struct BinNode { //二叉树节点模板类
// 成员（为简化描述起见统一开放，读者可根据需要进一步封装）
   T data; //数值
   BinNodePosi(T) parent; BinNodePosi(T) lc; BinNodePosi(T) rc; //父节点及左、右孩子
   int height; //高度（通用）
   int npl; //Null Path Length（左式堆，也可直接用height代替）
   RBColor color; //颜色（红黑树）
// 构造函数
   BinNode() :
      parent ( NULL ), lc ( NULL ), rc ( NULL ), height ( 0 ), npl ( 1 ), color ( RB_RED ) { }
   BinNode ( T e, BinNodePosi(T) p = NULL, BinNodePosi(T) lc = NULL, BinNodePosi(T) rc = NULL,
             int h = 0, int l = 1, RBColor c = RB_RED ) :
      data ( e ), parent ( p ), lc ( lc ), rc ( rc ), height ( h ), npl ( l ), color ( c ) { }
// 操作接口
   int size(); //统计当前节点后代总数，亦即以其为根的子树的规模
   BinNodePosi(T) insertAsLC ( T const& ); //作为当前节点的左孩子插入新节点
   BinNodePosi(T) insertAsRC ( T const& ); //作为当前节点的右孩子插入新节点
   BinNodePosi(T) succ(); //取当前节点的直接后继
   template <typename VST> void travLevel ( VST& ); //子树层次遍历
   template <typename VST> void travPre ( VST& ); //子树先序遍历
   template <typename VST> void travIn ( VST& ); //子树中序遍历
   template <typename VST> void travPost ( VST& ); //子树后序遍历
// 比较器、判等器（各列其一，其余自行补充）
   bool operator< ( BinNode const& bn ) { return data < bn.data; } //小于
   bool operator== ( BinNode const& bn ) { return data == bn.data; } //等于
};
```
这里通过宏 BinNodePosi 来指代节点位置, 使用 stature 返回节点的高度值

###### 成员变量
BinNode 节点由多个成员变量组成, 它们分别记录了当前节点的父亲和孩子的位置, 节点内存放的数据以及节点的高度等指标, 这些是二叉树的相关算法赖以实现的基础
```
     parent                         
       |                            [lc] [parent] [rc]
 --- [data] ---                     [      data      ]
 |            |                     [height|npl|color]
lChild       rChild
```
###### 快捷方式
二叉树节点状态和性质的常用功能
```
// BinNode 状态与性质的判断
#define IsRoot(x) (!(x).parent)
#define IsLChild(x) (!IsRoot(x) && (&(x) == (x).parent->lc))
#define IsRChild(x) (!IsRoot(x) && (&(x) == (x).parent->rc))
#define HasParent(x) (!IsRoot(x))
#define HasLChild(x) ((x).lc)
#define HasRChild(x) ((x).rc)
#define HasChild(x) (HasLChild(x) || HasRChild(x)) // 至少有一个孩子
#define HasBothChild(x) (HasLChild(x) && HasRChild(x)) // 同时有两个孩子
#define IsLeaf(x) (!HasChild)

// 与 BinNode 具有特定关系的节点及指针
#define sibling(p) (IsLChild(*(p)) ? (p)->parent->rc : (p).parent->lc) // 兄弟
#define uncle(x) (IsLChild(*((x)->parent)) ? (x)->parent->parent->rc) : (x)->parent->lc) // 叔叔
#define FromParentTo(x) (IsRoot(x) ? _root : (IsLChild(x) ? (x).parent->lc : (x).parent->rc)) // 来自父亲的引用
```

##### 二叉树节点操作接口
###### 插入孩子节点
```
template <typename T> BinNodePosi(T) BinNode<T>::insertAsLC(T const& e) {
    return lc = new BinNode(e, this); // 将 e 作为当前节点的左孩子插入二叉树
}

template <typename T> BinNodePosi(T) BinNode<T>::insertAsRC(T const& e) {
    return rc = new BinNode(r, this); // 将 e 作为当前节点的右孩子插入二叉树
}
```
###### 定位直接后继
TODO
###### 遍历
```
template <typename T> template <typename T> void BinNode<T>::travIn(VST& visit) { // 二叉树中序遍历算法的统一入口
    swith(rand() % 5) { // 随机选择
        case 1: travIn_I1(this, visit); break; // 迭代版 1
        case 2: travIn_I1(this, visit); break; // 迭代版 2
        case 3: travIn_I1(this, visit); break; // 迭代版 3
        case 4: travIn_I1(this, visit); break; // 迭代版 4
        case 5: travIn_I1(this, visit); break; // 迭代版 5
        default: travIn_R(this, visit); break; // 递归版
    }
}
```

##### 二叉树
```
#include "BinNode.h" //引入二叉树节点类
template <typename T> class BinTree { //二叉树模板类
protected:
   int _size; BinNodePosi(T) _root; //规模、根节点
   virtual int updateHeight ( BinNodePosi(T) x ); //更新节点x的高度
   void updateHeightAbove ( BinNodePosi(T) x ); //更新节点x及其祖先的高度
public:
   BinTree() : _size ( 0 ), _root ( NULL ) { } //构造函数
   ~BinTree() { if ( 0 < _size ) remove ( _root ); } //析构函数
   int size() const { return _size; } //规模
   bool empty() const { return !_root; } //判空
   BinNodePosi(T) root() const { return _root; } //树根
   BinNodePosi(T) insertAsRoot ( T const& e ); //插入根节点
   BinNodePosi(T) insertAsLC ( BinNodePosi(T) x, T const& e ); //e作为x的左孩子（原无）插入
   BinNodePosi(T) insertAsRC ( BinNodePosi(T) x, T const& e ); //e作为x的右孩子（原无）插入
   BinNodePosi(T) attachAsLC ( BinNodePosi(T) x, BinTree<T>* &T ); //T作为x左子树接入
   BinNodePosi(T) attachAsRC ( BinNodePosi(T) x, BinTree<T>* &T ); //T作为x右子树接入
   int remove ( BinNodePosi(T) x ); //删除以位置x处节点为根的子树，返回该子树原先的规模
   BinTree<T>* secede ( BinNodePosi(T) x ); //将子树x从当前树中摘除，并将其转换为一棵独立子树
   template <typename VST> //操作器
   void travLevel ( VST& visit ) { if ( _root ) _root->travLevel ( visit ); } //层次遍历
   template <typename VST> //操作器
   void travPre ( VST& visit ) { if ( _root ) _root->travPre ( visit ); } //先序遍历
   template <typename VST> //操作器
   void travIn ( VST& visit ) { if ( _root ) _root->travIn ( visit ); } //中序遍历
   template <typename VST> //操作器
   void travPost ( VST& visit ) { if ( _root ) _root->travPost ( visit ); } //后序遍历
   bool operator< ( BinTree<T> const& t ) //比较器（其余自行补充）
   { return _root && t._root && lt ( _root, t._root ); }
   bool operator== ( BinTree<T> const& t ) //判等器
   { return _root && t._root && ( _root == t._root ); }
}; //BinTree
```
###### 高度更新
二叉树任一节点的高度, 都等于其孩子节点的最大高度加一; 于是每当某一节点的孩子或后代有所增减, 其高度都有必要及时更新; 然而节点自身很难发现后代的变化, 因此不妨反过来采用另一处理策略: 一旦有节点加入或者离开二叉树, 则更新其所有祖先的高度; 在每一节点 v 处, 只需读出其左右孩子的高度并取二者之间的最大者, 再计入当前节点本身, 就得到了 v 的新高度; 通常还需要从 v 出发沿 parent 指针逆行向上, 依次更新各代祖先的高度记录
```
template <typename T> int BinTree<T>::updateHeight(BinNodePosi(T) x) { // 更新节点新高度
    return x->height = 1 + max(stature(x->lc), stature(x->rc)); // 具体规则因树而异
}

template <typename T> void BinTree<T>::updateHeightAbove(BinNodePosi(T) x) { // 更新高度
    while (x) {
        updateHeight(x);
        x = x->parent;
    }
}
```
更新灭一节点本身的高度, 只需执行两次 getHeight() 操作, 两次加法以及两次取最大操作; 不过常数时间, 故 updateHeight() 算法总体运行时间为 O(depth(v) + 1), 其中 depth(v) 为节点 v 的深度; 这一步操作可以进一步优化
###### 节点插入
二叉树节点可以通过三种方式插入二叉树中
```
template <typename T> BinNodePosi(T) BinNode<T>::insertAsRoot(T const& e) {
    _size = 1;
    return _root = new BinNode<T>(e); // 将 e 当作根节点插入空的二叉树
}

template <typename T> BinNodePosi(T) BinNode<T>::insertAsLC(BinNodePosi(T) x, T const& e) {
    _size++;
    x->insertAsLC(e); // e 插入为 x 的左孩子
    updateHeightAbove(x);
    return x->lc;
}

template <typename T> BinNodePosi(T) BinNode<T>::insertAsRC(BinNodePosi(T) x, T const& e) {
    _size++;
    x->insertAsRC(e); // e 插入为 x 的右孩子
    updateHeightAbove(x);
    return x->lc;
}
```
###### 子树接入
任一二叉树均可作为另一二叉树的指定节点的左子树或右子树
```
// 二叉树子树接入算法: 将 S 当作节点的左子树接入, S 本身置空
template <typename T> BinNodePosi(T) BinTree<T>::attachAsLC(BinNodePosi(T) x, BinTree<T>* &S) { // x->lc == NULL
    if (x->lc = S->_root) x->lc->parent = x; // 接入
    _size += S._size; // 更新数规模
    updateHeightAbove(x); // 更新祖先高度
    S->_root = NULL; // 释放原树
    S->_size = 0;
    release(S);
    S = NULL;
    return x; // 返回接入位置
}

// 二叉树子树接入算法: 将 S 当作节点的右子树接入, S 本身置空
template <typename T> BinNodePosi(T) BinTree<T>::attachAsRC(BinNodePosi(T) x, BinTree<T>* &S) { // x->rc == NULL
    if (x->lc = S->_root) x->rc->parent = x; // 接入
    _size += S._size; // 更新数规模
    updateHeightAbove(x); // 更新祖先高度
    S->_root = NULL; // 释放原树
    S->_size = 0;
    release(S);
    S = NULL;
    return x; // 返回接入位置
}
```
###### 子树删除
子树删除过程与子树接入过程相反, 不同的是需要将被摘除子树中的节点, 逐一释放并归还系统
```
// 删除二叉树中位置 x 处的节点及其后代, 返回被删除节点的个数
template <typename T> int BinTree<T>::remove(BinNodePosi(T) x) {
    FromParentTo(*x) = NULL; // 切断来自父节点的指针
    updateHeightAbove(x->parent); // 更新祖先高度
    int n = removeAt(x); // 删除子树
    _size -= n; // 更新高度
    return n; // 返回删除节点的个数
}

// 删除二叉树中位置 x 处的节点及其后代, 返回被删除节点的数值
template <typename T> static int removeAt(BinNodePosi(T) x) {
    if (!x) return 0; // 递归基, 空置
    int n = 1 + removeAt(x->lc) + removeAt(x->rc); // 释放左右子树
    release(x->data);
    release(x);
    return n; // 释放被摘除节点, 并返回删除节点的总数
}
```
###### 子树分离
子树分离过程与子树删除过程基本一致, 不同的是需要对分离出来的子树重新封装并返回给上层调用者
```
// 二叉树子树分离算法: 将子树 x 从当前树中摘除, 将其封装为一颗独立子树返回
template <typename T> BinTree* BinTree<T>::secede(BinNodePosi(T) x) {
    FromParentTo(*x) = NULL; // 切断来自父节点的指针
    updateHeightAbove(x->parent); // 更新原树中所有祖先的高度
    BinTree<T>* S = new BinTree<T>;
    S->_root = x; // 新树以 x 为根
    x-parent = NULL;
    S->_size = x->size();
    _size -= S->_size; // 更新规模
    retunr S; // 返回分离出的子树
}
```
###### 复杂度
以上算法的中除了更新祖先高度和释放节点等操作, 只需常数时间

#### 遍历
对于二叉树的访问多可抽象为如下形式: 按照某种约定的次序, 对节点各访问一次且仅一次; 与向量和列表等线性结构一样, 二叉树的这类访问也称作遍历 (traversal); 这一过程也等效于将半线性的树形结构, 转换为线性结构

##### 递归式遍历
二叉树本身不具有天然的全局次序, 故为实现遍历, 首先需要在各节点与其孩子之间约定某种局部次序, 从而间接地定义出全局次序; 按照惯例左孩子优先于右孩子, 故若将节点及其孩子分别记作 V,L,R, 则局部访问次序有 VLR, LVR, LRV 三种选; 根据节点 V 在其中的访问次序, 这三种策略也相应地分别称作先序遍历, 中序遍历, 后序遍历

###### 先序遍历
```
// 二叉树先序遍历算法 (递归版)
template <typename T, typename VST> void travPre_R(BinNodePosi(T) x, VST& visit) {
    if (!x) return;
    visit(x->data);
    travPre_R(x->lc, visit);
    travPre_R(x->rc, visit);
}
```
为遍历 (子) 树 x, 首先核对 x 是否为空, 若为空则直接退出, 这也是递归基; 反之若 x 非空, 则按照先序遍历关于局部次序的定义, 优先访问其根节点 x, 然后左子树和右子树, 递归地进行遍历
```
// preoredr traversal
              |
          --- i ------------------
          |                      |
      --- d ------           --- l ---
      |          |           |       |
  --- c      --- h       --- k   --- n ---
  |          |           |       |       |
  a ---  --- f ---       j       m   --- p
      |  |       |                   |
      b  e       g                   o

先序遍历顺序: -> a -> d -> c -> a -> b -> h -> f -> e -> g -> l -> k -> j -> n -> m -> p -> o ->
```
###### 后序遍历
```
// 二叉树后序遍历算法 (递归版)
template <typename T, typename VST> void travPre_R(BinNodePosi(T) x, VST& visit) {
    if (!x) return;
    travPre_R(x->lc, visit);
    travPre_R(x->rc, visit);
    visit(x->data);
}
```
若 x 非空, 则按照后序遍历关于局部次序的定义, 优先访问其左子树, 然后右子树, 最后才访问 x 的递归进行遍历
```
// postoredr traversal
              |
          --- i ------------------
          |                      |
      --- d ------           --- l ---
      |          |           |       |
  --- c      --- h       --- k   --- n ---
  |          |           |       |       |
  a ---  --- f ---       j       m   --- p
      |  |       |                   |
      b  e       g                   o

后序遍历顺序: -> b -> a -> c -> e -> g -> f -> h -> d -> j -> k -> m -> o -> p -> n -> l -> i ->
```
###### 中序遍历
```
// 二叉树中序遍历算法 (递归版)
template <typename T, typename VST> void travPre_R(BinNodePosi(T) x, VST& visit) {
    if (!x) return;
    travPre_R(x->lc, visit);
    visit(x->data);
    travPre_R(x->rc, visit);
}
```
若 x 非空, 则按照中序遍历关于局部次序的定义, 优先访问其左子树, 然后访问 x, 最后访问右子树的递归进行遍历
```
// inoredr traversal
              |
          --- i ------------------
          |                      |
      --- d ------           --- l ---
      |          |           |       |
  --- c      --- h       --- k   --- n ---
  |          |           |       |       |
  a ---  --- f ---       j       m   --- p
      |  |       |                   |
      b  e       g                   o

后序遍历顺序: -> a -> b -> c -> d -> e -> f -> g -> h -> i -> j -> k -> l -> m -> n -> o -> p ->
```
各节点在中序遍历序列中的局部次序, 与按照有序树定义所确定的全局左右次序完全吻合

##### 迭代版先序遍历
无论是递归式还是迭代式遍历算法, 都只需渐进的线性时间, 但递归版遍历算法时间, 空间复杂度的常系数相对迭代版更大
###### 版本 1
TODO
###### 版本 2
在二叉树 T 中, 从根节点出发沿着左分支一直下行的那条通路称作为最左侧通路 (leftmost path); 若将沿途的节点分别记作 $ L_k, k = 0, 1,, 2,..., d $, 则最左侧通路终止于没有左孩子末端节点 $ L_d $; 若这些节点的右孩子和右子树分别记作 $ R_k $ 和 $ T_k, k = 0, 1,, 2,..., d $; 则该二叉树的先序遍历序列可表示为
$$
\begin{align}
preoredr(T) & = visit(L_0), visit(L_1), ..., visit(L_0); \\
            & preoredr(T_d), ..., preoredr(T_1), preoredr(T_0);
\end{align}
$$
也就是说, 先序遍历序列可分解为两段: 沿最左侧通路自顶向下访问各节点, 以及自底向上遍历的对应右子树
```
// 从当前节点出发, 沿左分支不断深入, 直至没有左分支的节点; 沿途节点遇到后便访问
template <typename T, typename VST> static void visitAlongLeftBranch(BinNodePosi(T) x, VST& visit, Stack<BinNodePosi(T)>& S) {
    while (x) {
        visit(x->data); // 访问当前节点
        S.push(x-rc); // 右孩子入栈 (可优化, 避免空的右孩子入栈)
        x = x->lc; // 沿左分支深入一层
    }
}

// 二叉树先序遍历: 迭代版 2
template <typename T, typename VST> void travPre_I2(BinNodePosi(T) x, VST& visit) {
    Stack<BinNodePosi(T)> S; // 辅助栈
    while (true) {
        visitAlongLeftBranch(x, visit, S);
        if (S.empty()) break; // 直到栈空
        x = S.pop();
    }
}
```

##### 迭代版中序遍历
在中序遍历的递归版本中, 尽管右子树的递归遍历是尾递归, 但左子树绝对不是; 实现迭代版的难点正在于此
###### 版本 1
类似与先序遍历版本 2 的思路, T 的中序遍历序列可表示如下
\begin{align}
inoredr(T) & = visit(L_d), visit(T_d); \\
           & visit(L_{d-1}), visit(T_{d-1}); \\
           & ..., ...; \\
           & visit(L_1), visit(T_1); \\
           & visit(L_0), visit(T_0);
\end{align}
$$
也就是说, 沿最左侧通路自底向上, 以沿途各节点为界, 中序遍历序列可分解为 d + 1 段; 各段彼此独立, 且均包括访问来自最左侧通路的某一节点 $ L_k $, 以及遍历其对应的右子树 $ T_k $
```
// 从当前节点出发, 沿左分支不断深入, 直至没有左分支的节点
template <typename T> static void goAlongLeftBranch(BinNodePosi(T) x, Stack<BinNodePosi(T)>& S) {
    while (x) {
        S.push(x); // 当前节点入栈
        x = x->lc; // 向左分支深入, 直至无左孩子
    }
}

// 二叉树中序遍历: 迭代版 1
template <typename T, typename VST> void travIn_I1(BinNodePosi(T) x, VST& visit) {
    Stack<BinNodePosi(T)> S; // 辅助栈
    while (true) {
        goAlongLeftBranch(x, S); // 从当前节点出发, 逐批入栈
        if (S.empty()) break; // 治秩所有节点处理完毕
        x = S.pop();
        visit(x->data); // 弹出栈顶节点并访问
        x = x->rc; // 转向右子树
    }
}
```

###### 直接后继及其定位
中序遍历的实质功能也可理解为, 为所有节点赋予一个次序, 从而将半线性的二叉树转化为线性结构; 一旦定义了遍历策略, 即可与向量和列表一样, 在二叉树的节点之间定义前驱与后继关系; 其中没有前驱 (后继) 的节点成为首 (末) 节点; succ() 为定位任意节点在中序遍历序列中的直接后继
```
template <typename T> BinNodePosi(T) BinNode<T>::succ() { // 定位节点 v 的直接后继
    BinNodePosi(T) s = this; // 记录后继的临时变量
    if (rc) { // 若有右孩子, 则直接后继必在右孩子中
        s = rc;
        while (HasLChild(*s)) s = s->lc; // 右子树中最靠左的节点
    } else { // 否则, 直接后继应该是 "将当前节点包含于其左子树中的最低祖先"
        while (IsRChild(*s)) s = s->parent; // 逆向的沿右向分支, 不断朝左上方移动
        s = s->parent; // 最后再朝右上方移动一步, 即抵达直接后继 (如果存在)
    }
    return s;
}
```
这里共分为两大类情况:
- 若当前节点有右孩子, 则其直接后继必然存在, 且属于其右子树, 此时只需转入右子树, 再沿该子树的最左侧通路朝左下方深入, 直接抵达子树中最靠左的节点
- 若当前节点没有右子树, 则若其直接后继存在, 必然为该节点上的某一祖先, 且是将当前节点纳入其左子树的最低祖先, 于是沿着右侧通路朝左上方上升, 当不能继续前进时, 再朝右上方移动一步即可; 若有祖先为 NULL, 这意味着此前沿着右侧通路向上的回溯到达了树根; 也就是说, 当前节点是全树右侧通路的终点, 也是中序遍历的终点
###### 版本 2
```
// 二叉树中序遍历: 迭代版 2
template <typename T, typename VST> void travIn_I2(BinNodePosi(T) x, VST& visit) {
    Stack<BinNodePosi(T)> S; // 辅助栈
    while (true) {
        if (x) {
            S.push(x); // 根节点入栈
            x = x->lc; // 深入遍历左子树
        } else if (!S.empty()) {
            x = S.pop(); // 尚未访问的最低祖先节点出栈
            visit(x->data); // 访问该祖先节点
            x = x->rc; // 遍历祖先的右子树
        } else {
            break; // 遍历完成
        }
    }
}
```
版本 2 只是版本 1 的等价形式, 但确实到版本 3 的过渡
###### 版本 3
以上迭代式都需使用辅助栈, 所需辅助空间规模线性正比于二叉树的高度, 最坏情况下与节点总数相当; 版本 3 则无需使用任何结构, 总体仅需 O(1) 的辅助空间, 属于就地算法, 但因为需要反复调用 succ(), 时间效率有所倒退
```
// 二叉树中序遍历: 迭代版 3
template <typename T, typename VST> void travIn_I2(BinNodePosi(T) x, VST& visit) {
    bool backtrack = false; // 前一步是否刚从右子树回溯 (为了省去栈)
    while (true) {
        if (!backtrack && HasLChild(*x)) { // 若有左子树且不是刚刚回溯
            x = x->lc; // 深入左子树
        } else { // 无左子树或刚回溯
            visit(x-data); // 访问该节点
            if (HasRChild(*x)) { // 若右子树非空
                x = x->rc; // 深入右子树
                backtrack = false; // 并关闭回溯
            } else { // 若右子树为空
                if (!(x = x->succ())) break; // 回溯 (含抵达末节点的退出返回)
                backtrack = true; // 并设置回溯标识
            }
        }
    }
}
```
这里相当于将原辅助栈替换为了一个标志位 backtrack; 每当抵达一个节点, 借助该标识可判断此前是否刚做过一次自下而上的回溯; 若不是, 则按照中序遍历的策略优先遍历左子树, 反之若刚发生过一次回溯, 则意味着当前节点的左子树已经遍历完毕; 每个节点被访问完毕后, 都应该转向其在遍历序列中的直接后继

##### 迭代版后序遍历
后序遍历算法的递归版本中, 左右子树的递归遍历均严格不属于尾递归  
将树 T 画在二维平面上, 并假设所有节点和边均不透明, 从左侧水平向右看去, 未被遮挡的最高叶节点 v, 称作最高左侧可见叶节点 (HLVFL), 即为后序遍历首先访问的节点  
与先序中序类似, 自底向上的沿着该通路, 整个后序遍历序列也可分解为若干个片段; 在每一片段, 分别起始于通路上的一个节点, 并包括三步: 访问当前节点, 遍历以其右兄弟 (若存在) 为根的子树, 以及向上回溯至其父节点 (若存在) 并转入下一片段
```
// 在以S栈顶节点为根的子树中，找到最高左侧可见叶节点
template <typename T> static void gotoHLVFL(Stack<BinNodePosi(T)>& S) { // 沿途所遇节点依次入栈
   while (BinNodePosi(T) x = S.top()) // 自顶而下，反复检查当前节点 (即栈顶)
      if (HasLChild(*x)) { // 尽可能向左
         if (HasRChild(*x)) S.push ( x->rc ); // 若有右孩子，优先入栈
         S.push(x->lc); // 然后才转至左孩子
      } else // 实不得已
         S.push(x->rc); // 才向右
   S.pop(); // 返回之前, 弹出栈顶的空节点
}
// 二叉树的后序遍历 (迭代版)
template <typename T, typename VST> void travPost_I(BinNodePosi(T) x, VST& visit) {
   Stack<BinNodePosi(T)> S; // 辅助栈
   if (x) S.push (x); // 根节点入栈
   while (!S.empty()) {
      if (S.top() != x->parent) // 若栈顶非当前节点之父 (则必为其右兄), 此时需
         gotoHLVFL (S); // 在以其右兄为根之子树中, 找到HLVFL (相当于递归深入其中)
      x = S.pop(); visit (x->data); // 弹出栈顶 (即前一节点之后继), 并访问之
   }
}
```

##### 层次遍历
在所谓的广度优先遍历或者层次遍历 (level-order traversal) 中, 确定节点访问次序可以概括为: 先上后下, 先左后右
```
// level-order traversal
              |
          --- i ------------------
          |                      |
      --- d ------           --- l ---
      |          |           |       |
  --- c      --- h       --- k   --- n ---
  |          |           |       |       |
  a ---  --- f ---       j       m   --- p
      |  |       |                   |
      b  e       g                   o

后序遍历顺序: -> i -> d -> l -> c -> h -> k -> n -> a -> f -> j -> m -> p -> b -> e -> g -> o ->
```
###### 算法实现
```
// 二叉树层次遍历算法
template <typename T> template <typename VST> void travLevel( VST& visit) {
    Queue<BinNodePosi(T)> Q; // 辅助队列
    Q.enqueue(this); // 根节点入队
    while (!Q.empty()) { // 在队列变空前反复迭代
        BinNodePosi(T) x = Q.dequeue();
        visit(x->data); // 取出队首节点并访问之
        if (HasLChild(*x)) Q.enqueue(x->lc); // 左孩子入队
        if (HasRChild(*x)) Q.enqueue(x->rc); // 右孩子入队
    }
}
```

###### 完全二叉树
完成二叉树的叶节点只能出现在最底层的两层, 且最底层叶节点均处于次底层叶节点的左侧; 完全二叉树的节点数在 $ 2^h $ 至 $ 2_{h+1} - 1 $ 之间, 即规模为 n 的完全二叉树的高度 $ h = \lfloor log_2n \rfloor = O(logn) $; 另外, 叶节点虽不至于少于内部节点, 但至多多出一个
###### 满二叉树
完全二叉树的一种特例是满二叉树, 其所有叶节点同处于最底层, 即二叉树的每一层的节点数都应达到饱和; 高度为 h 的满二叉树由 $ 2_{h+1} - 1 $ 节点组成, 其中叶子节点正好比内部节点多一个

#### Huffman 编码
TODO
