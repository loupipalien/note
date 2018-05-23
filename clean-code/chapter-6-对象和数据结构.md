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
