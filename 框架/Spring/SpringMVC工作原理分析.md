# SpringMVC工作原理分析
部分素材来源于：[https://github.com/WeiKangJian/JavaGuide/blob/master/docs/system-design/framework/spring/SpringMVC-Principle.md](https://github.com/WeiKangJian/JavaGuide/blob/master/docs/system-design/framework/spring/SpringMVC-Principle.md)

在本科时候，最早开始学的是简单的jsp+servlet来进行网页开发，后来又使用了SSM等框架来完成，一直都有老师在强调MVC，也知道model,view,controler等三个部分，但对于mvc尤其是在Spring中具体的实现过程没有去深究，直到今天特意整理了一下这方面的知识体系：

## MVC和用户交互过程
网上找了一张图来描述MVC的过程交互：
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS8wY2UxNWVmMTNhZDY1YzI3MTM2MmVjNDhiMmFjNTczYTY4MTFjOThiLzY4NzQ3NDcwM2EyZjJmNmQ3OTJkNjI2YzZmNjcyZDc0NmYyZDc1NzM2NTJlNmY3MzczMmQ2MzZlMmQ2MjY1Njk2YTY5NmU2NzJlNjE2YzY5Nzk3NTZlNjM3MzJlNjM2ZjZkMmYzMTM4MmQzMTMwMmQzMTMxMmYzNjMwMzYzNzM5MzQzNDM0MmU2YTcwNjc?x-oss-process=image/format,png)
在不考虑具体框架实现下，用户的请求都是交给 controller层的，由这一层对请求进行解析，就是我们在springBoot里面常用的加cortroller注解的类，这些都是控制器，具体的下面讲，在获取到用户请求后（请求中往往带有get和post的数据），在controller层调用server层的服务，再完成服务后，会进行页面的渲染（通俗的说就是执行你嵌在html中的代码类似jsp,或者执行freemaker等模板引擎，生成所需要的动态页面），讲渲染后的页面传给用户，这就是MVC的第一层。
在具体的server层会调用dao层（即和数据库连接的那一层），其中dao返回的数据是和model一一对应的，即一般从数据库的操作返回的都是一个对象或者某个属性值（springboot的约定大于配置）。

## SpringMVC工作原理和适配器模式
在以前的SSM等框架里面，我们往往要配置一个web.xml的配置文件，里面有下面的一些配置：

```
<listener>
	<listener-class>org.springframework.web.context.ContextLoaderListener
	</listener-class>
</listener>
<servlet>
	<servlet-name>springmvc</servlet-name>
	<servlet-class>org.springframework.web.servlet.DispatcherServlet
	</servlet-class>
	<!-- 如果不设置init-param标签，则必须在/WEB-INF/下创建xxx-servlet.xml文件，其中xxx是servlet-name中配置的名称。 -->
	<init-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:spring/springmvc-servlet.xml</param-value>
	</init-param>
	<load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
	<servlet-name>springmvc</servlet-name>
	<url-pattern>/</url-pattern>
</servlet-mapping>
```
需要显式的配置一个监听器。所有的请求都会传给DispatcherServlet，由DispatcherServlet来分发请求。具体的请求过程如下：
客户端发送请求-> 前端控制器 DispatcherServlet 接受客户端请求 -> 找到处理器映射 HandlerMapping 解析请求对应的 Handler-> HandlerAdapter 会根据 Handler 来调用真正的处理器开处理请求，并处理相应的业务逻辑 -> 处理器返回一个模型视图 ModelAndView -> 视图解析器进行解析 -> 返回一个视图对象->前端控制器 DispatcherServlet 渲染数据（Moder）->将得到视图对象返回给用户：
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS82ODg5ZjgzOTEzOGRlNzMwZmNlNWY2YTBkNjRlMzMyNThhMmNmOWI1LzY4NzQ3NDcwM2EyZjJmNmQ3OTJkNjI2YzZmNjcyZDc0NmYyZDc1NzM2NTJlNmY3MzczMmQ2MzZlMmQ2MjY1Njk2YTY5NmU2NzJlNjE2YzY5Nzk3NTZlNjM3MzJlNjM2ZjZkMmYzMTM4MmQzMTMwMmQzMTMxMmYzNDM5MzczOTMwMzIzODM4MmU2YTcwNjc?x-oss-process=image/format,png)
流程说明：
（1）客户端（浏览器）发送请求，直接请求到 DispatcherServlet。

（2）DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。

（3）解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由 HandlerAdapter 适配器处理。

