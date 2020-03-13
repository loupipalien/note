### 虚拟机字节码执行引擎
代码编译的结果从本地机器码转变为字节码, 是存储格式发展的一小步, 却是编程语言发展的一大步

#### 概述
在 Class 文件格式与执行引擎这部分中, 用户程序能直接影响的内容并不太多, Class 文件以何种格式存储, 类型何时加载, 如何连接, 以及虚拟机如何执行字节码指令等都是由虚拟机直接控制的行为, 用户程序无法对其进行改变; 能通过程序进行操作的, 主要是字节码生成和类加载器这两部分的功能

#### 案例分析
以下四个案例中, 类加载器和字节码的案例各有两个

##### Tomcat: 正统的类加载器架构
主流的 Java Web 服务器, 如 Tomcat , Jetty, WebLogic, WebSphere 等, 都实现了自己定义的类加载器 (一般不止一个); 因为一个功能健全的 Web 服务器要解决以下几个问题
- 部署在同一个服务器上的两个 Web 应用程序所使用的 Java 类库可以实现相互隔离; 两个不同的应用程序可能会依赖同一个第三方类库的不同版本, 不能要求一个类库在一个服务器中只有一份, 服务器应当保证两个应用程序的类库可以相互独立使用
- 部署在同一个服务器上的两个 Web 应用程序所使用的 Java 类库可以互相共享; 如果有多个应用部署时, 如果类库不能共享, 虚拟机的方法区就会很容易出现过度膨胀的风险
- 服务器需要尽可能的保证自身的安全不受部署的 Web 应用程序影响; 目前有许多主流服务器也是由 Java 语言来实现的; 因此服务器本身也有类库依赖的问题, 基于安全考虑, 服务器所使用的类库应该与应用程序的类库互相独立
- 支持 JSP 应用的 Web 服务器, 大多数都需要支持 HotSwap 功能; 但 JSP 文件最终要编译成 Class 文件才可以由虚拟机执行, 但 JSP 文件由于其纯文本的特性, 运行时修改的概率远远大于第三方类库或者程序本身

由于上述问题, 在部署 Web 应用时, 单独的一个 ClassPath 就无法满足需求了, 所以各种 Web 服务器都不约而同的提供了好几个 ClassPath 路径供用户存放第三方类库, 这些路径一般都以 `lib` 或 `classes` 命名; 被放置到不同的路径中的类库, 具备不同的访问范围和服务对象, 通常每一个目录都会有相应的自定义类加载器去加载放置在里面的 Java 类库  
在 Tomcat (5.x) 目录结构中有 3 组目录 ("/common", "/server", "/shard") 可以存放 Java 类库, 另外还可以加上 Web 应用程序自身的目录 "/WEB-INF", 一共 4 组; 把 Java 类库放置在这些目录中的含义分别如下
- 放置在 /common 目录中, 类库可被 Tomcat 和所有的 Web 应用程序共同使用
- 放置在 /server 目录中, 类库可被 Tomcat 使用, 对所有的 Web 应用程序都不可见
- 放置在 /shared 目录中, 类库可被所有的 Web 应用程序使用, 但对 Tomcat 自己不可见
- 放置在 /WebApp/WEB-INF 目录中, 类库可被所有的 Web 应用程序共同使用, 但对 Tomcat 自己不可见

为了支持这套目录结构, 并对目录里面的类库进行加载和隔离, Tomcat 自定义实现了多个类加载器, 这些类加载器按照经典的双亲委派模型来实现
```
                                                                Catalina 类加载器
                                                               /
启动类加载器 <- 扩展类加载器 <- 应用程序类加载器 <- Common 类加载器
                                                               \
                                                                Shared 类加载器 <- WebApp 类加载器 <- Jsp 类加载器
```
CommmonClassLoader, CatalinaClassLoader, SharedClassLoader 和 WebAppClassLoader 是 Tomcat 自定义的类加载器, 它们分别加载 /common, /server, /shared 和 /WebApp/WEB-INF 中的 Java 类库; 其中 WebApp 类加载和 Jsp 类加载器通常会存在多个实例, 每一个 Web 应用程序对应一个 Jsp 类加载器; CommmonClassLoader 能加载的类都可以被 CatalinaClassLoader 和 SharedClassLoader 使用, 而 CatalinaClassLoader 和 SharedClassLoader 自己能加载的类与对方相互隔离; WebAppClassLoader 可以使用 SharedClassLoader 加载到的类, 但各个 WebAppClassLoader 实例之间相互隔离; 而 JspClassLoader 的加载范围仅仅是这个 JSP 文件编译出的 Class, 它出现的目的就是为了被丢弃: 当服务器检测到 JSP 文件被修改时, 会替换掉目前的 JspClassLoader 的实例, 并通过再建立一个新的 Jsp 类加载器来实现 JSP 文件的 HotSwap 功能

