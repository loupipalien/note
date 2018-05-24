### 对象和数据结构

#### 对象抽象
以下两段代码都表示笛卡尔平面上的一个点, 其中之一暴露了其实现, 另外一个完全隐藏了实现
- 具象点
```
public class Point {
    public double x;
    public double y;
}
```
- 抽象点
```
public interface Point {
    double getX();
    double getY();
    void setCartesian(double x, double y);
    double getR();
    double getTheta();
    void setPolar(double r, double theta);
}
```

后一段代码好处在于, 读者不知道该实现是在矩形坐标系还是极坐标系中, 也可能两个都不是 ! 然而该接口还是明白无误的呈现了一种数据结构, 还有其方法固定了一套存取策略, 可以单独读取某个坐标, 但必须通过一次原子操作设定所有坐标

#### 数据, 对象的反对称性
以下两段代码展示了对象与结构之间的差异; 对象把数据隐藏于抽象之后, 曝露操作数据的函数; 数据结构曝露其数据, 没有提供有意义的函数
- 过程式形状代码
```
public class Square {
    public Point topLeft;
    public double side;
}

public class Rectangle {
    public Point topLeft;
    public double height;
    public double width;
}

public class Circle {
    public Point center;
    public double radius;
}

public class Geomnetry {
    public final double PI = 3.141592653589793;

    public double area(Object shape) throws NoSuchShapeException {
        if (shape  instanceof Square) {
            Square s = (Square) shape;
            return s.side * s.side;
        } else if (shape instanceof Rectangle) {
            Rectangle r = (Rectangle) shape;
            return r.height * r.width;
        } else if (shape instanceof Circle) {
            Circle c = (Circle) shape;
            return PI * radius * radius;
        }
        throws new NoSuchShapeException;
    }
}
```
这时过程式代码, 如果给 Geomnetry 类添加一个 primeter() 函数根本不会影响到那些形状类, 但另一方面, 如果添加一个新形状, 就得修改 Geomnetry 中所有函数来处理它; 这两种情况也是对立的
- 多态式形状
```
pubic class Square implements Shape {
    private Point topLeft;
    private double side;

    public double area() {
        return side * side;
    }
}

public class Rectangle implements Shape {
    private Point topLeft;
    private double height;
    private double width;

    public double area() {
      return height * width;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.141592653589793;

    public double area() {
        return PI * radius * radius;
    }
}
```
浏览上述代码可以看到这两种定义的本质, 它们是截然对立的; 这说明了对象与数据结构之间的二分原理: 过程式代码 (使用数据结构的代码) 便于在不改动既有数据结构的前提下添加新的函数; 面向对象代码便于在不改动既有函数的前提下添加新类; 反之也成立: 过程式代码难以添加新数据结构, 因为必须修改所有函数; 面向对象代码难以添加新函数, 因为必须修改所有类

#### 得墨忒耳律
著名的得墨忒耳律 (The Law of Demeter) 认为: 模块不应该了解它所操作对象的内部情形; 即意味着对象隐藏数据, 曝露操作; 这意味着对象不应通过存取器曝露其内部结构, 因为这样更像是曝露而非隐藏其内部结构; 更准确的说, 得墨忒耳认为类 C 的方法 f 只应该调用以下对象的方法
- C
- 由 f 创建的对象
- 作为参数传递给 f 的对象
- 由 C 的实体变量持有的对象

方法不应调用任何函数返回对象的方法, 例如以下代码级违反了得墨忒耳律
```
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

##### 火车失事
连续调用的代码常被称为火车失事, 因为其看起来就像是一列火车, 这种连续的调用被认为是肮脏的风格, 因该避免; 最好做如下切分 (ElasticSearch 反而是这种风格...)
```
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```
这些代码是否违反得墨忒耳律, 取决于 ctxt, Options 和 ScratchDir 是对象还是数据结构; 如果是对象, 则它们的内部结构应当不被曝露, 而有关内部细节的知识就明显违反了得墨忒耳律; 如果 ctxt, Options 和 ScratchDir 只是数据结构, 没有任何行为, 则它们自然会曝露其内部结构, 得墨忒耳律自然也就不适用了, 这里是因为属性访问器函数的使用吧问题搞复杂了

##### 混杂
这种混淆有时会不幸的导致混合结构, 一半是对象, 一般是数据结构; 这种结构拥有执行操作的函数, 也有公共变量或公共访问器及改值器; 无论出于怎样的初衷, 公共访问器及改值器都把私有化变量公开化, 诱导外部函数以过程式程序使用数据结构的方式使用这些变量; 此类混杂增加了新函数的难度, 也增加了添加新数据结构的难度, 应该避免这种结构

##### 隐藏结构
假设 ctxt 是拥有真实行为的对象, 由于对象应隐藏在其内部结构, 防止让外部用户看到不该看到的对象而违反了得墨忒耳律; 所以直接让 ctxt 直接提供操作从而达到隐藏结构的效果
```
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

#### 数据传送对象
最为精炼的数据结构是一个只有公共变量, 没有函数的类; 这种数据结构有时候被称为数据传送对象, 或 DTO (Data Transfer Objects), 即通常说的 JavaBean

#### 小结
对象曝露行为, 隐藏数据; 便于添加新对象类型而无需修改既有行为, 同时也难以在既有对象中添加新行为; 数据结构曝露数据, 没有明显的行为; 便于向既有数据结构添加新行为, 同时也难以向既有函数添加新数据结构
