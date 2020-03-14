# Spring MVC
[toc]

Spring MVC是Spring框架提供的Web组件，它的实现基于MVC的设计模式。

# 执行流程
1. 客户端发送请求至前端控制器(DispatcherServlet)。
2. 前端控制器根据请求路径，进入对应的处理器。
3. 处理器调用相应的业务方法。
4. 处理器获取到相应的业务数据。
5. 处理器把组装好的数据还给前端控制器。
6. 前端控制器将获取的ModelAndView对象传给视图解析器(ViewResolver)。
7. 前端控制器获取到解析好的页面数据。
8. 前端控制器将解析好的页面返回给客户端。

# 核心组件
1. DispatcherServlet：核心处理器(前端控制器)，负责调度其他组件的执行。可降低不同组件之间的耦合性，是整个Spring MVC的核心模块。
2. Handler：处理器，完成具体业务逻辑，相当于Servlet或Action。
3. HandlerMapping：DispatcherServlet是通过HandlerMapping将请求映射到不同Handler。
4. HandlerInterceptor：处理器拦截器，是一个接口，如果需要做一些拦截处理，可以实现这个接口。
5. HandlerExecutionChain：处理器执行链，包括Handler和HandlerInterceptor。
6. HandlerAdapter：处理器适配器，Handler执行业务方法之前，需要进行一系列的操作包括表单数据的验证、数据类型的转换、将表单数据封装到POJO等，这一系列的操作都是由HandlerAdapter来完成，DispatcherServlet通过HandlerAdapter执行不同的Handler。
7. ModelAndView：装载了模型数据和视图信息，作为Handler的处理结果，返回给DispatcherServlet。
8. ViewResolver：视图解析器，DispatcherServlet通过它将逻辑视图解析成物理视图，最终将渲染结果响应给客户端。

# forward和redirect的区别
1. forward表示请求转发，请求转发是服务器的行为；redirect表示重定向，重定向是客户端行为。
2. forward是服务器请求资源，服务器直接访问把请求的资源转发给浏览器，浏览器根本不知道服务器的内容是从哪来的，因为它的地址栏还是原来的地址；redirect是服务器发送一个状态码告诉浏览器重新请求新的地址，因此地址栏显示的是新的URL。
3. forward转发页面和转发到的页面可以共享request里面的数据；redirect不能共享数据。
4. 从效率来说，forward比redirect效率更高。

# 拦截器的使用场景
1. 日志记录。
2. 权限检查。
3. 统一安全处理。