##### OSGi: 灵活的类加载架构
OSGi (Open Service Gateway Initiative) 是 OSGi 联盟执行的一个基于 Java 语言的动态模块化规范; OSGi 中的每个模块 (称为 Bundle) 与普通的 Java 类库区别不大, 两者一般都以 JAR 格式进行封装, 并且内部储存的都是 Java Package 和 Class; 但是一个 Bundle 可以声明它所依赖的 Java Package (通过 Import-Pachage 描述), 也可以声明它所允许导出发布的 Java Package (通过 Export-Package 描述); 在 OSGi 中, Bundle 之间的依赖关系从传统的上层模块依赖底层模块转变为平级模块之间的依赖 (至少外观上如此), 而且类库的可见性能得到非常精确的控制, 一个模块里只有被 Export 过的 Package 才可能由外界访问, 其他的 Package 和 Class 将会隐藏起来; 除了更精确的模块划分和可见性控制外, 引入 OSGi 的另外一个重要的理由是, 基于 OSGi 的程序很可能可以实现模块级的热插拔功能, 当程序升级更新或调试除错时, 可以停用或重新安装后启用程序中的其中一部分, 这对企业级程序开发来说是一个非常诱惑力的特性  
OSGi 的 Bundle 类加载器之间只有规则没有固定的委派关系; 例如: 某个 Bundle 声明了一个它依赖的 Package, 如果有其他 Bundle 声明发布了这个 Package, 那么所有对这个 Package 的类加载动作都会委派给发布它的 Bundle 类加载器去完成; 不涉及某个具体的 Package 时, 各个Bundle 加载器都是平级关系, 只有具体使用某个 Package 和 Class 时, 才会根据 Package 导入导出定义构造 Bundle 间的委派和依赖; 另外, 一个 Bundle 类加载器为其他 Bundle 提供服务时, 会根据 Export-Package 列表严格控制访问范围, 如果一个类存在于 Bundle 的类库中但是没有被 Export, 那么这个 Bundle 的类加载器能找到这个类, 但不会提供给其他 Bundle 使用, 而且 OSGi 平台也不会把其他 Bundle 的类加载请求分配给这个 Bundle 来处理  
例子: Bundle A 声明发布了 Package A, 依赖了 java.* 的包; Bundle B 声明依赖了 Package A 和 Package C 的包, 同时也依赖了 java.* 的包; Bundle C 声明发布了 Package C, 依赖了 Package A; 依赖图如下所示
```
      ------------------------------------ ----------------    
    /                                    /                 \
父类加载器 <- Bundle A 类加载器 <- Bundle C 类加载器 <- Bundle B 类加载器
                     \                  /
                       ----------------    
```
OSGi 的类加载时可能进行的查找如下
- 以 java.* 开头的类, 委派给父类类加载器
- 否则, 委派列表名单内的类委派给父类加载器加载
- 否则, Import 列表中的类, 委派给 Export 这个类的 Bundle 的类加载器加载
- 否则, 查找当前 Bundle 的 ClassPath, 使用自己的类加载器加载
- 否则, 查找是否在自己的 Frafment Bundle 中, 如果是, 则委派给 Frafment Bundle 的类加载器加载
- 否则, 查找 Dynamic Import 列表的 Bundle, 委派给对应的 Bundle 的类加载器加载
- 否则, 类查找失败

