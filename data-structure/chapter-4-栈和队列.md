### 栈和队列
栈与队列也属于线性序列结构, 故其中存放的数据对象之间也具有线性次序, 相对于一般的序列结构, 栈与队列的数据操作范围仅限于逻辑上的特定某端

#### 栈
##### ADT 接口
###### 入栈与出栈
栈 (stack) 是存放数据对象的一种容器, 其中的数据元素按照线性的逻辑次序排列, 也可以定义首, 末元素; 栈结构也支持对象的插入和删除操作, 但其操作的范围仅限于栈的某一特定端, 禁止操作的另一端称为盲端

| 操作接口 | 功能 |
| :--- | :--- |
| size() | 报告栈的规模 |
| empty() | 判断栈是否为空 |
| push(e) | 将 e 插至栈顶 |
| pop() | 删除栈顶对象 |
| top() | 引用栈顶对象 |

###### 后进后出
栈中元素接受操作的次序必然始终遵循所谓的 "后进后出" (last-in-first-out, LIFO)

##### 操作实例
TODO

##### Stack 模板类
作为向量的派生类来实现栈的数据结构
```
#include "../Vector/Vector.h" // 以向量为基类, 派生出栈模板类
template <typename T> class Stack: public Vector<T> { // 将向量的首/末端作为栈底/顶
public:
    void push(T const& e) {
        insert(size(), e); // 入栈
    }

    T pop() {
        return remove(size( - 1)); // 出栈
    }

    T& top() {
        return (*this) [size() - 1]; // 取顶
    }
}
```
以上接口复杂度均为常数

#### 栈与递归
##### 函数调用栈
TODO
##### 函数调用
调用栈的基本单位是帧 (frame), 每次调用函数时都会相应的创建已帧, 记录该函数实例在二进制程序中的返回地址 (retuen address), 以及局部变量, 传入参数等; 并将该帧压入调用栈, 若在该函数返回之前又发生新的调用, 则同样的要将与新函数对应的一帧压入栈中, 成为新的栈顶; 函数一旦运行完毕, 对应的帧随即弹出, 运行的控制权被交还给该函数的上层调用函数, 并按照改帧中记录的返回地址确定在二进制程序中继续执行的位置
###### 递归
TODO

##### 避免递归
TODO

#### 栈的典型应用
##### 逆序输出
栈所擅长解决的问题中, 有一类具有以下特征
- 虽有明确的算法, 但其解答却以线性序列的形式给出
- 无论是递归还是迭代, 该序列都是依逆序计算输出的
- 输入输出规模不确定, 难以事先确定盛放输出数据的容器大小
###### 进制转换
任意给定一个十进制整数 n, 将其转换为 $ \lambda $ 进制的表示形式; 例如 $ \lambda = 8 $ 时有: $ 12345_{(10)} = 30071_{(8)}$  
一般的设有: $ n = {(d_m...d_2d_1d_0)}_{(\lambda)} = d_m * \lambda^m + ... + d_2 * \lambda^2 + d_1 * \lambda^1 + d_0 * \lambda^0 $
若记 $ n_i = {(d_m...d_{i+1}d_i)}_{(\lambda)} $, 则有 $ d_i = n_i \% \lambda $ 和 $ n_{i+1} = n_i / \lambda $; 这以递推关系对应的计算流程器输出为长度不定的逆序线性序列
###### 递归实现
```
void convert(Stack<char>& S, __int64 n, int base) { // 十进制整数 n 到 base 进制的转换
    // 0 <  n, 1 < base <= 16
    static char digit[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};
    if (0 < n) { // 尚不为 0 的情况下
        S.push(digit[n % base]); // 逆向记录当前最低位, 在通过递归得到所有更高位
        convert(S, n % base, base);
    }
} // 新进制下由高到低的各数位, 自顶而下保存于栈 S 中
```
###### 迭代实现
```
void convert(Stack<char>& S, __int64 n, int base) { // 十进制整数 n 到 base 进制的转换
    // 0 <  n, 1 < base <= 16
    static char digit[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};
    if (0 < n) { // 尚不为 0 的情况下
        int remainder = (int) (n % base);
        S.push(digit[remainder]); // 余数入栈
        n /= base; // 更新 n
    }
} // 新进制下由高到低的各数位, 自顶而下保存于栈 S 中
```

