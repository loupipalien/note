### 代理模式 (Proxy Pattern)
为对象提供一种代理以控制对此对象的访问; 代理模属于结构型模式

#### 模式角色
- Subject: 抽象主题类
- RealSubject: 真实主题类
- Proxy: 代理类


#### 代码示例
```Java
// 抽象主题类
public abstract class Subject {
    public abstract void request();
}

// 真实主题类
public class RealSubject extends Subject {

    @Override
    public void request() {
        System.out.println("request.");
    }
}

// 代理类
public class Proxy extends Subject {

    private Subject subject = new RealSubject();

    @Override
    public void request() {
        subject.request();
    }
}
```

#### 优缺点

| 优点 | 缺点 |
| :--- | :--- |
| 代理模式能够协调调用者和被调用者, 在一定程度上降低了系统的耦合性 | 由于在客户端和真实主题之间增加了代理对象, 因此有些类型的代理模式可能会造成请求额处理速度变慢 |
| 保护代理以控制对真实对象的使用权限 | 实现代理模式需要额外的工作, 有些代理模式的实现非常复杂 |

#### 适用场景
- 远程代理: 为一个位于不同的地址空间的对象提供一个本地的代理对象, 这个不同的地址空间可以是同一台主机中, 也可以是在另一台主机中
- 虚拟代理: 如果需要创建一个资源消耗大的对象, 先创建一个消耗相对较小的对象来表示, 真实对象只在需要时才会被创建
- 安全代理: 控制对一个对象的访问, 可以给不同的用户提供不同级别的使用权限
- 智能引用代理: 当一个对象被引用时, 提供一些额外的操作

>**参考:**
[大话设计模式](https://book.douban.com/subject/2334288/)   
[代理模式](https://design-patterns.readthedocs.io/zh_CN/latest/structural_patterns/proxy.html)
