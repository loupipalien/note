### 高级搜索树
#### 伸展树
伸展树 (splay tree) 也是平衡二叉搜索树的一种, 相对于 AVL 树实现更为简洁且时刻保持着全树的平衡, 也能够保持分摊意义上的高效率, 且不用对基本二叉树做额外的扩展, 适用性更强
#### 局部性
信息处理的模式是将所有数据项视作是一个集合, 并将器组织为某种适宜的数据结构, 进而借助操作接口高效访问  
通常在任意数据结构的生命周期内, 不仅执行不同操作的概览不均衡, 而且各操作之间具有极强的相关性, 并在整体上多呈现出极强的规律性; 这就是所谓的 "数据局部性" (data locality)
- 刚刚被访问过的元素, 极有可能在补不久之后再次被访问到
- 将被访问的下一元素, 极有可能就处于之前被访问过的某个元素的附近

对于二叉搜索树的数据局部性具体表现为
- 刚刚被访问过的节点, 极有可能在补不久之后再次被访问到
- 将被访问的下一节点, 极有可能就处于之前被访问过的某个节点的附近
