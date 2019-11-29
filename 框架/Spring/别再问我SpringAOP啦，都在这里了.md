**201911.30**
   * [别再问我SpringAOP啦，都在这里了](#别再问我springaop啦都在这里了)
      * [SpringAOP的简单使用](#springaop的简单使用)
      * [SpringAOP实现分析](#springaop实现分析)

# 别再问我SpringAOP啦，都在这里了
在搞明白动态代理之后，终于有勇气去阅读SpringAOP的源码啦。自己这一块的分析是自己读了一部分代码，也看了些相关视频和博客总结的，可能会有一些错误，后面随着学习深入会慢慢改进的！

## SpringAOP的简单使用
在我的SpringBoot项目中，并没有用AOP实现一些功能，向用户权限登陆控制都是用拦截器来实现的，不过也是简单体验了一下AOP的使用，采用注解的方式，配置如下：

```
@Aspect
@Component
public class Log {
    private static  final Logger logger =  LoggerFactory.getLogger(Log.class);
    //在切点之前执行
    @Before("execution(* net.bewithu.questioncommunity.Controller.Index.test(..))")
    public void before(JoinPoint joinPoint){
        for(Object o:joinPoint.getArgs()){
            logger.info(o.toString());
        }
        logger.info("befor");

    }
    //在切点之后执行
    @After("execution(* net.bewithu.questioncommunity.Controller.Index.test(..))")
    public void after(){
        logger.info("after");
    }
}
```
@Aspect声明是个切面类，好像是AOP借鉴了aspectJ的语法。
@Component交给容器管理，自动去扫描，配置

AOP的专业术语

连接点（JoinPoint）：增强执行的位置（增加代码的位置），Spring只支持方法；即我们给的方法
切点（PointCut）：具体的连接点；一般可能通过一个表达式来描述；。即（）里面的位置。包括类方法名

通过 before ,after,around等注解，说明切面方法执行的位置，通过execution(*）制定切点，即在哪些类的哪些方法中进行增强。

我们之前说的代理模式也是对原有方法的一个增强，可以在之前和之后传入一些方法。代理模式和AOP的实现原理又有哪些相似之处呢，与之类别，切点，即在execution(*）中定义的类即使被代理的对象，在执行这些类的方式时，由代理类来实现。好了，然后我们去看具体在底层怎么实现的

## SpringAOP实现分析
我们知道，一般每个类在IOC初始化的时候就已经被构建好了（非延迟加载）存放在工厂中。beanFactory.
然后通过getBean的方式每次去工厂中将实例好的bean取出来。传统的Spring在XML配置好了之后，调用的方式是这个样子：

```
 ApplicationContext context = new ClassPathXmlApplicationContext(new String[] { "applicationContext.xml" });
 
        Product p = (Product) context.getBean("p");
```
即我们在调用getBean之后才能获区工厂中的对象。而在我们执行完 `ApplicationContext context = new ClassPathXmlApplicationContext(new String[] { "applicationContext.xml" });`之后，再调用进行切面后的类的getBean的时候，发现这时候返回的已经不再是我们定义的哪个CLASS了，即这时候返回的已经是被代理的类了，那么啥时候被代理的，用的又是哪些动态代理手段呢。

很明显第一个问题是在IOC的初始化阶段就生成代理类啦，那么来看第二个问题：


在这个接口中可以看出AOP的代理模式是两种的JDK和cglib
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129233956436.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191129234108376.png)
每种具体的实现类（JDK或者Cglib）负责区实现getProx。拿JDK来举个例子看下面代码：
在`JdkDynamicAopProxy`这个类中，该类也实现了我上篇笔记介绍的InvocationHandler的接口
```
/**
    * <ol>
    * <li>获取代理类要实现的接口,除了Advised对象中配置的,还会加上SpringProxy, Advised(opaque=false)
    * <li>检查上面得到的接口中有没有定义 equals或者hashcode的接口
    * <li>调用Proxy.newProxyInstance创建代理对象
    * </ol>
    */
   public Object getProxy(ClassLoader classLoader) {
       if (logger.isDebugEnabled()) {
           logger.debug("Creating JDK dynamic proxy: target source is " +this.advised.getTargetSource());
       }
       Class[] proxiedInterfaces =AopProxyUtils.completeProxiedInterfaces(this.advised);
       findDefinedEqualsAndHashCodeMethods(proxiedInterfaces);
       return Proxy.newProxyInstance(classLoader, proxiedInterfaces, this);
}
```
传进来了类加载器，然后通过AopProxyUtiles这个工具辅助传递了接口。然后就是我之前笔记中很熟悉的

```
 Proxy.newProxyInstance(classLoader, proxiedInterfaces, this)
```
这个方法啦，这个方法内部实现的原理就是我上一篇笔记中所记录的JDK动态代理的原理。

知道了代理类的创建，那么在AOP中，怎么讲方法进行织入呢，之前我们实现的JDK的增强就是简单写在invoke里面的，可是AOP面临的更加复杂，可能有多个切面类和连接点方法。我们来看在AOP里面的 invoke是怎么重写的：这个里面的注释描述的十分清楚啦
```
public  Object invoke(Object proxy, Method method, Object[] args) throwsThrowable {
       MethodInvocation invocation = null;
       Object oldProxy = null;
       boolean setProxyContext = false;
 
       TargetSource targetSource = this.advised.targetSource;
       Class targetClass = null;
       Object target = null;
 
       try {
           //eqauls()方法，具目标对象未实现此方法
           if (!this.equalsDefined && AopUtils.isEqualsMethod(method)){
                return (equals(args[0])? Boolean.TRUE : Boolean.FALSE);
           }
 
           //hashCode()方法，具目标对象未实现此方法
           if (!this.hashCodeDefined && AopUtils.isHashCodeMethod(method)){
                return newInteger(hashCode());
           }
 
           //Advised接口或者其父接口中定义的方法,直接反射调用,不应用通知
           if (!this.advised.opaque &&method.getDeclaringClass().isInterface()
                    &&method.getDeclaringClass().isAssignableFrom(Advised.class)) {
                // Service invocations onProxyConfig with the proxy config...
                return AopUtils.invokeJoinpointUsingReflection(this.advised,method, args);
           }
 
           Object retVal = null;
 
           if (this.advised.exposeProxy) {
                // Make invocation available ifnecessary.
                oldProxy = AopContext.setCurrentProxy(proxy);
                setProxyContext = true;
           }
 
           //获得目标对象的类
           target = targetSource.getTarget();
           if (target != null) {
                targetClass = target.getClass();
           }
 
           //获取可以应用到此方法上的Interceptor列表
           List chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method,targetClass);
 
           //如果没有可以应用到此方法的通知(Interceptor)，此直接反射调用 method.invoke(target, args)
           if (chain.isEmpty()) {
                retVal = AopUtils.invokeJoinpointUsingReflection(target,method, args);
           } else {
                //创建MethodInvocation
                invocation = newReflectiveMethodInvocation(proxy, target, method, args, targetClass, chain);
                //真正执行增强类方法的地方
                retVal = invocation.proceed();
           }
 
           // Massage return value if necessary.
           if (retVal != null && retVal == target &&method.getReturnType().isInstance(proxy)
                    &&!RawTargetAccess.class.isAssignableFrom(method.getDeclaringClass())) {
                // Special case: it returned"this" and the return type of the method
                // is type-compatible. Notethat we can't help if the target sets
                // a reference to itself inanother returned object.
                retVal = proxy;
           }
           return retVal;
       } finally {
           if (target != null && !targetSource.isStatic()) {
                // Must have come fromTargetSource.
               targetSource.releaseTarget(target);
           }
           if (setProxyContext) {
                // Restore old proxy.
                AopContext.setCurrentProxy(oldProxy);
           }
       }
    }
```

此代码段参考自：[https://blog.csdn.net/MoreeVan/article/details/11977115](publicObject%20invoke%28Object%20proxy,%20Method%20method,%20Object%5B%5D%20args%29%20throwsThrowable%20%7B%20%20%20%20%20%20%20%20MethodInvocation%20invocation%20=%20null;%20%20%20%20%20%20%20%20Object%20oldProxy%20=%20null;%20%20%20%20%20%20%20%20boolean%20setProxyContext%20=%20false;%20%20%20%20%20%20%20%20%20%20TargetSource%20targetSource%20=%20this.advised.targetSource;%20%20%20%20%20%20%20%20Class%20targetClass%20=%20null;%20%20%20%20%20%20%20%20Object%20target%20=%20null;%20%20%20%20%20%20%20%20%20%20try%20%7B%20%20%20%20%20%20%20%20%20%20%20%20//eqauls%28%29%E6%96%B9%E6%B3%95%EF%BC%8C%E5%85%B7%E7%9B%AE%E6%A0%87%E5%AF%B9%E8%B1%A1%E6%9C%AA%E5%AE%9E%E7%8E%B0%E6%AD%A4%E6%96%B9%E6%B3%95%20%20%20%20%20%20%20%20%20%20%20%20if%20%28!this.equalsDefined%20&&%20AopUtils.isEqualsMethod%28method%29%29%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20%28equals%28args%5B0%5D%29?%20Boolean.TRUE%20:%20Boolean.FALSE%29;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20%20%20//hashCode%28%29%E6%96%B9%E6%B3%95%EF%BC%8C%E5%85%B7%E7%9B%AE%E6%A0%87%E5%AF%B9%E8%B1%A1%E6%9C%AA%E5%AE%9E%E7%8E%B0%E6%AD%A4%E6%96%B9%E6%B3%95%20%20%20%20%20%20%20%20%20%20%20%20if%20%28!this.hashCodeDefined%20&&%20AopUtils.isHashCodeMethod%28method%29%29%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20newInteger%28hashCode%28%29%29;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20%20%20//Advised%E6%8E%A5%E5%8F%A3%E6%88%96%E8%80%85%E5%85%B6%E7%88%B6%E6%8E%A5%E5%8F%A3%E4%B8%AD%E5%AE%9A%E4%B9%89%E7%9A%84%E6%96%B9%E6%B3%95,%E7%9B%B4%E6%8E%A5%E5%8F%8D%E5%B0%84%E8%B0%83%E7%94%A8,%E4%B8%8D%E5%BA%94%E7%94%A8%E9%80%9A%E7%9F%A5%20%20%20%20%20%20%20%20%20%20%20%20if%20%28!this.advised.opaque%20&&method.getDeclaringClass%28%29.isInterface%28%29%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20&&method.getDeclaringClass%28%29.isAssignableFrom%28Advised.class%29%29%20%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%20Service%20invocations%20onProxyConfig%20with%20the%20proxy%20config...%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20return%20AopUtils.invokeJoinpointUsingReflection%28this.advised,method,%20args%29;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20%20%20Object%20retVal%20=%20null;%20%20%20%20%20%20%20%20%20%20%20%20%20%20if%20%28this.advised.exposeProxy%29%20%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%20Make%20invocation%20available%20ifnecessary.%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20oldProxy%20=%20AopContext.setCurrentProxy%28proxy%29;%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20setProxyContext%20=%20true;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%E8%8E%B7%E5%BE%97%E7%9B%AE%E6%A0%87%E5%AF%B9%E8%B1%A1%E7%9A%84%E7%B1%BB%20%20%20%20%20%20%20%20%20%20%20%20target%20=%20targetSource.getTarget%28%29;%20%20%20%20%20%20%20%20%20%20%20%20if%20%28target%20!=%20null%29%20%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20targetClass%20=%20target.getClass%28%29;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%E8%8E%B7%E5%8F%96%E5%8F%AF%E4%BB%A5%E5%BA%94%E7%94%A8%E5%88%B0%E6%AD%A4%E6%96%B9%E6%B3%95%E4%B8%8A%E7%9A%84Interceptor%E5%88%97%E8%A1%A8%20%20%20%20%20%20%20%20%20%20%20%20List%20chain%20=%20this.advised.getInterceptorsAndDynamicInterceptionAdvice%28method,targetClass%29;%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%E5%A6%82%E6%9E%9C%E6%B2%A1%E6%9C%89%E5%8F%AF%E4%BB%A5%E5%BA%94%E7%94%A8%E5%88%B0%E6%AD%A4%E6%96%B9%E6%B3%95%E7%9A%84%E9%80%9A%E7%9F%A5%28Interceptor%29%EF%BC%8C%E6%AD%A4%E7%9B%B4%E6%8E%A5%E5%8F%8D%E5%B0%84%E8%B0%83%E7%94%A8%20method.invoke%28target,%20args%29%20%20%20%20%20%20%20%20%20%20%20%20if%20%28chain.isEmpty%28%29%29%20%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20retVal%20=%20AopUtils.invokeJoinpointUsingReflection%28target,method,%20args%29;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20else%20%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%E5%88%9B%E5%BB%BAMethodInvocation%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20invocation%20=%20newReflectiveMethodInvocation%28proxy,%20target,%20method,%20args,%20targetClass,%20chain%29;%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20retVal%20=%20invocation.proceed%28%29;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%20Massage%20return%20value%20if%20necessary.%20%20%20%20%20%20%20%20%20%20%20%20if%20%28retVal%20!=%20null%20&&%20retVal%20==%20target%20&&method.getReturnType%28%29.isInstance%28proxy%29%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20&&!RawTargetAccess.class.isAssignableFrom%28method.getDeclaringClass%28%29%29%29%20%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%20Special%20case:%20it%20returned%22this%22%20and%20the%20return%20type%20of%20the%20method%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%20is%20type-compatible.%20Notethat%20we%20can%27t%20help%20if%20the%20target%20sets%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%20a%20reference%20to%20itself%20inanother%20returned%20object.%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20retVal%20=%20proxy;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20return%20retVal;%20%20%20%20%20%20%20%20%7D%20finally%20%7B%20%20%20%20%20%20%20%20%20%20%20%20if%20%28target%20!=%20null%20&&%20!targetSource.isStatic%28%29%29%20%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%20Must%20have%20come%20fromTargetSource.%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20targetSource.releaseTarget%28target%29;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%20%20%20%20if%20%28setProxyContext%29%20%7B%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20//%20Restore%20old%20proxy.%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20%20AopContext.setCurrentProxy%28oldProxy%29;%20%20%20%20%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%20%20%20%7D%20%20%20%20%20%7D%20%20%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%E2%80%94%20%E7%89%88%E6%9D%83%E5%A3%B0%E6%98%8E%EF%BC%9A%E6%9C%AC%E6%96%87%E4%B8%BACSDN%E5%8D%9A%E4%B8%BB%E3%80%8Cmoreevan%E3%80%8D%E7%9A%84%E5%8E%9F%E5%88%9B%E6%96%87%E7%AB%A0%EF%BC%8C%E9%81%B5%E5%BE%AA%20CC%204.0%20BY-SA%20%E7%89%88%E6%9D%83%E5%8D%8F%E8%AE%AE%EF%BC%8C%E8%BD%AC%E8%BD%BD%E8%AF%B7%E9%99%84%E4%B8%8A%E5%8E%9F%E6%96%87%E5%87%BA%E5%A4%84%E9%93%BE%E6%8E%A5%E5%8F%8A%E6%9C%AC%E5%A3%B0%E6%98%8E%E3%80%82%20%E5%8E%9F%E6%96%87%E9%93%BE%E6%8E%A5%EF%BC%9Ahttps://blog.csdn.net/MoreeVan/article/details/11977115)
（感谢大佬们的乐意分享）

注意上述代码段里面我认为最关键的一句： 

    //获取可以应用到此方法上的Interceptor列表
       List chain = this.advised.getInterceptorsAndDynamicInterceptionAdvice(method,targetClass);

这里存的是一个需要增强的列表，通过此列表来进行增强。如果有,则应用通知,并执行joinpoint; 如果没有,则直接反射执行joinpoint。而这里的关键是通知链是如何获取的以及它又是如何执行的，这个方法蛮长的，就不拿出来，简单说下这个方法做了什么事情：

**publicList getInterceptorsAndDynamicInterceptionAdvice(Advised config, Methodmethod, Class targetClass)**



    * 从提供的配置实例config中获取advisor列表,遍历处理这些advisor.如果是IntroductionAdvisor,
    * 则判断此Advisor能否应用到目标类targetClass上.如果是PointcutAdvisor,则判断
    * 此Advisor能否应用到目标方法method上.将满足条件的Advisor通过AdvisorAdaptor转化成Interceptor列表返回.
   
即使遍历我们的连接点，回到之前定义的：`"execution(* net.bewithu.questioncommunity.Controller.Index.test(..))"`
这里就是判断这些是否应用在目标类上，如果上，方法是否也符合，符合的话就进行增强。


到此为止，JDK的代理分析完了，不过JDK只能代理那些接口或者实现接口的类，对于不符合的就是Cglib来完成，关于Cglib实现的过程就更难了，我有时间再分析吧。不过校招和实习真的能问到这一块嘛？我太难了
