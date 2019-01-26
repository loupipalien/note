### 图
图属于非线性结构 (non-linear structure); 一般性的二元关系属于图论 (graph theory) 的研究范畴
#### 概述
##### 图
图 (graph) 可定义为 G = (V, E); 其中, 集合 V 中的元素称作顶点 (vertex), 集合 E 中的元素分别对应于 V 中某一对顶点 (u, v), 表示它们之间存在某种关系, 称作边 (edge); 约定 V 和 E 为有限集, 通常将其规模分别记作 n = |V| 和 e = |E|

##### 无向图, 有向图, 混合图
若边 (u, v) 所对应的顶点 u 和 v 的次序无所谓, 则称作无向边 (undirected edge), 反之 u 和 v 不对等则称 (u, v) 为有向边 (directed edge); 无向边 (u, v) 可以记作 (v, u), 而有向边 (u, v) 和 (v, u) 则不可混淆; 这里约定, 有向边 (u, v) 从 u 指向 v, 其中 u 称作该边的起点 (origin) 或尾顶点 (tail), 而 v 称作该边的终点 (destination), 或头顶点 (head)  
若 E 中各边均无方向, 则称 G 为无向图 (undirected graph, 简称 undigraph); 反之, 若 E 中只含有向边, 则 G 称作有向图 (directed graph, 简称 digraph); 若 E 同时包含无向边和右向边, 则 G 称作混合图 (mixed graph); 相对而言, 有向图的通用性更强, 因为无向图和混合图都可转化为有向图

##### 度
对于任何边 e = (u, v), 称顶点 u 和 v 彼此邻接 (adjacent), 互为邻居; 而它们都于边 e 彼此关联 (incident); 在无向图中, 与顶点 v 关联的边数称作 v 的度数 (degree), 记作 deg(v); 对于右向边 e = (u, v), e 称作 u 的出边 (outgoing edge), v 的入边 (incoming edge), v 的出边总数称作其出度 (out-degree), 记作 outdeg(v), 入边总数称作其入度 (in-degree), 记作 indeg(v)

##### 简单图
联接于同一顶点之间的边称作自环 (self-loop), 不含任何自环的图称作简单图 (simple graph)

