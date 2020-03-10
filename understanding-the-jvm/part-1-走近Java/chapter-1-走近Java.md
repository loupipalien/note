### 走进 Java
世界上并没有完美的程序, 但我们并不因此而沮丧, 因为写程序本来就是一个不断追求完美的过程

#### 概述

#### Java 技术体系
- Java 程序设计语言
- 各种硬件平台上的 Java 虚拟机
- Class 文件格式
- Java API 类库
- 来自商业机构和开源社区的第三方 Java 类库

JDK: Java 程序设计语言, Java 虚拟机, Java API 类库的统称
JRE: Java SE API 子集和 Java 虚拟机的统称

#### Java 发展史
- 1996 年 1 月 23 日, JDK1.0 版本发布; 代表技术包括: Java 虚拟机, Applet, AWT 等
- 1997 年 2 月 19 日, JDK1.1 版本发布; 代表技术包括: JAR 文件格式, JDBC, JavaBeans, RMI; Java 语法中的内部类和反射也是这个时候出现的
- 1998 年 12 月 4 日, JDK1.2 版本发布, 将 Java 技术体系拆分为三个方向: J2SE, J2EE, J2ME; 代表技术包括: EJB, Java Plug-in, Java IDL, Swing等; 在这个版本中 Java 虚拟机第一次内置了 JIT(just in time) 编译器, JDK1.2 版本中存在过三个虚拟机, 分别是 Classic VM, HotSpot VM, Exact VM, 其中 Exact VM 只在 Solaris 平台出现过; Java 语法中的 strictfp 关键字和 Collections 集合类也在此版本添加
- 2000 年 5 月 8 日, JDK1.3 版本发布; HotSpot 虚拟机成为默认虚拟机; 代表技术包括: JDNI, RMI 等
, 改进了 Java2D API 和添加了 JavaSound 类库; JDK1.3 后 Sun 大约每隔两年发布一个 JDK 的主版本, 以动物命名, 期间发布的各个修正版本则以昆虫作为工程名称
- 2002 年 2 月 13 日, JDK1.4 版本发布, 这是 Java 真正走向成熟的一个版本; 代表技术包括: 正则表达式, 异常链, NIO, 日志类, XML 解析器和 XSLT 转换器等
- 2004 年 9 月 30 日, JDK1.5 版本发布; 此版本改进了 Java 的内存模型, 提供了 java.util.concurrent 并发包; 在语法上添加了 自动装箱拆箱, 泛型, 动态注解, 枚举, 可变长度参数, 遍历循环等语法特性
- 2006 年 12 月 11 日, JDK1.6 版本发布; J2SE, J2EE, J2ME 命名修改为 Java SE 6, Java EE 6, Java ME 6; 并提供动态语言支持, 提供编译 API 和 微型 HTTP 服务器 API 等; 同时虚拟机内部做了大量改进, 包括锁与同步, 垃圾收集, 类加载等
- 2009 年 2 月 19 日, JDK1.7 版本发布
- ...

#### Java 虚拟机发展史
- Sun Classic / Exact VM
- Sun HotSpot VM
- Sun Mobile-Embedded VM / Meta-Circular VM
- BEA JRockit / IBM J9 VM
- Azul VM / BEA Liquid VM
- Apache Harmony / Google Android Dalvik VM
- Microsoft JVM 及其他

#### 展望 Java 技术的未来
- 模块化
- 混合语言
- 多核并行
- 进一步丰富语法
- 64 位虚拟机

#### 实战: 自己编译 JDK

>**参考:**
[Java Platform Standard Edition 7 Documentation](http://download.oracle.com/javase/7/docs)