（4）HandlerAdapter 会根据 Handler 来调用真正的处理器开处理请求，并处理相应的业务逻辑。
***（这里其实蛮重要的。在SpringMVC中一个典型的适配器模式的应用，因为对于获得的 handle（即controller），需要判断其是属于哪一种类型的controller，如果不适用适配器模式，需要一个一个的去判断是不是哪个类型，需不需要执行，在使用了adpter后，只需要通过adapter实现handle方法就可以了，这样就不违反对拓展开放，修改关闭的开发原则。其实就像我们开发异步队列里面的事件一样，一个handle是一个 event,有它的类型，对支持该类型的handleradpet（eventhandle），执行它。通过容器获得实现adpter的所有具体adpter,依次执行，这样拓展就很方便，只需要自己再实现一个adpter就可以了）***

（5）处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。

（6）ViewResolver 会根据逻辑 View 查找实际的 View。

（7）DispaterServlet 把返回的 Model 传给 View（视图渲染）。

（8）把 View 返回给请求者（浏览器）

## SpringMVC 重要组件说明
1、前端控制器DispatcherServlet（不需要工程师开发）,由框架提供（重要）

作用：Spring MVC 的入口函数。接收请求，响应结果，相当于转发器，中央处理器。有了 DispatcherServlet 减少了其它组件之间的耦合度。用户请求到达前端控制器，它就相当于mvc模式中的c，DispatcherServlet是整个流程控制的中心，由它调用其它组件处理用户的请求，DispatcherServlet的存在降低了组件之间的耦合性。

2、处理器映射器HandlerMapping(不需要工程师开发),由框架提供

作用：根据请求的url查找Handler。HandlerMapping负责根据用户请求找到Handler即处理器（Controller），SpringMVC提供了不同的映射器实现不同的映射方式，例如：配置文件方式，实现接口方式，注解方式等。

3、处理器适配器HandlerAdapter

作用：按照特定规则（HandlerAdapter要求的规则）去执行Handler 通过HandlerAdapter对处理器进行执行，这是适配器模式的应用，**通过扩展适配器可以对更多类型的处理器进行执行** *（**适配器的好处**）*。

4、处理器Handler(需要工程师开发)

注意：编写Handler时按照HandlerAdapter的要求去做，这样适配器才可以去正确执行Handler Handler 是继DispatcherServlet前端控制器的后端控制器，在DispatcherServlet的控制下Handler对具体的用户请求进行处理。 由于Handler涉及到具体的用户业务请求，所以一般情况需要工程师根据业务需求开发Handler。

5、视图解析器View resolver(不需要工程师开发),由框架提供

作用：进行视图解析，根据逻辑视图名解析成真正的视图（view） View Resolver负责将处理结果生成View视图，View Resolver首先根据逻辑视图名解析成物理视图名即具体的页面地址，再生成View视图对象，最后对View进行渲染将处理结果通过页面展示给用户。 springmvc框架提供了很多的View视图类型，包括：jstlView、freemarkerView、pdfView等。 一般情况下需要通过页面标签或页面模版技术将模型数据通过页面展示给用户，需要由工程师根据业务需求开发具体的页面。

6、视图View(需要工程师开发)

View是一个接口，实现类支持不同的View类型（jsp、freemarker、pdf...）

注意：处理器Handler（也就是我们平常说的Controller控制器）以及视图层view都是需要我们自己手动开发的。其他的一些组件比如：前端控制器DispatcherServlet、处理器映射器HandlerMapping、处理器适配器HandlerAdapter等等都是框架提供给我们的，不需要自己手动开发。



着重看一些Adapter:
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS9lZjZhMTRlNGM4NzQ4ZDBlMjgxZjY5ZjI4MDQ2MWFiMjU0OTI1YzJiLzY4NzQ3NDcwM2EyZjJmNmQ3OTJkNjI2YzZmNjcyZDc0NmYyZDc1NzM2NTJlNmY3MzczMmQ2MzZlMmQ2MjY1Njk2YTY5NmU2NzJlNjE2YzY5Nzk3NTZlNjM3MzJlNjM2ZjZkMmYzMTM4MmQzMTMwMmQzMTMxMmYzOTMxMzQzMzMzMzEzMDMwMmU2YTcwNjc?x-oss-process=image/format,png)
(我们常用的注解requestMapper来自RequestMappingHanlerAdapter)

AnnotationMethodHandlerAdapter：通过注解，把请求URL映射到Controller类的方法上。
![在这里插入图片描述](https://img-blog.csdn.net/20180819115012416?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MDc5Mjg3OA==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
还有的例如：
HttpRequestHandler的例子比如静态资源请求处理器ResourceHttpRequestHandler,一般通过ResourceHandlerRegistry程序化配置进来



