### 组合模式 (Composite Pattern)
将对象组合成树形结构以表示 "部分-整体" 的层次结构, 组合模式使得客户端对单个对象和组合对象的使用具有一致性; 组合模式属于结构型模式

#### 模式角色
- Component: 抽象构件类
- Leaf: 树叶构件类
- Composite: 树枝构件类

#### 代码示例
```
// 抽象构建类
public abstract class Component {

    protected String name;

    public abstract void add(Component component);

    public abstract void remove(Component component);

    public abstract void show();

}

// 树叶构建类
public class Composite extends Component {

    private List<Component> components = new ArrayList<>();

    public Composite(String name) {
        this.name = name;
    }

    @Override
    public void add(Component component) {
        component.add(component);
    }

    @Override
    public void remove(Component component) {
        components.remove(component);
    }

    @Override
    public void show() {
        components.forEach(component -> System.out.println(component.name));
    }
}

// 树枝构件类
public class Leaf extends Component {

    public Leaf(String name) {
        this.name = name;
    }

    @Override
    public void add(Component component) {
        throw new UnsupportedOperationException("Unsupported operation.");
    }

    @Override
    public void remove(Component component) {
        throw new UnsupportedOperationException("Unsupported operation.");
    }

    @Override
    public void show() {
        System.out.println(name);
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 可以清楚的定义分层次的复杂对象, 容易增加新的构件 | 设计变得更加抽象, 业务复杂时实现困难 |
| 客户端可以使用一致的组合结构或单个对象 | - |

#### 适用场景
- 需要表示一个对象整体或者部分层次, 在具有整体和部分的层次结构中, 希望通过一种方式忽略整体与部分的差异, 一致的对待
- 让客户端能够忽略不同对象层次的变化, 可以针对抽象构件编程, 无须关心实现细节

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)   
[组合模式](https://blog.csdn.net/hguisu/article/details/7530783)
[组合模式](https://segmentfault.com/a/1190000011988172)