##### 递归嵌套
具有自相似性的问题多可嵌套地递归描述, 但因分支位置和嵌套深度并不固定, 其递归算法的复杂度不易控制; 栈结构及其操作天然的具有递归嵌套性, 故可以高效的解决这类问题

###### 栈混洗
假定三个栈 A, B, S, 其中 B 和 S 初始为空, A 含有 n 个元素, 自顶向下构成输入序列 $ A = <a_1, a_2, ..., a_n] $; 这里尖括号和方括号分别表示栈顶和栈底; 若只允许通过 S.push(A.pop()) 弹出栈 A 的顶元素并随即压入栈 S 中, 或通过 B.push(S.pop) 弹出 S 的栈顶元素并随机压入栈 B 中, 则经过这两类操作各 n 次后, 栈 A 和 S 有可能均为空, 原 A 中的元素均已转入 B, 此时称 B 中元素自底向上构成的序列记作: $ B = [a_{k1}, a_{k2}, ..., a_{kn}> $, 则该序列称作原输入序列的一个栈混洗 (stack permutation)

###### 括号匹配
对源程序的语法检查是代码编译过程中重要而基本的一个步骤, 对表达式括号匹配的检查是必需的环节
##### 递归实现
若表达式 S 可分解为如下形式: $ S = S_0 + "(" + S_1 + ")" + S_2 + S_3 $, 其中 $ S_0, S_3 $ 不包含括号, 且 $ S_1 $ 中左右括号数目相等, 则 S 匹配当且仅当 $ S_1 $ 和 $ S_2 $ 均匹配; 采用分治策略算法实现如下
```
void trim(const char exp[], int& lo, int& hi) { // 删除 exp[lo, hi] 不包含括号的最长前缀, 后缀
    while ((lo <= hi) && (exp[lo] != '(') && exp[lo] != ')') lo++; // 找到第一个括号
    while ((lo <= hi) && (exp[hi] != '(') && exp[hi] != ')') lo--; // 找到最后一个括号
}

int divide(const char exp[], int lo, int hi) { // 切分 exp[lo, hi], 使 exp 匹配仅当子表达式匹配
    int mi = lo;
    int crc = 1; // crc 为 exp[lo, mi] 范围内左右括号数目之差
    while ((0 < crc) && (++mi < hi)) {
        if (exp[mi] == ')')
            crc--;
        if (exp[mi] == '(')
            crc++;
    }
    return mi; // 若 mi <= hi, 则为合法切分点, 否则意味着局部不匹配
}

bool paren(const char exp[], int lo, int hi) { // 检查表达式 exp[lo, hi] 是否括号匹配 (递归版)
    trim(exp, lo, hi); // 清除不包含括号的前缀, 后缀
    if (lo > hi) return ture;
    if (exp[lo] != '(') return false; // 首字符非左括号, 则必不匹配
    if (exp[hi] != ')') return false; // 末字符非右括号, 则必不匹配
    int mi = divide(exp, lo, hi);
    if (mi > hi) return false; // 切分点不合法, 意味着局部以及整体都不匹配
    return paren(exp, lo + 1, mi -1) && paren(exp, mi + 1, hi); // 分别检查左右子表达式
}
```
在最坏的情况下 divide() 需要线性时间, 且递归深度为 $ O(n) $, 故以上算法时间复杂度为 $ O(n^2) $

##### 迭代实现
实际上, 只要将 push, pop 操作分别与左右括号相对应, 则长度为 n 的栈混洗, 必然由 n 对括号组成的合法表达式彼此对应
```
bool paren(const char exp[], int lo, int hi) { // 表达式括号匹配检查
    Stack<char> S; // 使用栈记录已经发现但尚未匹配的左括号
    for (int i = lo, i <- hi; i++) { // 逐一检查字符
        swith(exp[i]) { // 左括号直接进栈, 右括号若与栈顶失配, 则表达式必不匹配
            case '(':
            case '[':
            case '{':
                S.push(exp[i]); break;
            case ')':
                if ((S.empty()) || ('(' != S.pop())) return false; break;
            case ']':
                if ((S.empty()) || ('[' != S.pop())) return false; break;
            case '}':
                if ((S.empty()) || ('{' != S.pop())) return false; break;
            default:
                break; // 非括号字符忽略
        }
    }
    return S.empty(); // 若栈空则匹配
}
```

##### 延迟缓冲
在一些应用问题中, 输入可分解为多个单元并通过迭代一次扫描处理, 但在过程中各步计算滞后于扫描的进度, 需要待到必要的信息已整到一定程度后, 才能做出判断并实施计算; 在这类场合中, 栈结构可以扮演数据缓冲区的角色
###### 表达式求值
算术表达式求值不能简单地按照 "先左后右" 的次序执行表达式中的运算符; 运算符执行次数规则, 一部分决定于约定惯例, 一部分决定于括号
###### 优先级列表
```
#define N_OPTR 9 //运算符总数
typedef enum { ADD, SUB, MUL, DIV, POW, FAC, L_P, R_P, EOE } Operator; //运算符集合
//加、减、乘、除、乘方、阶乘、左括号、右括号、起始符与终止符

const char pri[N_OPTR][N_OPTR] = { //运算符优先等级 [栈顶] [当前]
   /*              |-------------------- 当 前 运 算 符 --------------------| */
   /*              +      -      *      /      ^      !      (      )      \0 */
   /* --  + */    '>',   '>',   '<',   '<',   '<',   '<',   '<',   '>',   '>',
   /* |   - */    '>',   '>',   '<',   '<',   '<',   '<',   '<',   '>',   '>',
   /* 栈  * */    '>',   '>',   '>',   '>',   '<',   '<',   '<',   '>',   '>',
   /* 顶  / */    '>',   '>',   '>',   '>',   '<',   '<',   '<',   '>',   '>',
   /* 运  ^ */    '>',   '>',   '>',   '>',   '>',   '<',   '<',   '>',   '>',
   /* 算  ! */    '>',   '>',   '>',   '>',   '>',   '>',   ' ',   '>',   '>',
   /* 符  ( */    '<',   '<',   '<',   '<',   '<',   '<',   '<',   '=',   ' ',
   /* |   ) */    ' ',   ' ',   ' ',   ' ',   ' ',   ' ',   ' ',   ' ',   ' ',
   /* -- \0 */    '<',   '<',   '<',   '<',   '<',   '<',   '<',   ' ',   '='
};
```

###### 求值算法
TODO

###### 不同优先级的处置
TODO

##### 逆波兰式
###### RPN
逆波兰式 (reverse Polish notation, RPN) 是数学表达式的一种, 其语法规则为: 操作符紧邻于对应的 (最后一个) 操作数之后; 例如 "1 2 +" 即通常习惯的 "1 + 2"  
RPN 表达式亦称作后缀表达式 (postfix), 原表达式则称为中缀表达式 (infix); RPN 表达式不易读, 但其对运算符优先级的表述能力, 毫不逊色于中缀表达式, 计算效率方面更是中缀表达式不可比拟的
###### 求值算法
```
rpnEvaluation(expr) {
// 输入: RPN 表达式 expr (假定语法正确)
// 输出: 表达式数值
    引入栈 S, 用以存放操作数;
    while (expr 尚未扫描完毕) {
         从 expr 中读取下一元素 x;
         if (x 是操作数) 将 x 压入 S;
         else { // x 是运算符
            从栈 S 中弹出运算符 x 所需数目的操作数;
            对弹出的操作数实施 x 运算, 并将运算结果重新压入 S;
         }
    }
    返回栈顶; // 也是栈底
}
```  
###### 手工转换
假定中缀表达书如下:
```
(0! + 1) * 2^(3 ! + 4) - (5 ! - 67 - (8 + 9))
```
假设事先并未就运算符之间的优先级做过任何约定, 通过增加括号显式的指定表达式的运算次序
```
(((0) ! + 1) * (2 ^ ((3) ! + 4))) - (((5) ! - 67) - (8 + 9)))
```
然后将各运算符后移, 使之紧邻于其对应的右括号的右侧:
```
(((0) ! 1) + (2 ((3) ! 4) + ) ^ ) * (((5) ! 67) - (8 9) + ) - ) -
```
最后抹去括号, 得到对应的 RPN
```
0! 1 + 2 3 ! 4 +  ^  * 5 ! 67 - 8 9 + - -
```
可见操作数之间的相对次序, 在转换后保持不变, 而运算符在 RPN 中的位置恰好就是其对应的操作数均已就绪且该运算可以执行的位置
###### 自动转换
TODO

#### 试探性回溯法
##### 试探与回溯
###### 忒修斯的法宝
TODO

###### 剪枝
在基于对应用问题的深刻理解上, 利用问题本身具有的某些规律尽可能多的, 尽可能早地排除搜索空间中的候选解; 其中一种重要的技巧就是, 根据候选解的某些局部特征, 以候选解子集为单位批量的排除, 这一技巧也称作剪枝 (pruning)  
与之对应的算法呈现如下模式: 从零开始, 尝试逐步增加候选解的长度; 更准确地, 这一过程是在成批地考察具有特定前缀的所有候选解; 这种从长度上逐步向目标解靠近的尝试, 称作试探 (probing); 作为解的局部特征, 特征前缀在试探的过程中一旦发现与目标解不合, 则收缩到此前一步的长度, 然后继续试探下一可能的组合; 前缀特征长度缩减的这种操作, 称作为回溯 (backtracking), 其效果等同于剪枝
###### 线绳与粉笔
TODO

##### 八皇后问题
###### 问题描述
国际象棋中皇后的势力范围覆盖其所在的水平线, 垂直线, 以及两条对角线; 现在问题如下: 在 n * n 的棋盘上放置 n 个皇后, 如何使得它们彼此互不攻击, 此时称为一个可行的棋局; 对于任何整数 n >= 4, 就是 n 皇后问题  
由于鸽巢原理可知, 在 n 行 n 列的棋盘上至多放置 n 个皇后, 反之, n 个皇后在 n * n 棋盘上的可行棋局通常也存在

###### 皇后
皇后是组成棋局和最终解的基本单元, 这里将同行, 同列或同对角线的任意两个皇后视作 "相等", 于是两个皇后相互冲突当且仅当二者被判作 "相等"
```
struct Queen { // 皇后类
    int x, y; // 皇后在棋盘上的位置坐标
    Queen (int xx = 0, int yy = 0) : x (xx), y (yy) {};
    bool operator == (Queen const& q) const { // 重载判等操作符, 以检测不同皇后之间可能的冲突
        return (x == q.x) // 行冲突 (这一情况其实不会发生, 可省略)
                || (y == q.y) // 列冲突
                || (x + y == q.x + q,y) // 沿正对角线冲突
                || (x - y) == (q.x + q.y) // 沿反对角线冲突
    }
    bool operator != (Queen const& q) const { // 重载不等操作符
        return !(*this == q);
    }
}
```
###### 算法实现
基于试探回溯策略, 实现通用的 N 皇后问题; 既然每行能且仅能放置一个皇后, 故不妨首先将各皇后分配至每一行; 然后从空棋盘开始, 逐个尝试着将它们放置在无冲突的某列, 放置好一个皇后, 才继续试探下一个; 若当前皇后在任何列都会造成冲突, 则后续皇后的试探都必将是徒劳的, 故此时应该回溯到上一皇后
```
void placeQueens(int N) { // N 皇后算法 (迭代版): 采用试探/回溯的策略, 借助栈记录查找的结果
    Stack<Queen> solu; // 存放 (部分) 解的栈
    Queen q(0,0); // 从原点出发
    do { // 反复试探回溯
        if (N  <= solu.size() | N <= q.y) { // 若已出界
            q = solu.pop(); // 则回溯一行
            q.y++; // 并继续试探下一列
        } else { // 否则试探下一行
            while ((q.y < N) && (0 <= sulo.find(q))) { // 通过与已有皇后对比, 尝试找到可摆放下一皇后的列
                q.y++;
                nCheck++;
            }
            if (N > q.y) { // 若存在可摆放下一皇后的列
                sulo.push(q); // 摆上皇后
                if (N <= sulo.size()) nSulo++; // 若部分解已成为全局解, 则通过全局变量 nSulo 计数
                q.x++;
                q.y = 0; // 转入下一行, 从 0 列开始试探下一皇后
            }
        }
    } while((0 < q.x) || q.y < N); // 所有分支均已穷举或者剪枝后, 算法结束
}
```
###### 实例
TODO

##### 迷宫寻径
###### 问题描述
路径规划要求依照约定的行进规则, 在具有特定几何结构的空间区域内, 找到从起点到终点的一条通路; 以下问题是一个简化版本: 在空间区域内限定为由 n * n 个方格组成的迷宫, 除了四周的围墙, 还有分布其间的若干障碍物: 只能水平或垂直移动, 最终在任意指定的起始格点与目标格点之间找出一条通路 (如果确实存在)
###### 迷宫格点
格点是迷宫的基本组成单位, 除了记录器位置坐标以外, 格点还需记录其所处的状态, 共有四种可能的状态: 原始可用 (AVAILABLE), 在当前路径上的 (ROUTE), 所有方向均尝试失败后回溯过的 (BACKTRACKED), 不可穿越的 (WALL); 属于当前路径上的格点, 还需要记录其前驱和后继格点的方向; 既然只有上下左右四个联通方向, 故以 EAST, SOUTH, WEST, NORTH 区分, 特别的, 因尚未搜索到而处于 AVAILABLE 状态的格点, 邻格的方向都是未知的 (UNKNOWN), 经过回溯后处于 BACKTRACKED 状态的格点, 与邻格之间的联通关系均已关闭, 故记作 NO_WAY
```
// 原始可用, 在当前路径上的, 所有方向均尝试失败后回溯过的, 不可穿越的
typedef enum {AVAILABLE, ROUTE, BACKTRACKED, WALL} Status; // 迷宫单元的状态

// 未定, 东, 南, 西, 北, 无路可通
typedef enum {UNKNOWN, EAST, SOUTH,EAST, NORTH, NO_WAY} ESWN; // 单元的相对邻接方向

inline ESWN nextESWN (ESWN eswn) {
    return ESWC(eswn + 1); // 依次转至下一邻接方向
};

struct Cell { // 迷宫格点
    int x, y;
    Status status; // 状态
    ESWN incoming, outgoing; // 进入, 走出方向
};

#define LABY_MAX 24 // 最大迷宫尺寸
Cell laby[LABY_MAX][LABY_MAX]; // 迷宫
```

###### 邻格查询
在路径试探过程中需反复确定当前位置的相邻格点
```
inline Cell* neighbor(Cell* cell) { // 查询当前位置的相邻格点
    switch (cell->outgoing) {
        case EAST: return cell + LABY_MAX; // 向东
        case SOUTH: return cell + 1; // 向南
        case WEST: return cell + LABY_MAX; // 向西
        case NORTH: return cell + 1; // 向北
        default: exit(-1);
    }
}
```

###### 邻格转入
在确认某一相邻格点可用之后, 算法将朝对应的方向向前试探一步, 同时路径延长一个单元
```
inline Cell* advance(Cell* cell) { // 从当前位置转入相邻格点
    Cell* next;
    swith(cell->outgoing) {
        case EAST: next = cell + LABY_MAX; next->incoming = WEST; break; // 向东
        case SOUTH: next = cell + 1; next->incoming = NORTH; break; // 向南
        case WEST: next = cell - LABY_MAX; next->incoming = EAST; break; // 向西
        case NORTH: next = cell - 1; next->incoming = SOUTH; break; // 向北
        default: exit(-1);
    }
    return next;
}
```

###### 算法实现
基于试探回溯策略实现寻径算法
```
// 迷宫寻径算法: 在格单元 s 到 t 直接规划一条道路 (如果的确存在)
bool labyrinth(Cell Laby[LABY_MAX][LABY_MAX], Cell* s, Cell* t) {
    if ((AVAILABLE != s->status) || (AVAILABLE != t->status)) return false; // 退化情况
    Stack<Cell*> path; // 用栈记录通路 (Theseus 的线绳)
    s->incoming = UNKNOWN;
    s->status = ROUTE;
    path.push(s); // 起点
    do { // 从起点出发不断试探, 回溯, 直到抵达终点, 或者穷尽所有可能
        Cell* c = path.top(); // 检查当前位置 (栈顶)
        if (c == t) return true; // 若已抵达终点, 则找到了一条路; 否则沿尚未试探的方向继续试探
        while (NO_WAY > (c->outgoing = nextESWN(c->outgoing))) // 逐一检查所有方向
            if (AVAILABLE == neighbor(c)->status) break; // 试图找到尚未试探的方向
        if (NO_WAY <= c->outgoing) { // 若所有方向都已试探过
            c->status = BACKTRACKED;
            c = path.pop(); // 则后退回溯一步
        } else { // 否则向前试探一步
            path.push(c = advance(c));
            c->outgoing = UNKNOWN;
            c->status = ROUTE;
        }
    } while (!path.empth());
    return false;
}
```
这问题的搜索中, 局部解是一条源自于格起点的路径, 随着试探回溯相应的伸长缩短; 因此在这里借助 path 按次序记录组成当前路径的所有格点, 并动态的随着试探回溯做出栈入栈操作; 路径的其实格点, 当前的末端格点分别对应于 path 的栈底和栈顶, 当后者抵达目标格点时搜索成功, 此时 path 所对应的路径应作为全局返回
###### 实例
TODO
###### 正确性
TODO
###### 复杂度
TODO

#### 队列
##### 概述
###### 入队和出队
与栈一样, 队列 (queue) 也是存放数据对象的一种容器, 其中的数据对象也按线性的逻辑次序排列, 队列结构同样支持对象的插入和删除, 但是两种操作的范围分别限制于队列的两端, 若约定新对象只能从某一端插入其中, 从另一端删除已有的元素; 允许取出元素的一端称作对头 (front), 而允许插入元素的另一端称作为队尾 (rear)
###### 先进先出
由以上约定则知与栈结构恰恰相反, 队列中各对象的操作次序遵循所谓的先进先出 (first-in-first-out, FIFO) 的规律
###### ADT 接口

| 操作 | 功能 |
| :--- | :--- |
| size() | 报告队列的规模 (元素总数) |
| empty() | 判断队列是否为空 |
| enqueue(e) | 将 e 插入队尾 |
| dequeue(e) | 删除对首对象 |
| front() | 引用对首对象 |
###### 操作实例
TODO

##### Queue 模板类
作为列表的派生类来实现队列的数据结构
```
#include "../List/List.h" // 以 List 作为基类
template <typename T> class Queue: public List<T> { // 队列模板类 (继承 List 原有接口)
public: // size(), empty() 以及其它开放接口均可直接沿用
    void enqueue(T const& e) {
        insertAsLast(e); // 入队: 尾部插入
    }
    T dequeue() {
        return remove(first()); // 出队: 首部删除
    }
    T& front() {
        return first()->data; // 队首
    }
}
```

#### 队列应用
##### 循环分配器
在为客户群体中共享某一资源 (例如多个应用程序共享同一 CPU), 一套公平且高效的分配规则必不可少, 而队列结构则非常适用于定义和实现这样一套分配规则
```
RoundRobin { // 循环分配器
    Queue Q(clients); // 参与资源分配的所有客户组成队列 Q
    while (!ServiceClosed()) { // 在服务关闭之前
        e = Q.enqueue(); // 队首客户出队
        serve(e); // 接受服务
        Q.dequeue(e); // 重新入队
    }
}
```

##### 银行服务模拟
使用队列结构实现顾客服务的调度和优化
```
struct Customer { // 顾客类
    int window; // 所属窗口
    unsigned int time; // 服务时长
}

void simulate (int nWin, int servTime) { // 按指定窗口数, 服务总时间模拟银行业务
    Queue<Customer>* windows = new Queue<Customer>[nWin]; // 为每一个窗口创建一个队列
    for (int now = 0; now < servTime; now++) { // 在接受服务前
        if (rand() % (1 + nWin)) { // 新顾客以 win/(nWin + 1) 的概率到达
            Customer c;
            c.time = 1 + rand() % 98; // 新顾客到达, 服务是时长随机确定
            c.window = bestWindow(windows, nWin); // 找出最佳 (最短) 的服务窗口
            windows[c.window].enqueue(c); // 新顾客加入队列
        }
        for (int i = 0; i < nWin; i++) { // 分别检查
            if (!windows[i].empty()) { // 各非空队列
                if (--windows[i].front.time == 0) { // 队首顾客的服务时长减少一个单位
                    windows[i].dequeue();  // 服务完毕的顾客出列
                }
            }
        }
    }
    delete [] windows; // 释放所有队列
}

int bestWindow(Queue<Customer> windows[], int nWin) { // 为新到顾客寻找最佳队列
    int minSize = windows[0].size(), optWin = 0; // 最优队列
    for (int i = 1; i < nWinl i++) { // 在所有窗口中
        if (minSize > windows[i].size())  {
            minSize = windows[i].size();
            optWin = i; // 挑选出最优队列
        }
    }
    return optWin; // 返回
}
```
