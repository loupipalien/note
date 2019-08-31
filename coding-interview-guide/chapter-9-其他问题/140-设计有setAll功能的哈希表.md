### 设计有 setAll 功能的哈希表

#### 题目
哈希表常见的三个操作是 put, get, containsKey, 而且这三个操作的时间复杂度为 O(1); 现在想加一个 setAll 的功能, 就是把所有记录的 value 都设计为统一的值; 请设计并实现这种有 setAll 功能的哈希表, 并且 put, get, containsKey 和 setAll 四个操作的时间复杂度都是 O(1)

#### 难度
:star:

#### 思路
存放时为每个记录加一个时间戳, setAll 时设置一个全局的值和一个时间戳, 获取时比对时间戳进行返回

#### 实现
```Java
public class MyHashMap<K, V> {
    private HashMap<K, AbstractMap.SimpleEntry<V,Long>> baseMap = new HashMap<>();
    private AbstractMap.SimpleEntry<V,Long> baseEntry = new AbstractMap.SimpleEntry<>(null, Long.MIN_VALUE);

    public void put(K key, V value) {
        baseMap.put(key, new AbstractMap.SimpleEntry<>(value, System.currentTimeMillis()));
    }

    public V get(K key) {
        AbstractMap.SimpleEntry<V,Long> entry = baseMap.get(key);
        return entry.getValue() > baseEntry.getValue() ? entry.getKey() : baseEntry.getKey();
    }

    public boolean constainsKey(K key) {
        return baseMap.containsKey(key);
    }

    public void setAll(V value) {
        baseEntry = new AbstractMap.SimpleEntry<>(value, System.currentTimeMillis());
    }
}
```
