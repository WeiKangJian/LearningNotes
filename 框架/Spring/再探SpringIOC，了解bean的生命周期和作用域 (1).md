# Bean的作用域
前章回顾和复习：在上一个我的笔记中，通过阅读源码的方式了解了Spring是怎么通过扫描配置文件，通过refesh创建beanfactory然后获取beandefition，保存在工厂的一个defition的map中，扫描完成后即完成了ioc容器的初始化，紧接着依赖注入的过程，进行非延迟加载对象（且作用域为singleton）的settle注入，将反射生成的对象保存在工厂的另一个map中，通过getBean方式提供给外界。

## singleton——默认的唯一bean 实例
那么这样的对象实例的作用域是那些呢，当我们的服务端为客户提供服务的时候，如果没有在配置文件中特地设置scope，默认是单例的，即只生成一个对象，每个请求在创建一个线程的时候都复用这个对象，所以一般我们将这个对象都构造成无状态的（可以理解为不设置可以改变的成员变量）。单例singleton是默认的，可以这样配置：

```
<bean id="ServiceImpl" class="cn.csdn.service.ServiceImpl" scope="singleton">

```

```
@Service
@Scope("singleton")
public class ServiceImpl{
}
```

我们可以指定Bean节点的 lazy-init=”true” 来延迟初始化bean，这时候，只有在第一次获取bean时才会初始化bean，即第一次请求该bean时才初始化。

##  prototype——每个请求都会创建一个新的 bean 
当一个bean的作用域为 prototype，表示一个 bean 定义对应多个对象实例。注意（当配置作用域为prototype的时候，不在IOC初始化完成后依赖注入，即默认在请求的时候容器才创建对象，且不负责对象的销毁，这一点后面会讲到）对有状态的 bean 应该使用 prototype 作用域，而对无状态的 bean 则应该使用 singleton 作用域。和singleton相似，其XML配置方式有下面两种：

```
<bean id="account" class="com.foo.DefaultAccount" scope="prototype"/>  
<bean id="account" class="com.foo.DefaultAccount" singleton="false"/> 
```

注解的配置方式：
```
@Service
@Scope("prototype")
public class ServiceImpl{
}
```

除这两种作用域之外，还有以下两种，不过用的较少：

**request——每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效
session——每一次HTTP请求都会产生一个新的 bean，该bean仅在当前 HTTP session 内有效**
***

# Bean的生命周期
那么在创建（注入）一个bean的时候，容器会执行哪些方法，是怎么管理不同作用域的生命周期的呢?
我最近在开发SpringBoot的时候，一些工具的Service都会实现一个IntiaizeBean的接口，实现其中一个方法，在Bean被注入完成后即执行这个方法，这些都是由容器 本身完成的。具体的生命周期，可以参考下面的这张图：
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jYW1vLmdpdGh1YnVzZXJjb250ZW50LmNvbS9hM2Q0NDE1MTYyZDMwZDQ2NTk3NzlmNmRiMzcxN2Y5YTY4ZmQzYzk3LzY4NzQ3NDcwM2EyZjJmNmQ3OTJkNjI2YzZmNjcyZDc0NmYyZDc1NzM2NTJlNmY3MzczMmQ2MzZlMmQ2MjY1Njk2YTY5NmU2NzJlNjE2YzY5Nzk3NTZlNjM3MzJlNjM2ZjZkMmYzMTM4MmQzOTJkMzEzNzJmMzUzNDM5MzYzNDMwMzcyZTZhNzA2Nw?x-oss-process=image/format,png)
针对上面图的生命周期的流程，看下Bean在使用前调用的这些方法：

**实现*Aware接口 在Bean中使用Spring框架的一些对象**

ApplicationContextAware: 获得ApplicationContext对象,可以用来获取所有Bean definition的名字。
BeanFactoryAware:获得BeanFactory对象，可以用来检测Bean的作用域。
BeanNameAware:获得Bean在配置文件中定义的名字。
ResourceLoaderAware:获得ResourceLoader对象，可以获得classpath中某个文件。
ServletContextAware:在一个MVC应用中可以获取ServletContext对象，可以读取context中的参数。
ServletConfigAware： 在一个MVC应用中可以获取ServletConfig对象，可以读取config中的参数。

在我的springBoot项目的异步事务队列中，在事件处理分发上面就有用到过这个config。

**BeanPostProcessor**
 该接口中包含两个方法，postProcessBeforeInitialization和postProcessAfterInitialization。 postProcessBeforeInitialization方法会在容器中的Bean初始化之前执行
 postProcessAfterInitialization方法在容器中的Bean初始化之后执行。
 
**InitializingBean接口和在配置文件中指定init-method**
这个是用的最多的，在对象注入后即执行此方法，不过多描述。


**DisposableBean接口和配置文件中指定destroy-method**
同上

```
public class MyService {
    //通过<bean>的destroy-method属性指定的销毁方法
    public void destroyMethod() throws Exception {
        System.out.println("执行配置的destroy-method");
    }
    //通过<bean>的init-method属性指定的初始化方法
    public void initMethod() throws Exception {
        System.out.println("执行配置的init-method");
    }
}
```

```
<bean name="giraffeService" class="com,......,..MyService" init-method="initMethod" destroy-method="destroyMethod">
</bean>
```

需要注意的，单例模式管理的对象，其生命周期是容器控制的，容器结束即销毁其对象，而prototype的销毁容器是不管的，是由客户端（即开发者自己负责）进行销毁处理。

下面这段我引用的话很经典：
**Spring 容器可以管理 singleton 作用域下 bean 的生命周期，在此作用域下，Spring 能够精确地知道bean何时被创建，何时初始化完成，以及何时被销毁。而对于 prototype 作用域的bean，Spring只负责创建，当容器创建了 bean 的实例后，bean 的实例就交给了客户端的代码管理，Spring容器将不再跟踪其生命周期，并且不会管理那些被配置成prototype作用域的bean的生命周期。**

参考：
[https://github.com/WeiKangJian/JavaGuide/blob/master/docs/system-design/framework/spring/SpringBean.md](https://github.com/WeiKangJian/JavaGuide/blob/master/docs/system-design/framework/spring/SpringBean.md)

[https://blog.csdn.net/fuzhongmin05/article/details/73389779](https://blog.csdn.net/fuzhongmin05/article/details/73389779)

[https://yemengying.com/2016/07/14/spring-bean-life-cycle/](https://yemengying.com/2016/07/14/spring-bean-life-cycle/)