##### 字节码生成技术和动态代理实现
在 Java 里除了 javac 和字节码类库外, Web 服务器的 JSP 编译器, 编译时植入的 AOP 框架, 还有很常用的动态代理技术, 甚至在反射的时候虚拟机都有可能会在运行时生成字节码来提高执行速度; 在 Spring 中, 大多数情况都会用过动态代理, 因为如果 Bean 是面向接口编程的, 那么 Spring 内部都是通过动态代理的方式来实现对 Bean 的增强的; 动态代理中所谓的 "动态" 是针对使用 Java 代码实际编写了代理类的 "静态" 代理而言的, 它的优势不在于省去了编写代理类那一些工作, 而是实现了可以在原始类和接口还未知的时候, 就确定了代理类的代理行为, 当代理类与原始类脱离直接联系之后, 就可以很灵活的重用在不同的应用场景中
```
public class DynamicProcyTest {

    interface IHello {
        void sayHello();
    }

    static class Hello implements IHello {

        @Override
        public void sayHello() {
            System.out.println("Hello World!");
        }
    }

    static class DynamicProxy implements InvocationHandler {

        Object originalObj;

        Object bind(Object originalObj) {
            this.originalObj = originalObj;
            return Proxy.newProxyInstance(originalObj.getClass().getClassLoader(), originalObj.getClass().getInterfaces(), this);
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            System.out.println("Welcome!");
            return method.invoke(originalObj, args);
        }
    }

    public static void main(String[] args) {
        IHello hello = (IHello) new DynamicProxy().bind(new Hello());
        hello.sayHello();
    }
}
```
Proxy.newProxyInstance() 方法返回实现了一个 IHello 的接口, 并且代理了 new Hello() 实例行为的对象, 进行了验证, 优化, 缓存, 同步, 生成字节码, 显示类加载等操作, 最后它调用了 sun.misc.ProxyGenerator,generateProxyClass() 方法来完成生成字节码的动作, 这个方法可以在运行时产生一个描述代理类的字节码 byte[] 数组

##### Retrotranslator: 跨越 JDK 版本
Retrotranslator 是一种 Java 逆向移植工具, 它的作用是将 JDK 1.5 编译出来的 Class 文件转变为可以在 JDK 1.4 或 JDK 1.3 上部署的版本, 它可以很好的支持自动装箱, 泛型, 动态注解, 枚举, 变长参数, 遍历循环, 静态导入等语法特性, 甚至额可以支持 JDK 1.5 中新增的集合改进, 并发包以及对泛型, 注解等的反射操作等  
要想知道 Retrotranslator 如何在旧版本 JDK 中模拟新版本的 JDK 的功能, 首先要弄清楚 JDK 升级中会提供哪些新的功能, JDK 每次升级新增的功能大致可以分为以下 4 类:
- 在编译器层面做的改进, 如自动装箱拆箱, 实际上就是编译器在程序中使用到包装对象的地方自动插入了很多 Integer.valueOf(), Float.valueOf() 之类的代码; 变长参数在编译之后就自动转化为了一个数组来完成参数传递; 泛型的信息则在编译阶段就已经擦除掉了 (但是在元数据中还保留着), 相应的地方被编译器自动插入了类型转换代码
- 对 Java API 的代码增强; 譬如 JDK 1.2 时代引入的 java.util.Collections 等一系列集合类, 在 JDK 1.5 时代引入的 java.util.concurrent 并发包等
- 需要在字节码进行支持的改动; 如在 JDK 1.7 里新加入的语法特性: 动态语言支持, 就需要在虚拟机中新增一条 invokedynamic 字节码指令来实现相关的调用功能; 不过字节码指令集一直处于相对比较稳定的状态, 这种需要在字节码层面直接进行改动的是比较少见的
- 虚拟机内部的改进; 如 JDK 1.5 中实现的 JSR-133 规范重新定义的 Java 内存模型 (Java Memory Model, JMM), CMS 收集器之类的改动, 这类改动对于程序员编写代码基本是透明的, 但会对程序运行时产生影响

上述 4 类新功能中, Retrotranslator 只能模拟前两类, 对于后面两类在虚拟机内部实现的改进, 一般所有的逆向移植工具都是无能为力的, 至少不能完整的或者在可接受的效率上完全全部模拟; 在可模拟的两类功能中, 第二类模拟相对容易一些, 如在 JDK 1.5 引入的 java.util.concurrent 包, 已有对应包来替换; 至于 JDK 在编译阶段进行处理的哪些改进, Retrotranslator 则是使用 ASM 框架直接对字节码进行处理, 由于组成 Class 文件的字节码指令数量并没有改变, 所以无论是 JDK 1.3 JDK 1.4 和 JDK 1.5, 能用字节码表达的语义范围应该是一致的

#### 实战: 自己动手实现远程执行功能
TODO...
