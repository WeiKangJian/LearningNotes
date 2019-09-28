**2019.9.28**

   * [初探Spring之IOC](#初探Spring之IOC)
      * [IOC理解](#IOC理解)
         * [IOC控制反转概念](#IOC控制反转概念)
         * [IOC别名依赖注入(DI)]()
         * [IOC技术浅析]()
         * [IOC的优势]()
         * [IOC缺点]()
      * [IOC实例]()
         * [传统配置文件形式]()
         * [注解的形式]()
         
# 初探Spring之IOC
## IOC理解
### IOC控制反转概念
过去我们生成对象的控制权都是掌握在自己的手里，需要对象的时候就自己取new一个出来。这样建立的各个对象之间耦合的十分紧密，对象之间的依赖关系也是特别复杂，对系统的扩展，维护带来了很大的困难。

IOC出现的背景：**解决对象之间的耦合度过高的问题**

IOC的思想：**借助第三方（Spring ）（XML）来实现对象之间的解耦**即对象之间的调用都由第三方来完成，类似于“粘合剂”将所有对象合在一起发挥作用。只需要维护XML就可以了。

软件系统在引入IOC容器之后由于IOC容器的加入，对象A（后面实例中的Category）与对象B(Product)之间失去了直接联系，所以，当对象A运行到(构造器等)需要对象B的时候，IOC容器会主动创建一个对象B注入到对象A需要的地方。

**对象A获得依赖对象B的过程,由主动行为变为了被动行为，控制权颠倒过来了**，这就是“控制反转”这个名称的由来。

### IOC别名依赖注入(DI)
对象A依赖于对象B,当对象 A需要用到对象B的时候，IOC容器就会立即创建一个对象B送给对象A。IOC容器就是一个对象制造工厂，你需要什么，它会给你送去，你直接使用就行了，而再也不用去关心你所用的东西是如何制成的，也不用关心最后是怎么被销毁的，这一切全部由IOC容器包办。

在传统的实现中，由程序内部代码来控制组件之间的关系。我们经常使用new关键字来实现两个组件之间关系的组合，这种实现方式会造成组件之间耦合。IOC很好地解决了该问题，它将实现组件间关系从程序内部提到外部容器，也就是说由容器在**运行期**将组件间的某种依赖关系**动态注入**组件中。


### IOC技术浅析
最核心的技术就是**反射**（即根据给出的类名（字符串方式）来动态地生成对象）

工厂模式：可以把IOC容器看作是一个工厂，这个工厂里要生产的对象都在配置文件中给出定义，然后利用编程语言的的反射编程，根据配置文件中给出的类名生成相应的对象。从实现来看，IOC是把以前在工厂方法里写死的对象生成代码，改变为由配置文件来定义，也就是把工厂和对象生成这两者独立分隔开来，目的就是提高灵活性和可维护性。

### IOC的优势
1：可维护性高，便于调试

2：可复用性好（把复用组件提取出来，应用到其他部分）

3:把对象生成放在配置文件里进行定义，这样，当我们更换一个实现子类将会变得很简单，只要修改配置文件就可以了，完全具有热插拨的特性。

### IOC缺点
1：反射生成对象成本较大
2：运行效率相比较低

## IOC实例
### 传统配置文件形式
**实体类A：**

```
 public class Category {

    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    private int id;
    private String name="default";
}
 
```
**实体类B（其中属性包括实体类A**

```
public class Product {

    private int id;
    private String name="default p1";
    private Category category;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Category getCategory() {
        return category;
    }
    public void setCategory(Category category) {
        this.category = category;
    }
}
```

**XML文件进行类别和产品配置**
指定了类创造所需要信息，和B类中需要的A类的引用，在需要的时候就将A类注入到B类中
```
<bean name="car" class="net.bewithu.pojo.Category">
       <property name="name" value="a car"></property>
   </bean>
    <bean name="subway" class="net.bewithu.pojo.Product">
        <property name="category" ref="car"></property>
        <property name="name" value="a subway"/>
    </bean>
```
**测试代码：**

```
   ApplicationContext context = new ClassPathXmlApplicationContext(new String[] { "applicationContext.xml" });
        Category car = (Category) context.getBean("car");
        Product sub = (Product) context.getBean("subway");
        System.out.println(car.getName()+" "+sub.getName()+" "+sub.getCategory().getName());
```


**输出结果**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190928215910440.png)


### 注解的形式
注解的形式相比于XML方式较为简洁，却增加了理解的难度和代码之间的耦合度，虽然逻辑上各个组件之间仍然是解耦的，但代码上却暴露了对象间的交互。

**实体类A**
只需要添加一行Component来指明当前需要实例化的实例名

```
package net.bewithu.pojo;

import org.springframework.stereotype.Component;

/**
 * demo test
 * @author Mrs W
 * @date 2019/9/28
 */
@Component("car")
public class Category {

    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    private int id;
    private String name;
}
```
**实体类B：**
在需要关联的其他类前通过@Resource的方式指明其被注入的组件

```
package net.bewithu.pojo;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.stereotype.Component;

import javax.annotation.Resource;
/**
 * demo test
 * @author Mrs W
 * @date 2019/9/28
 */
@Component("subway")
public class Product {

    private int id;
    private String name;
    @Resource(name = "car")
    private Category category;
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Category getCategory() {
        return category;
    }
    public void setCategory(Category category) {
        this.category = category;
    }
}
```
**XML配置文件**
只需要配置Spring自己扫描的范围就可以了

```
<context:component-scan base-package="net.bewithu"/>

```


**输出结果**


由于每个类的属性没有初始化，为NULL

![在这里插入图片描述](https://img-blog.csdnimg.cn/20190928221600444.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)

除了以上列出的几个注解的关键字，还有很多关键字（自动找最近的，指定子类等）不一一列出，作用各不相同。后面还会介绍AOP的使用。以及IOC和AOP的实现原理的探究。
