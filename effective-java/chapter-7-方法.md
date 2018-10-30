### 方法

#### 第 38 条: 检查参数的有效性
绝大多数方法和构造器对于传递给它们的参数值都会有某些限制, 例如索引值必须是非负数, 对象引用不能为 null 等等; 这些应该在文档中清楚的指明, 并且在方法体的开头处检查参数, 以强制施加这些限制; 这是 "应该在发生错误之后尽快检测出错误" 这一普遍原则的一个具体情形; 如果不能做到这一点, 检测到错误的可能性就比较小了, 即使检测到了错误, 也较难确定错误的根源   
不要从本条目内容得出这样的结论: 对参数的任何限制都是好事; 相反, 在设计方法时, 应该使它们尽可能的通用, 并符合实际的需要; 假如方法对于它能接受的所有参数值能够完成合理的工作, 对参数的限制越少越好

#### 第 39 条: 必要时进行保护性拷贝
使 Java 使用起来如此舒适的一个因素在于它是一门安全的语言 (safe language), 并不像 C 或者 C++ 那样把内存当作一个巨大的数组来看待, 所以对于内存破坏的错误都可以免疫; 但即使在安全的语言中, 如果不采取一点措施, 还是无法与其他类隔离开来; 假设客户端尽其所能的来破坏, 或者对 API 的误用所导致的不可预期的情况, 都只好由类来处理; 编写一些面对客户的不良行为时仍能保持健壮的类, 这是非常值得投入的事情  
没有对象的帮助, 另一个类不可能修改对象的内部状态, 但是对象很容易在无意识的情况下提供这种帮助; 考虑以下的类, 声称可以表示一段不可变的时间周期
```
// Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;

    public Period(Date start, Date end) {

        this.start = start;
        this.end = end;
    }

    public Date start() {
        retuen start;
    }

    public Date end() {
       return end;
    }
    ...
}
```
乍一看这个类是不可变的, 然而因为 Date 类本身是可变的, 因此很容易违反这个条件; 为了保护 Period 实例的内部信息避免受到这种攻击, 首先对于构造器的每个可变参数进行保护性拷贝 (defensive copy) 是必要的, 并且使用备份对象作为 Period 实例的组件, 而不适用原始的对象
```
// Required constructor - makes defensive copies of parameters
public Period(Date start, Date end) {
    this.start = new Date(start.getTime());
    this.end = new Date(end.getTime);

    if (start.comparaTo(end) > 0) {
        throw new IllegalArgumentException(start + "after" + end);
    }
}
```
用了新的构造器后, 外部对于传递给构造器的 Date 对象的修改不会影响到 Period 实例; 注意, 保护性拷贝是在检查参数的有效性 (见第 38 条) 之前进行的, 并且有效性检查是针对拷贝州的对象, 而不是针对原始的对象; 这样看起来有点不太自然, 但却是有必要的; 这样做可以避免在 "危险阶段" 期间从另一个线程改变类的参数, 这里的危险阶段是指从检查参数开始直到拷贝参数之间的时间段 (在计算机安全社区中, 被称作 Time-Of-Check/Time-Of-Use 或者 TOCTOU 攻击)
