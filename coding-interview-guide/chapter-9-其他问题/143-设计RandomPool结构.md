### 设计 RandomPool 结构

#### 题目
设计一种结构, 在该结构中有如下三个功能
- insert(key): 将某个 key 加入到该结构中, 做到不重复加入
- delete(key): 将原本在结构中的某个 key 移除
- getRandom(): 等概率随机返回结构中的任何一个 key

#### 要求
insert, delete, getRandom 的方法时间复杂度都为 O(1)

#### 难度
:star::star:

#### 思路
可用 HashMap 实现

#### 实现
```Java
public class RandomPool<K> {
    private Map<K,Integer> keyIndexMap = new HashMap<>();
    private Map<Integer,K> indexKeyMap = new HashMap<>();

    public void insert(K key) {
        if (!keyIndexMap.containsKey(key)) {
            keyIndexMap.put(key, keyIndexMap.size());
            indexKeyMap.put(keyIndexMap.size() - 1, key);
        }
    }

    public void delete(K key) {
        if (keyIndexMap.containsKey(key)) {
            indexKeyMap.remove(keyIndexMap.remove(key));
        }
    }

    public K getRandom() {
        int index = (int) (Math.random() * (keyIndexMap.size() - 1));
        return indexKeyMap.get(index);
    }
}
```
