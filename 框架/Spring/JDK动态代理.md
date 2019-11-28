**201911.28**
   * [JDK动态代理](#jdk动态代理)
      * [传统的静态代理](#传统的静态代理)
      * [JDK动态代理](#jdk动态代理-1)
      * [使用需要注意的点](#使用需要注意的点)
     
# JDK动态代理
由于在阅读AOP底层实现的时候，涉及到代理模式。如果不先了解清楚JAVA几种动态代理的实现的话，对于深入了解Spring还是有很大困难的，今天终于看明白了JDK动态代理，在这里记录一下我的学习过程。

## 传统的静态代理
首先，先看一下静态代理的实现。我们使用代理模式的目的一方面是在调用被代理类时进行一些增强（就是在调用前后进行一些处理，另一方面使得客户端不能直接操作代理类，提供了权限的控制）。下面的类图就要求的很清楚，代理类和被代理类都要实现相同的接口。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128122746364.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQwODQzNjM5,size_16,color_FFFFFF,t_70)
## JDK动态代理
而在静态代理中，对于代理类必须要进行显式的创建和定义。如果一个系统中有大量的类都需要被代理，且这些类实现了不同的接口。那么一个个去创建代理类则是一个十分麻烦的过程。而动态代理则解决了这个问题，即代理哪个对象时不确定的，在运行时才实现代理，并自动创建一个代理类向上转型为父类型返回，实现了对代理类的增强。

来看一个动态代理的实例：

待实现的接口：

```
public interface subject {
	public void play();
}
```
真正实现的被代理类：

```
public class RealSubject implements subject {

	@Override
	public void play() {
		System.out.print("我是正常方法");		
	}
}
```
JDK代理类的实现：

```
public class JDKProxSubject implements InvocationHandler{
	//传进来的真正实现类
	private Object realSubject;
	//构造器将被代理类传入
	public JDKProxSubject(Object realSubject){
		this.realSubject =realSubject;
	}
	//为了方便定义在这里面，最重要的就是这个里面的方法，传进类加载器，接口，和增强的代理类
	@SuppressWarnings("unchecked")
	public <T> T getProx(){
		return (T) Proxy.newProxyInstance(realSubject.getClass().getClassLoader(), realSubject.getClass().getInterfaces(), this);
	}
	//通过反射实现真实代理类的方法，并进行增强
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Do something before");
        Object result = method.invoke(realSubject, args);
        System.out.println("Do something after");
        return result;
	}

}
```
这个代理类只需要定义一次即可，即可实现对多种实现不同接口的代理类进行代理。因为构造器中传入的是Object的对象，这就体现出来了面向对象的强大之处。我们每次只需要向这个代理类传入真实类调用getProx即可获取代理类。

那么最重要的方法即是 `Proxy.newProxyInstance(realSubject.getClass().getClassLoader(),
realSubject.getClass().getInterfaces(), this);`
这个代理类究竟是怎么实现的呢，沿着这个方法断点一路走下去发现最后是在：

```
defineClass0(loader, proxyName,  proxyClassFile, 0, proxyClassFile.length)
```
这个里面来返回的一个代理类，这是一个native的方法，如果我们通过把这个代理类保存在本地，并反编译时候，就能看出代理类的结构，这个反编译的类来自于网上的一段代码。看到这个代理类的结构，就很明显了，它继承于Prox，同时实现了传进来的接口。将实现的 接口传递给了父类构造器，即调用代理类方法时候回到我们实现的代理类的invoke方法里面。完成了如此精妙的动态代理的设计。

```
public final class $Proxy0 extends Proxy implements Subject {
    private static Method m1;
    private static Method m2;
    private static Method m3;
    private static Method m0;
​
    public $Proxy0(InvocationHandler h) throws  {
        super(h);//通过这句，将invocationHandler实例传给了父类构造器。
    }
​
    public final boolean equals(Object var1) throws  {
        try {
            return ((Boolean)super.h.invoke(this, m1, new Object[]{var1})).booleanValue();
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }
​
    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
​
    //$Proxy0继承了Proxy，且将InvocationHandler h在构造时传给了Proxy。
    //因此，super.h.invoke(this, m3, (Object[])null)其实就是调用的invoker handler的invoke方法。
    //也正是像InvocationHandler定义中所说的，【当proxy instance的方法被调用时，方法调用将会委派为invocation handler的invoke方法】。
    public final void request() throws Exception {
        try { 
            super.h.invoke(this, m3, (Object[])null);
        } catch (Exception | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
​
    public final int hashCode() throws  {
        try {
            return ((Integer)super.h.invoke(this, m0, (Object[])null)).intValue();
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
​
    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[]{Class.forName("java.lang.Object")});
            m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
            m3 = Class.forName("proxy.pattern.Subject").getMethod("request", new Class[0]);
            m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }
}
```

## 使用需要注意的点
使用JDK动态代理只能代理接口或者实现了接口的类，因为在`Prox.newProxInstance`里面明确了要传类的实现接口信息。

实现了一个JDK代理类完全可以代理多个代理类,优势很明显，还解耦合：

```
public class Client {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		subject subject =new JDKProxSubject(new RealSubject()).getProx();
		subject.play();
		subject2 subject2 =new JDKProxSubject(new RealSubjecct2()).getProx();
		subject2.play();
	}

}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191128130808316.png)