##### 通路与环路
路径和通路 (path), 就是由 m + 1 个顶点与 m 条边交替而成的一个序列:
$$ \pi = \{v_0, e_1, v_1, e_2, v_2, ..., e_m, v_m\} $$
且对于任意 $ 0 < i \le m $ 都有 $ e_i = (v_{i-1}, v_i) $; 也就是说, 这些边依次的首尾相联, 其中沿途边的总数为 m, 亦称作通路的长度, 记作 $ |\pi| = m $; 为简化描述, 也可依次给出通路沿途的各个顶点, 而省略联接于其间的边
$$ \pi = \{v_0, v_1, v_2, ..., v_m\} $$
沿途顶点互异的通路称作简单通路 (simple path), 特别的对于长度 $ m \ge 1$ 的通路, 若起止顶点相同 (即 $ v_0 = v_m $, 则称作环路 (cycle); 不含任何环路的有向图, 称作有向无环图 (directed acyclic graph, DAG);  若环路上的各边互异, 沿途除了 $ v_0 = v_m $ 外所有顶点互异, 则称作简单环路 (simple cycle), 特别的经过各边一次且恰好一次的环路称作欧拉环路 (Eulerian tour), 而经过图中各顶点一次且恰好一次的环路称作哈密尔顿环路 (Hamiltonian tour)

##### 带权网络
为每一边 e 指定一个权重 (weight), 记作 wt(e); 各边均带有权重的图, 称作带权图 (weighted graph) 或带权网络 (weighted network), 记作 G(V, E, wt())

##### 复杂度
对于无向图, 每一对顶点至多贡献一条边, 故总共不超过 n(n-1)/2 条边; 对于有向图, 每一对顶点都可能贡献 (互逆的) 两条边, 因此至多可有 n(n-1) 条边

#### 抽象数据类型
##### 操作接口
- 支持的边操作接口

| 操作接口 | 功能描述 |
| :--- | :--- |
| e() | 边总数 \|E\| |
| exist(v, u) | 判断联边 (v, u) 是否存在 |
| insert(v, u) | 引入从顶点 v 到 u 的联边 |
| remove(v, u) | 删除从顶点 v 到 u 的联边 |
| type(v, u) | 边在遍历树中所属的类型 |
| edge(v, u) | 边所对应的数据域 |
| weight(v, u) | 边的权重 |

- 支持的顶点的操作接口

| 操作接口 | 功能描述 |
| :--- | :--- |
| n() | 顶点总数 \|V\| |
| insert(v) | 在顶点集 V 中插入新顶点 v |
| remove(v) | 将顶点 v 从顶点集中删除 |
| inDegree(v), outDegree(v) | 顶点 v 的入度和出度 |
| firstNbr(v) | 顶点 v 的首个邻接顶点 |
| nextNbr(v, u) | 在 v 的邻接顶点中, u 的后继 |
| status(v) | 顶点 v 的状态 |
| dTime(v), fTime(v) | 顶点 v 的时间标签 |
| parent(v) | 顶点 v 在遍历树中的父节点 |
| priority(v) | 顶点 v 在遍历树中的权重 |

##### Graph 模板类
```
typedef enum { UNDISCOVERED, DISCOVERED, VISITED } VStatus; //顶点状态
typedef enum { UNDETERMINED, TREE, CROSS, FORWARD, BACKWARD } EType; //边在遍历树中所属的类型

template <typename Tv, typename Te> //顶点类型、边类型
class Graph { //图Graph模板类
private:
   void reset() { //所有顶点、边的辅助信息复位
      for ( int i = 0; i < n; i++ ) { //所有顶点的
         status ( i ) = UNDISCOVERED; dTime ( i ) = fTime ( i ) = -1; //状态，时间标签
         parent ( i ) = -1; priority ( i ) = INT_MAX; //（在遍历树中的）父节点，优先级数
         for ( int j = 0; j < n; j++ ) //所有边的
            if ( exists ( i, j ) ) type ( i, j ) = UNDETERMINED; //类型
      }
   }
   void BFS ( int, int& ); //（连通域）广度优先搜索算法
   void DFS ( int, int& ); //（连通域）深度优先搜索算法
   void BCC ( int, int&, Stack<int>& ); //（连通域）基于DFS的双连通分量分解算法
   bool TSort ( int, int&, Stack<Tv>* ); //（连通域）基于DFS的拓扑排序算法
   template <typename PU> void PFS ( int, PU ); //（连通域）优先级搜索框架
public:
// 顶点
   int n; //顶点总数
   virtual int insert ( Tv const& ) = 0; //插入顶点，返回编号
   virtual Tv remove ( int ) = 0; //删除顶点及其关联边，返回该顶点信息
   virtual Tv& vertex ( int ) = 0; //顶点v的数据（该顶点的确存在）
   virtual int inDegree ( int ) = 0; //顶点v的入度（该顶点的确存在）
   virtual int outDegree ( int ) = 0; //顶点v的出度（该顶点的确存在）
   virtual int firstNbr ( int ) = 0; //顶点v的首个邻接顶点
   virtual int nextNbr ( int, int ) = 0; //顶点v的（相对于顶点j的）下一邻接顶点
   virtual VStatus& status ( int ) = 0; //顶点v的状态
   virtual int& dTime ( int ) = 0; //顶点v的时间标签dTime
   virtual int& fTime ( int ) = 0; //顶点v的时间标签fTime
   virtual int& parent ( int ) = 0; //顶点v在遍历树中的父亲
   virtual int& priority ( int ) = 0; //顶点v在遍历树中的优先级数
// 边：这里约定，无向边均统一转化为方向互逆的一对有向边，从而将无向图视作有向图的特例
   int e; //边总数
   virtual bool exists ( int, int ) = 0; //边(v, u)是否存在
   virtual void insert ( Te const&, int, int, int ) = 0; //在顶点v和u之间插入权重为w的边e
   virtual Te remove ( int, int ) = 0; //删除顶点v和u之间的边e，返回该边信息
   virtual EType& type ( int, int ) = 0;  //边(v, u)的类型
   virtual Te& edge ( int, int ) = 0; //边(v, u)的数据（该边的确存在）
   virtual int& weight ( int, int ) = 0; //边(v, u)的权重
// 算法
   void bfs ( int ); //广度优先搜索算法
   void dfs ( int ); //深度优先搜索算法
   void bcc ( int ); //基于DFS的双连通分量分解算法
   Stack<Tv>* tSort ( int ); //基于DFS的拓扑排序算法
   void prim ( int ); //最小支撑树Prim算法
   void dijkstra ( int ); //最短路径Dijkstra算法
   template <typename PU> void pfs ( int, PU ); //优先级搜索框架
};
```

#### 邻接矩阵
##### 原理
邻接矩阵 (adjacent matrix) 是图 ADT 最基本的实现方式; 使用方阵 A[n][n] 表示 n 个顶点构成的图, 其中每个单元, 各自负责描述一对顶点之间可能存在的邻接关系
```
(a) undigraph          (b) digraph              (c) network
  --- A ---             --> A ---               -7-> A -3/4-
  |       |             |       |               |          |
  B ----- C             B ----> C               B -- 9 --> C
  |                     |                       |
  --- D                 --- D                   -2/5- D
```
对于无权图, 存在 (不存在) 从顶点 u 到 v 的边, 当且仅当 A[u][v] = 1 (0); 这一表示方式可推广至带权网络, 居正各单元可从布尔型改为整型或浮点型, 记录所对应边的权重; 对于不存在的表, 通常统一取值为 0 或 $ \infty $
$$
\begin{array}{c}
    \begin{array}{c|cccc}
    \text{0} & A & B & C & D\\
    \hline
    A & - & 1 & 1 & -\\
    B & 1 & - & 1 & 1\\
    C & 1 & 1 & - & -\\
    D & - & 1 & - & -
    \end{array}
    &
    \begin{array}{c|cccc}
    \text{0} & A & B & C & D\\
    \hline
    A & - & - & 1 & -\\
    B & 1 & - & 1 & 1\\
    C & 1 & - & - & -\\
    D & - & 1 & - & -
    \end{array}
    &
    \begin{array}{c|cccc}
    \infty & A & B & C & D\\
    \hline
    A & - & - & 3 & -\\
    B & 7 & - & 9 & 2\\
    C & 4 & - & - & -\\
    D & - & 5 & - & -
    \end{array}
\end{array}
$$

##### 实现
```
#include "Vector/Vector.h" //引入向量
#include "Graph/Graph.h" //引入图ADT

template <typename Tv> struct Vertex { //顶点对象（为简化起见，并未严格封装）
   Tv data; int inDegree, outDegree; VStatus status; //数据、出入度数、状态
   int dTime, fTime; //时间标签
   int parent; int priority; //在遍历树中的父节点、优先级数
   Vertex ( Tv const& d = ( Tv ) 0 ) : //构造新顶点
      data ( d ), inDegree ( 0 ), outDegree ( 0 ), status ( UNDISCOVERED ),
      dTime ( -1 ), fTime ( -1 ), parent ( -1 ), priority ( INT_MAX ) {} //暂不考虑权重溢出
};

template <typename Te> struct Edge { //边对象（为简化起见，并未严格封装）
   Te data; int weight; EType type; //数据、权重、类型
   Edge ( Te const& d, int w ) : data ( d ), weight ( w ), type ( UNDETERMINED ) {} //构造
};

template <typename Tv, typename Te> //顶点类型、边类型
class GraphMatrix : public Graph<Tv, Te> { //基于向量，以邻接矩阵形式实现的图
private:
   Vector< Vertex< Tv > > V; //顶点集（向量）
   Vector< Vector< Edge< Te > * > > E; //边集（邻接矩阵）
public:
   GraphMatrix() { n = e = 0; } //构造
   ~GraphMatrix() { //析构
      for ( int j = 0; j < n; j++ ) //所有动态创建的
         for ( int k = 0; k < n; k++ ) //边记录
            delete E[j][k]; //逐条清除
   }
// 顶点的基本操作：查询第i个顶点（0 <= i < n）
   virtual Tv& vertex ( int i ) { return V[i].data; } //数据
   virtual int inDegree ( int i ) { return V[i].inDegree; } //入度
   virtual int outDegree ( int i ) { return V[i].outDegree; } //出度
   virtual int firstNbr ( int i ) { return nextNbr ( i, n ); } //首个邻接顶点
   virtual int nextNbr ( int i, int j ) //相对于顶点j的下一邻接顶点（改用邻接表可提高效率）
   { while ( ( -1 < j ) && ( !exists ( i, --j ) ) ); return j; } //逆向线性试探
   virtual VStatus& status ( int i ) { return V[i].status; } //状态
   virtual int& dTime ( int i ) { return V[i].dTime; } //时间标签dTime
   virtual int& fTime ( int i ) { return V[i].fTime; } //时间标签fTime
   virtual int& parent ( int i ) { return V[i].parent; } //在遍历树中的父亲
   virtual int& priority ( int i ) { return V[i].priority; } //在遍历树中的优先级数
// 顶点的动态操作
   virtual int insert ( Tv const& vertex ) { //插入顶点，返回编号
      for ( int j = 0; j < n; j++ ) E[j].insert ( NULL ); n++; //各顶点预留一条潜在的关联边
      E.insert ( Vector<Edge<Te>*> ( n, n, ( Edge<Te>* ) NULL ) ); //创建新顶点对应的边向量
      return V.insert ( Vertex<Tv> ( vertex ) ); //顶点向量增加一个顶点
   }
   virtual Tv remove ( int i ) { //删除第i个顶点及其关联边（0 <= i < n）
      for ( int j = 0; j < n; j++ ) //所有出边
         if ( exists ( i, j ) ) { delete E[i][j]; V[j].inDegree--; } //逐条删除
      E.remove ( i ); n--; //删除第i行
      Tv vBak = vertex ( i ); V.remove ( i ); //删除顶点i
      for ( int j = 0; j < n; j++ ) //所有入边
         if ( Edge<Te> * e = E[j].remove ( i ) ) { delete e; V[j].outDegree--; } //逐条删除
      return vBak; //返回被删除顶点的信息
   }
// 边的确认操作
   virtual bool exists ( int i, int j ) //边(i, j)是否存在
   { return ( 0 <= i ) && ( i < n ) && ( 0 <= j ) && ( j < n ) && E[i][j] != NULL; }
// 边的基本操作：查询顶点i与j之间的联边（0 <= i, j < n且exists(i, j)）
   virtual EType& type ( int i, int j ) { return E[i][j]->type; }  //边(i, j)的类型
   virtual Te& edge ( int i, int j ) { return E[i][j]->data; } //边(i, j)的数据
   virtual int& weight ( int i, int j ) { return E[i][j]->weight; } //边(i, j)的权重
// 边的动态操作
   virtual void insert ( Te const& edge, int w, int i, int j ) { //插入权重为w的边e = (i, j)
      if ( exists ( i, j ) ) return; //确保该边尚不存在
      E[i][j] = new Edge<Te> ( edge, w ); //创建新边
      e++; V[i].outDegree++; V[j].inDegree++; //更新边计数与关联顶点的度数
   }
   virtual Te remove ( int i, int j ) { //删除顶点i和j之间的联边（exists(i, j)）
      Te eBak = edge ( i, j ); delete E[i][j]; E[i][j] = NULL; //备份后删除边记录
      e--; V[i].outDegree--; V[j].inDegree--; //更新边计数与关联顶点的度数
      return eBak; //返回被删除边的信息
   }
};
```
在内部将所有顶点组织为一个向量 V[], 并通过嵌套定义, 将所有的 (潜在的) 边组织成一个二维向量 E[][], 即邻接矩阵; 每个顶点统一表示为 Vertex 对象, 每条边统一表示为 Edge 对象; 边对象的属性 weight 统一为整型, 可用于表示无权图, 也可表示带权网络

##### 时间性能
TODO

##### 空间性能
TODO

#### 邻接表
##### 原理
即便就有向图而言, $ \Theta (n^2)$ 的空间仍然有改进的余地; 实际上, 如此大的空间足以容纳所有潜在的边; 然而实际中处理的图, 所含的边通常远远少于 $ O(n^2) $; 由此可见邻接矩阵的空间效率极低, 是因为其中大量单元所对应的边通常并未在图中出现; 可以将向量换为列表来实现图的表示, 称作邻接表 (adjacent list)
```
        --- A ---            | A  B  C  D       [A | * ] -> [&B | 9 | * ] -> [&C | 3 | * ] -> [&D | 5 | ^ ]
        |   |   |         ---------------       
        9   3   5          A |    9  3  5       [B | * ] -> [&A | 9 | * ] -> [&B | 7 | * ] -> [&D | 2 | ^ ]
        |   |   |          B |9   7     2       
    --- B   C   D          C |3                 [C | * ] -> [&A | 3 | ^]
    |   |-- 2 --|          D |5   2             
    - 7 -                                       [D | * ] -> [&A | 5 | *] -> [&B | 2 | ^]
    (a) undigraph       (b) adjacent matrix                 (c) adjacent list
```

##### 复杂度
邻接列表数等于顶点总数 n, 每条边在其中仅存放一次 (有向图) 或两次 (无向图), 故空间总量为 $ O(n + e) $, 与图自身规模相当, 较之邻接矩阵有很大的改进; 空间性能的这一改进需以某方面时间性能的降低为代价, 但与顶点相关的操作接口, 时间性能依然保持甚至提高; 总体而言, 邻接表的效率较之邻接矩阵更高

#### 图遍历算法概述
图的遍历可以理解为, 将非线性结构转化为半线性结构的过程; 经遍历而确定的边类型中, 最重要的一类即树边, 它们与所有顶点共同构成了原图的一颗支撑树, 称作遍历树 (traversal tree); 图的遍历更加强调对处于特定状态顶点的甄别与查找, 故也称作图搜索 (graph search)

#### 广度优先搜索
各种图搜索之间的区别体现为边分类结果的不同, 以及所得遍历树的结构差异, 其决定因素在于搜索过程中的每一步迭代, 将依照何种策略来选取下一个接受访问的顶点  
通常都是选取某个已访问到的顶点的邻居, 同一顶点所有邻居之间的优先级, 在多数遍历中不必讲究; 因此实质性的差异体现在, 当有多个顶点已被访问到, 应该优先从谁的邻居中选取下一个顶点  
广度优先搜索 (breadth-first search, BFS) 采用的策略, 可以概括为: **越早被访问到的顶点, 其邻居越优先被选用**; 于是始自图中顶点 s 的 BFS 搜索, 将首先访问顶点 s: 再依次访问 s 所有尚未访问到的邻居: 再按后者被访问的先后次序, 逐个访问它们的邻居, 如此不断; 在所有已访问到的顶点中, 仍有邻居尚未访问者, 构成所谓的波峰集 (frontier); 于是 BFS 搜索过程也可等效地理解为: **反复从波峰集中找到最早被访问到顶点 v, 若其邻居均已被访问到, 则将其逐出波峰集, 否则随意选出一个尚未访问到的邻居, 并将其加入到波峰集中**  
由于每一步迭代都有一个顶点被访问, 故至多迭代 O(n) 步; 另一方面, 因为不会遗漏每个刚刚被访问顶点的任何邻居, 故对于无向图必能覆盖 s 所属的连通分量 (connected component), 对于有向图必能覆盖以 s 为起点的可达分量 (reachable component)

##### 实现
```
// 广度优先搜索 BFS 算法 (全图)
template <typename Tv, typename Te> void Graph<Tv,Te>::bfs(int s) {
    reset();
    int clock = 0;
    int v = s; // 初始化
    do {
        if (UNDISCOVERED == status(v)) { // 一旦遇到尚未发现的顶点
            BFS(v, clock); // 即从该顶点出发启动一次 BFS
        }
    } while(s !=(v = (++v % n))); // 按序号检查, 故不重不漏
}

// 广度优先搜索 BFS 算法 (单个连通域)
template <typename Tv, typename Te> void Graph<Tv,Te>::BFS(int s, int& clock) {
    Queue<int> Q; // 引入辅助队列
    status(v) = DISCOVERED;
    Q.enqueue(v); // 初始化起点
    while (!Q.empty()) { // 在 Q 变空之前, 不断循环
        int v = Q.dequeue(); // 取出队首顶点 v
        dTime(v) = ++clock;
        for (int u = firstNbr;  -1 < u; u = nextNbr(v, u)) { // 枚举 v 的所有邻居
            if (UNDISCOVERED  == status(u)) { // 如果尚未发现该顶点
                status(u) = DISCOVERED;
                Q.enqueue(u);
                type(v, u) = TREE; // 引入树边扩展支撑树
                parent(u) = v;
            } else { // 若 u 已被发现, 或者已访问完毕
                type(v, u) = CROSS; // 将 (v, u) 归类于跨边
            }
        }
        status(v) = VISITED; // 至此当前顶点访问完毕
    }
}
```
若顶点 u 处于 UNDISCOVERED 状态, 将边 (v,u) 标记为树边 (tree edge); 当顶点 u 已处于 DISCOVERED 状态 (无向图), 或者处于 VISITED 状态 (有向图), 则意味着边 (v, u) 不属于遍历树, 于是将该边归为跨边 (cross edge); BFS(s) 遍历完成后, 所有访问过的顶点通过 parent[] 指针依次联接, 从整体上给出了顶点 s 所属的某一连通或可达域的一颗遍历树, 称作广度优先搜索树, 简称为 BFS 树 (BFS tree)

##### 实例
TODO

BFS(s) 将覆盖起始顶点 s 所属的连通分量或可达分量, 但无法到达此外的顶点, 上层函数 bfs() 的作用即是处理多个连通分量或可达分量并存的情况; 如此 BFS() 调用所得的 BFS 树构成一个森林, 称作 BFS 森林 (BFS forest)

##### 复杂度
空间方面, BFS 搜索使用的空间主要消耗在用于维护顶点访问次序的辅助队列, 用于记录顶点和边状态的标识位向量, 累计 O(n) + O(n) + O(e) = O(n + e)  
时间方面, 首先复位需要 O(n + e) 的时间, bfs() 本身对所有顶点的枚举共需 O(n) 的时间, 在 BFS() 的所有调用中需耗费 (n + e) 的时间, 故 BFS 搜索总体需要 O(n + e) 的时间

##### 应用
可以解决连通域分解, 最短路径等问题

#### 深度优先搜索
##### 策略
深度优先搜索 (Depth-First Search, DSF) 选取下一顶点的策略, 可概括为: **优先选取最后一个被访问到的顶点的邻居**; 以顶点 s 为基点的 DFS 搜索, 将首先访问顶点 s, 再从 s 所有未访问到的另据中任取之一, 并以之为基点递归的执行 DFS 搜索

##### 实现
```
// 深度优先搜索 DFS 算法 (全图)
template <typename Tv, typename Te> void Graph<Tc,Te>::dfs(int s) {
    reset();
    int clock = 0;
    int v = s; // 初始化
    do {
        if (UNDISCOVERED == status(v)) { // 一旦遇到尚未发现的顶点
            DFS(v, clock); // 即从该顶点触发启动一次 DFS
        }
    } while(s != (v = (++v % n))); // 按序号检查, 故不重不漏
}

// 深度优先搜索 DFS 算法 (单个连通域)
template <typename Tv, typename Te> void Graph<Tv,Te>::DFS(int v, int& clock) {
    dTime(v) = ++clock;
    status(v) = DISCOVERED; // 发现当前顶点 v
    for (int u = firstNbr; -1 < u; u = nextNbr(v, u)) { // 枚举 v 的所有邻居 u
        switch(status(u)) { // 视其状态分别处理
            case UNDISCOVERED: // u 尚未发现, 意味支撑树可在此扩展
                type(v, u) = TREE; parent(u) = v; DFS(v, clock); break;
            case DISCOVERED: // u 已被发现当尚未访问完毕, 应属于被后代指向的祖先
                type(v, u) = BACKWARD; break;
            default:
                type(v, u) = (dTime(V) < dTime(u)) ? FORWARD: CROSS; break;
         }
    }
    status(v) = VISITED; fTime(v) = ++clock; // 至此当前顶点 v 访问完毕
}
```
在每一递归实例中, 都先将当前顶点 v 标记为 DISCOVERED 状态, 再逐一对其各邻居 u 的状态并做出相应处理, 待其所有邻居均已处理完毕, 将顶点 v 置为 VISITED 状态便可回溯  
若顶点 u 为 DISCOVERED 状态, 则将边 (v, u) 归为树边 (tree edge), 并将 v 记作 u 的父节点, 此后便可将 u 作为当前节点, 继续递归遍历  
若顶点 u 处于 DISCOVERED 状态, 则意味着此处发现一个有向环路, 此时 DFS 遍历树中 u 必为 v 的祖先, 故应将边 (v, u) 归为后向边 (back edge)  
这里为每个顶点 v 都记录了被发现和访问完成的时刻, 对应的时间区间 [dTime(v), fTime(v)] 称作 v 的活跃期 (active duration); 顶点 v 和 u 之间是否存在祖先/后代关系, 完全取决于二者的活跃期是否相互包含  
对于有向图, 顶点 u 还可能处于 VISITED 状态, 此时只要对比 v 和 u 的活跃期, 即可判定在 DFS 树中 v 是否为 u 的祖先, 若是则边 (v, u) 应归为前向边 (forward edge), 否则二者必然来自相互独立的两个分支, 边 (v, u) 应归为跨边 (cross edge)  
DFS(s) 返回后, 所有访问过的顶点通过 parent[] 指针依次联接, 从整体上给出了顶点 s 所属的某一连通或可达域的一颗遍历树, 称作深度优先搜索树, 简称为 DFS 树 (DFS tree)

##### 实例
TODO

##### 复杂度
空间方面, BFS 搜索使用的空间主要消耗于各顶点的时间标签和状态标记, 二者累计不超过累计 O(n) + O(e) = O(n + e)  
时间方面, 首先复位需要 O(n + e) 的时间, bfs() 本身对所有顶点的枚举共需 O(n) 的时间, 在 DFS() 的所有调用中需耗费 (n + e) 的时间, 故 DFS 搜索总体需要 O(n + e) 的时间

##### 应用
深度优先搜索是最为重要的图遍历算法, 可以用于寻找路径, 分解连通分量, 以及判定有向无环图等

#### 拓扑排序
每一顶点都不会通过边, 指向其在序列中的前驱顶点, 这样的一个序列, 称作原有向图的一个拓扑排序 (topological sorting)

##### 有向无环图
有向无环图的拓扑排序必然存在, 反之亦然; 这是因为有向无环图对应于偏序关系, 而拓扑排序对应于全序关系; 在顶点数目有限时, 与任一偏序相容的全序必然存在  
在任意有限偏序集中, 必有极值元素 (未必唯一): 相应的, 任一有向无环图, 也必包含入读为零的顶点, 否则每个顶点都至少有一条入边, 那么这也意味着图中包含环路; 于是, 只要将入度为 0 的顶点 m (及其关联边) 从图中取出, 则剩余的图依然是有向无环图, 故其拓扑排序也必然存在

##### 算法
同理, 有限偏序集中也必然存在极小元素 (未必唯一), 该元素作为顶点, 出度必然为零; 而在有向无环图的 DFS 搜索中, 首先因访问完成而转换至 VISITED 状态的顶点 m, 也必然具有这一性质  
进一步的, 根据 DFS 搜索的特性, 顶点 m (及其关联边) 对此后的搜索过程将不起任何作用, 于是下一转至 VISITED 状态的顶点可等效的理解为是从图中剔除顶点 m (及其关联边) 之后的出度为零者, 在拓扑排序中, 该顶点应为顶点 m 的前驱; 由此可见, DFS 搜索过程中各顶点被标记为 VISITED 的次序, 恰好 (按逆序) 给出了一个拓扑排序

##### 实现
```
// 基于DFS的拓扑排序算法
template <typename Tv, typename Te> Stack<Tv>* Graph<Tv, Te>::tSort ( int s ) { //assert: 0 <= s < n
   reset();
   int clock = 0;
   int v = s;
   Stack<Tv>* S = new Stack<Tv>; // 用栈记录排序顶点
   do {
      if (UNDISCOVERED == status (v))
         if (!TSort(v, clock, S)) { // clock 并非必需
            while (!S->empty()) // 任一连通域（亦即整图）非DAG
               S->pop(); break; // 则不必继续计算，故直接返回
         }
   } while (s != (v = ( ++v % n )));
   return S; // 若输入为 DAG, 则 S 内各顶点自顶向底排序; 否则（不存在拓扑排序）,S 空
}

// 基于DFS的拓扑排序算法（单趟）
template <typename Tv, typename Te> bool Graph<Tv, Te>::TSort ( int v, int& clock, Stack<Tv>* S ) { //assert: 0 <= v < n
   dTime(v) = ++clock;
   status(v) = DISCOVERED;
   //发现顶点v
   for (int u = firstNbr(v); -1 < u; u = nextNbr(v, u)) // 枚举 v 的所有邻居 u
      switch (status(u)) { // 并视 u 的状态分别处理
         case UNDISCOVERED:
            parent(u) = v;
            type(v, u) = TREE;
            if (!TSort(u, clock, S)) // 从顶点 u 处出发深入搜索
               return false; // 若 u 及其后代不能拓扑排序 (则全图亦必如此), 故返回并报告
            break;
         case DISCOVERED:
            type (v, u) = BACKWARD; // 一旦发现后向边 (非 DAG)
            return false; // 则不必深入, 故返回并报告
         default: // VISITED (digraphs only)
            type (v, u) = (dTime(v) < dTime (u)) ? FORWARD : CROSS;
            break;
      }
   status (v) = VISITED;
   S->push(vertex(v)); // 顶点被标记为 VISITED 时, 随即入栈
   return true; // v 及其后代可以拓扑排序
}
```

##### 实例
TODO

##### 复杂度
这里引入了额外的栈, 规模不超过顶点总数 O(n), 但空间复杂度与基本深度优先算法一致, 仍为 O(n + e), 时间复杂度也与基本深度优先算法一致, 为 O(n + e)

#### 双连通域分解
##### 关节点与双连通图
考察无向图 G, 若删除顶点 v 后 G 所包含的连通域增多, 则 v 称作切割节点 (cut vertex) 或 (articulation point); 反之不包含关节点的图称为双连通图, 任一无向图都可视作若干个极大的双连通子图组成, 这样的每一子图都称作原图的一个双连通域 (bi-connected component)

##### 蛮力算法
通过 BFS 或 DFS 搜索统计出图 G 所含连通域的数目, 然后逐一枚举每个顶点 v, 暂时将其从图 G 中删去, 并再次通过搜索统计图 G\\{v} 所含的连通数目; 当且仅当图 G\\{v} 包含的连通域多于图 G 时, 顶点 v 是关节点

##### 可行算法
- DFS 树中的叶节点, 绝不可能是原图中的关节点, 此类顶点的删除不会影响 DFS 树的连通性, 也不会影响原图的连通性
- DFS 树的根节点若至少有两个分支, 则必是一个关节点
- 假定内部顶点 C, 若移除顶点 C 导致其一颗真子树与其真祖先之间无法连通, 则 C 必为关节点; 反之, 若 C 的所有真子树都能与 C 的某一祖先连通, 则 C 就不可能是关节点; 因此只要在 DFS 搜索中记录并更新各顶点 v 所能 (经后向边) 连通的最高祖先 (highest connected ancestor, HCA) hca[v], 即可及时认定关节点, 并报告对应的双连通域

##### 实现
```
// 基于 DFS 的 BCC 分解算法
template <typename Tv, typename Te> void Graph<Tv,Te>::bbc(int s) {
    reset();
    int clock = 0;
    int v = s;
    Stack<int> S; // 用栈记录已访问的顶点
    do {
        if (UNDISCOVERED == status(v)) { // 一旦发现未发现的顶点 (新连通分量)
            BBC(v, clock, S); // 从该顶点出发启动一次 BBC
            S.pop(); // 遍历返回后, 弹出栈中的最后一个顶点, 当前连通域的起点
        }
    }
}

// 利用闲置的 fTime[] 充当 hca[]
#define hca(x) (fTime(x))

template <typename Tv, typename Te> void Graph<Tv,Te>::BBC(int v, int& clock, Stack<int>& S) {
    hca(v) = dTime(v) = ++clock;
    status(v) = DISCOVERED;
    S.pop(v); // v 被发现并入栈
    for (int u = firstNbr(v); -1 < u; u = nextNbr(v, u)) { // 枚举 v 的所有邻居 u
        switch(status(u)) { // 并视 u 的状态分别处理
            case UNDISCOVERED:
                parent(u) = v;
                type(v, u) = TREE;
                BCC(u, clock, S); // 从顶点 u 处深入
                if (hca(u) < dTime(v)) { // 遍历返回后若发现 u (通过后向边) 可指向 v 的真祖先
                    hca(v) = min(hca(v), hca(u)); // 则 v 亦必如此
                } else { // 否则, 以 v 为关节点 (u 以下即是一个 BBC, 且其中顶点此时集中于栈 S 的顶部)
                    while(v != S.pop()); // 依次弹出当前 BBC 中的顶点
                    S.push(v); // 最后一个顶点 (关节点重新入栈), 分摊一次不足
                }
                break;
            case DISCOVERED:
                type(v, u) = BACKWARD; // 标记 (v, u), 并按照 "越小越高" 的原则
                if (u != parent(v)) {
                    hca(v) = min(hca(v), dTime(u)); // 更新 hca(v)
                }
                break;
            default:
                type(v, u) = (dTime(v) < dTime(u)) ? FORWARD : CROSS;
                break;
        }
    }
    status(v) = VISITED; // 对 v 的访问结束
}
```

##### 实例
TODO

##### 复杂度
TODO

#### 优先级搜索
##### 优先级与优先级数
BFS 与 DFS 的差异体现在每一步迭代中对新顶点的选取策略不同, 每一种选取策略等效于给所有顶点赋予不同的优先级, 而且随着算法的推进不断调整, 每一步迭代所选取的顶点都是当时优先级最高者; 按照这种理解, 所有的图搜素都可纳入统一框架, 由于优先级的重要性, 故称作优先级搜索 (priority-first search, PFS); 基于以上, 提供了 priority() 接口, 以支持对顶点优先级数 (priority number) 的读取与修改

##### 基本框架
```
// 优先级搜索 (全图)
template <typename Tv, typename tE> template <template PU> void Graph<Tv,Te>::pfs(int s, PU prioUpdater) {
    reset();
    int v = s;
    do {
        if (UNDISCOVERED == status(v)) { // 一旦遇到为发现的顶点
            PFS(v, prioUpdater); // 即从该顶点出发启动一次 PFS
        }
    } while (s != (v = (++v % n))); // 按序号检查
}

// 优先级搜索 (单个连通域)
template <typename Tv, typename Te> template <typename PU> void Graph<Tv,Te>::PFS(int s, PU prioUpdater) {
    priority(s) = 0;
    status(s) = VISITED;
    parent(s) = -1; // 初始化, 起点 s 加入 PFS 树中
    while(1) { // 下一顶点和边加至 PFS 树中
        for (int w = firstNbr(v); -1 < w; w = nextNbr(v, w)) { // 枚举 v 的所有邻居 w
            prioUpdater(this, s, w); // 更新顶点 w 的优先级及其父顶点
        }
        for (int shortest = INT_MAX, w = 0; w < n; w++) {
            if (UNDISCOVERED == status(w)) { // 从尚未加入遍历树的顶点中选出下一个
                if (shortest > priority(w)) {
                    shortest = priority(w);
                    s = w; // 优先级最高的顶点 s
                }
            }
        }
        if (VISITED == status(s)) break; // 直至所有顶点均已加入
        status(s) = VISITED;
        type(parent(s), s) = TREE; // 将 s 及其父的联边加入遍历树
    }
}
```
这里借助函数对象 prioUpdater 是算法可以根据不同的问题需求, 简明的描述和实现对应的更新策略; 具体的只需重定义 prioUpdater 对象即可, 而不必重复实现公共部分

##### 复杂度
PFS 搜索由两重循环构成, 若采用邻接表实现方式, 同时假定 prioUpdater() 只需要常数时间, 则前一循环累计为所有顶点的出度总和 O(e), 后一循环固定迭代 n 次, 故总体时间复杂度为 $ O(n^2) $

#### 最小支撑树
TODO

#### 最短路径
TODO
