### 构建 Spring Web 应用程序
Spring MVC 基于模型-视图-控制器 (Model-View-Controller, MVC) 模式实现, 能够实现构建像 S rping 框架那样灵活的松耦合的 Web 应用程序  

#### Spring MVC 起步
Spring 将请求在调度 Servlet, 处理器映射 (handler mapping), 控制器以及视图解析器 (view resolver) 之间移动

##### 跟踪 Spring MVC 的请求
![SpringMVC的请求.png](http://ww1.sinaimg.cn/large/d8f31fa4gy1gd9qkjueizj20hj0azwf1.jpg)  
- 在请求离开浏览器时, 会带有用户所请求内容的信息, 至少会包含请求的 URL
- 请求首先到达的是 Spring 的 DispatcherServlet, 这是一个单例的前端控制器 (front controller) Servlet, 这个 Servlet 会将请求委托给应用程序的其他组件来执行实际的处理
- DispatcherServlet 的任务是将请求发送给 Spring MVC 控制器 (controller), 控制器是一个用于处理请求的 Spring 组件; DispatcherServlet 需要知道应该将请求发送到哪个控制器, 所以 DispatcherServlet 会查询一个或多个处理器映射 (handler mapping) 来确定下一站是哪里; 处理器映射会根据请求所携带的 URL 信息来进行决策
- 一旦选择了合适的控制器, DispatcherServlet 会将请求发送给选中的控制器, 请求会卸下其负载 (用户提交的信息) 并耐心等待控制器处理这些信息 (实际上, 设计良好的控制器只处理很少或不处理工作, 而是将业务逻辑委托给一个或多个服务对象进行处理)
- 控制器在完成逻辑处理后, 通常会产生一个信息, 这些信息需要返回给用户并在浏览器上显示; 这些信息称为模型 (Model); 但仅仅返回信息是不够的, 这些信息需要以友好的方式进行格式化, 一般会是 HTML; 所以信息需要发送给一个视图 (View), 通常会是 JSP; 控制器所作的最后一件事就是将模型数据打包, 并且标示出用于渲染输出的视图名, 接下来会将请求连同模型和视图名发送给 DispatcherServlet
- 控制器不会与特定的视图耦合, 传递给 DispatcherServlet 的视图名并不直接表示某个 JSP, 甚至不能确定视图就是 JSP; 视图名仅仅是一个逻辑名称, 这个名字将会用来查找产生结果的真正视图, DispatcherServlet 将会使用视图解析器来将逻辑视图名匹配一个特定的视图实现, 它可能是 JSP 也可能不是
- DispatcherServlet 知道了由哪个视图渲染结果, 那么请求的任务基本上也就完成了, 它的最后一站是视图的实现 (可能是 JSP), 在这里它交付模型数据, 请求的任务就完成了; 视图将使用模型数据渲染输出, 这个输出会通过响应对象传递给客户端

##### 搭建 Sprng MVC
扩展 `AbstractAnnotationConfigDispatcherServletInitializer` 的任意类都会自动的配置 `DispatcherServlet` 和 Spring 应用上下文, Spring 应用上下文会位于应用程序的 Servlet 上下文中  
> 在 Servlet 3.0 的容器环境中, 容器会在类路径中查找实现了 `javax.servlet.ServletContainerInitializer` 接口的类, 如果能发现的话就用用它来配置 Servlet 容器
> Spring 提供了这个容器的实现 `SpringServletContainerInitializer`, 这个类反过来又会查找实现 `WebApplicationInitializer` 的类并将配置交给它们来完成; Spring 3.2 引入了一个便利的 `WebApplicationInitializer` 的基础实现, 也就是 `AbstractAnnotationConfigDispatcherServletInitializer`, 所以当你的应用继承了这个类并部署到 Servlet 3.0 容器中时, 容器就会自动发现它, 并使用它来配置 Servlet 上下文
