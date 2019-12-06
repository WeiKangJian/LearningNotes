**2019.10.17**
 * [再探SpringIOC，深入源码了解构建过程](#再探springioc深入源码了解构建过程)
   * [IOC初始化阶段](#ioc初始化阶段)
   * [IOC依赖注入阶段](#ioc依赖注入阶段)
      
# 再探SpringIOC，深入源码了解构建过程

## IOC初始化阶段
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jbG91ZC5naXRodWJ1c2VyY29udGVudC5jb20vYXNzZXRzLzE3MzYzNTQvNzg5NzM0MS8wMzIxNzliZS0wNzBiLTExZTUtOWVjZi1kN2JlZmM4MDRlOWQucG5n?x-oss-process=image/format,png)

先看这样的一张图，了解Sping IOC的大致构建过程，之前我们已经了解过反射的机制和其能动态构建一个对象，在上图中IOC的机制是从XML中读取对象和对象的各种属性，构造器，构造器等，（Resource)。获取到对象的信息后，将之作为BeanDefition注入到BeanFactory中，在工厂中保存着两个MAP，一个是BeanName和BeanFactory即确定待实例化的对象名称，和BeanDefition，即<String,BeanDefition>.还有一个是当前已经创建好了的对象，如果再遇到请求，直接拿出来用即可。即<String,object>。

来看系统初始化IOC容器时候的栈调用图
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jbG91ZC5naXRodWJ1c2VyY29udGVudC5jb20vYXNzZXRzLzE3MzYzNTQvNzg5NjI4NS84YTQ4ODA2MC0wNmU2LTExZTUtOWFkOS00ZGRkMzM3NTk4NGYucG5n?x-oss-process=image/format,png)

在创建的时候，实际上在准备阶段完成了以下的工作：
在SPRING的主函数中，都需要指定配置文件的位置，后面的扫描解析都是针对这个XML文件的。

```
public ClassPathXmlApplicationContext(String[] configLocations, boolean refresh, 
        ApplicationContext parent) throws BeansException {
    super(parent);
    // 保存位置信息，比如`com/springstudy/talentshow/talent-show.xml`
    setConfigLocations(configLocations);
    if (refresh) {
        // 刷新
        refresh();
    }
}
```

可以看到在制定配置文件后，进行了一次refresh的过程，在refesh的过程中，创建了一个对象工厂，BeanFactory，创建的过程在如下代码中：

```
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        prepareRefresh();
        // Tell the subclass to refresh the internal bean factory.
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        // Prepare the bean factory for use in this context.
        prepareBeanFactory(beanFactory);
        ............
       
```

```
protected final void refreshBeanFactory() throws BeansException {
    // ... ...
    DefaultListableBeanFactory beanFactory = createBeanFactory();
    // ... ...
    loadBeanDefinitions(beanFactory);
    // ... ...
}
```

在创建完对象工厂后，可以看到，紧接着执行的是 `loadBeanDefinitions(beanFactory)`代码，这里就是要扫描XML中一个对象具体的装配信息，包括构造器，属性等，注意这里并不是实例和初始化一个对象，只是获取这个待实例的对象需要装配的信息。

```
protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory)
     throws BeansException, IOException {
    // Create a new XmlBeanDefinitionReader for the given BeanFactory.
    XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
    // ... ...
    // Allow a subclass to provide custom initialization of the reader,
    // then proceed with actually loading the bean definitions.
    initBeanDefinitionReader(beanDefinitionReader);
    loadBeanDefinitions(beanDefinitionReader);
}
```
这里生成了一个beanDefinitionReader 的对象，用来读取XM的Resourse，在XML中处理读取到的每一个元素，可能是import、alias、bean：

```
protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
    // ... ...
    NodeList nl = root.getChildNodes();
    for (int i = 0; i < nl.getLength(); i++) {
        Node node = nl.item(i);
        if (node instanceof Element) {
            Element ele = (Element) node;
            if (delegate.isDefaultNamespace(ele)) {
                // 处理每个xml中的元素，可能是import、alias、bean
                parseDefaultElement(ele, delegate);
            }
            else {
                delegate.parseCustomElement(ele);
            }
        }
    }
    // ... ...
}
```
这里入如果读取到的元素是bean，就进行bean的**解析和注册**，全部解析完后进行注册，是IOC的关键部分，解析的步骤是解析类的各个要实例的字段信息，如构造器，变量的值等，注册即是借助BeanDefitionHolder（存储BeanDefition）将BeanDefition注册到BeanFactory中。

