## 串
### 目录

### 快速排序 (Quicksort)
将序列分为两个子序列: $S = S_L + S_R$  
规模缩小: $max \lbrace |S_L|, |S_R| \rbrace < n$  
彼此独立: $max(S_L) \leq min(S_R)$  
在子序列分别递归排序之后, 原序列自然有序: $sorted(S) = sorted(S_L) + sorted(S_R)$  
平凡解: 只剩单个元素时本身就是解
mergesort 的计算量和难点在于合, 而 quicksort 在于分  
- 轴点 (pivot)
左 / 右侧的元素, 均不比它更大 / 小  
坏消息: 在原始序列中, 轴点未必存在  
必要条件: 如果是序列轴点, 则必然已经就位, 反之不然  
特别的: 在有序序列中, 所有元素皆为轴点, 反之亦然  
快速排序: 就是将所有元素逐个转换为轴点的过程  
好消息: 通过适当的交换, 可使任一元素转换为轴点  

```
template <typename T> void Vector <T>::quickSort(Rank lo, Rank hi) {
  if (hi - lo < 2) return; // 单元素区间自然有序,否则
  Rank mi = partition(lo, hi - 1); // 先构造轴点
  quickSork(lo, mi); // 前缀排序
  quickSork(mi + 1, hi); // 后缀排序
}
```
- 性能分析
unstable: lo/hi 的移动方向相反, 左/右侧的大/小重复元素可能前/后颠倒
in-place: 只需 O(1) 附加空间
最好情况: 每次划分都 (接近) 平均, 轴点总是 (接近) 中央
$$ T(n) = 2 * T((n - 1) / 2) + O(n) = O(n\log n) $$
最坏情况: 每次划分都极不均衡
$$ T(n) = T(n - 1) + T(0) + O(n) = O(n^2) $$
即便采用随机选取, (Unix) 三者取中之类的策略也只能降低最坏情况的概率, 而无法杜绝  