```
protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {
    // 解析
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);
    if (bdHolder != null) {
        bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);
        try {
            // 注册
            // Register the final decorated instance.
            BeanDefinitionReaderUtils.registerBeanDefinition(
                bdHolder, getReaderContext().getRegistry());
        }
        catch (BeanDefinitionStoreException ex) {
            getReaderContext().error("Failed to register bean definition with name '" +
                    bdHolder.getBeanName() + "'", ele, ex);
        }
        // Send registration event.
        getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));
    }
}
```
下面细致了解解析和注册的两个过程的实现：

**处理每个Bean的元素：**

```
ublic AbstractBeanDefinition parseBeanDefinitionElement(
        Element ele, String beanName, BeanDefinition containingBean) {
    // ... ...
    // 创建beandefinition
    AbstractBeanDefinition bd = createBeanDefinition(className, parent);
    parseBeanDefinitionAttributes(ele, beanName, containingBean, bd);
    bd.setDescription(DomUtils.getChildElementValueByTagName(ele, DESCRIPTION_ELEMENT));
    parseMetaElements(ele, bd);
    parseLookupOverrideSubElements(ele, bd.getMethodOverrides());
    parseReplacedMethodSubElements(ele, bd.getMethodOverrides());
    // 处理“Constructor”
    parseConstructorArgElements(ele, bd);
    // 处理“Preperty”
    parsePropertyElements(ele, bd);
    parseQualifierElements(ele, bd);
    // ... ...
}
```
**处理每个Bean的值**

```
public Object parsePropertyValue(Element ele, BeanDefinition bd, String propertyName) {
    String elementName = (propertyName != null) ?
                    "<property> element for property '" + propertyName + "'" :
                    "<constructor-arg> element";
    // ... ...
    if (hasRefAttribute) {
    // 处理引用
        String refName = ele.getAttribute(REF_ATTRIBUTE);
        if (!StringUtils.hasText(refName)) {
            error(elementName + " contains empty 'ref' attribute", ele);
        }
        RuntimeBeanReference ref = new RuntimeBeanReference(refName);
        ref.setSource(extractSource(ele));
        return ref;
    }
    else if (hasValueAttribute) {
    // 处理值
        TypedStringValue valueHolder = new TypedStringValue(ele.getAttribute(VALUE_ATTRIBUTE));
        valueHolder.setSource(extractSource(ele));
        return valueHolder;
    }
    else if (subElement != null) {
    // 处理子类型（比如list、map等）
        return parsePropertySubElement(subElement, bd);
    }
    // ... ...
}
```

然后就是注册啦，将BeanDefition注册进工厂里，即put进HashMap中：


```
public static void registerBeanDefinition(
        BeanDefinitionHolder definitionHolder, BeanDefinitionRegistry registry)
        throws BeanDefinitionStoreException {
    // Register bean definition under primary name.
    String beanName = definitionHolder.getBeanName();
    registry.registerBeanDefinition(beanName, definitionHolder.getBeanDefinition());
    // Register aliases for bean name, if any.
    String[] aliases = definitionHolder.getAliases();
    if (aliases != null) {
        for (String alias : aliases) {
            registry.registerAlias(beanName, alias);
        }
    }
}
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
        throws BeanDefinitionStoreException {
    // ......
    // 将beanDefinition注册
    this.beanDefinitionMap.put(beanName, beanDefinition);
    // ......
}
```
## IOC依赖注入阶段
当完成初始化IOC容器后，如果bean没有设置lazy-init(延迟加载)属性，那么bean的实例就会在初始化IOC完成之后，及时地进行初始化。初始化时会先建立实例，然后根据配置利用反射对实例进行进一步操作：
![在这里插入图片描述](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9jbG91ZC5naXRodWJ1c2VyY29udGVudC5jb20vYXNzZXRzLzE3MzYzNTQvNzkyOTQyOS82MTU1NzBlYS0wOTMwLTExZTUtODA5Ny1hZTk4MmVmNzcwOWQucG5n?x-oss-process=image/format,png)
即可知，在初始化IOC容器之后，如果Bean没有设置延迟加载的属性，那么Bean的实例就会被创建，等待调用。且都是一个实例提供多个调用。这里创建实例就是反射进行属性的注入。来实例化Bean。而且Spring依赖注入Bean实例默认是单例。
```
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, 
        final Object[] args) {
    // Instantiate the bean.
    BeanWrapper instanceWrapper = null;
    if (mbd.isSingleton()) {
        instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
    }
    if (instanceWrapper == null) {
        // 创建bean的实例
        instanceWrapper = createBeanInstance(beanName, mbd, args);
    }
    // ... ...
    // Initialize the bean instance.
    Object exposedObject = bean;
    try {
        // 初始化bean的实例，如注入属性
        populateBean(beanName, mbd, instanceWrapper);
        if (exposedObject != null) {
            exposedObject = initializeBean(beanName, exposedObject, mbd);
        }
    }
    // ... ...
}
```

