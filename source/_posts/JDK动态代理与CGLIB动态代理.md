---
title: JDK动态代理与CGLIB动态代理    
date: 2019-12-17 10:36:55  
tags:  
- Swagger2  
---

### 代理模式
#### 什么是代理模式？  
> **定义：** 给目标对象提供一个代理对象，并由代理对象对目标对象的引用。代理对象和目标对象需要实现同一个接口。  

![image](https://note.youdao.com/yws/api/personal/file/423F6FB82F574D4784FE208A61DD63DD?method=download&shareKey=d5185122b7702ca2f8828c4b3c849ca7)  

#### 为什么要用动态
- **在不改变目标对象的情况下对方法进行增强**  

### JDK的动态代理  
> JDK的动态代理主页涉及到java.lang.reflect 包中的两个类：**Proxy** 和 **InvocationHandler**  

> InvocationHandler是一个接口，**通过实现该接口定义横切逻辑**，并通过**反射机制**调用目标类的代码，动态将横切逻辑和业务逻辑编制在一起。**Proxy 利用 InvocationHandler 动态创建 一个符合某一接口的实例，生成目标类的代理对象。**
- **创建接口和目标类**

```
public interface Star {
    String sing();

    String dance();
}

public class LiuDeHua implements Star {
    @Override
    public String sing() {
        System.out.println("给我一杯忘情水");
        return "唱完";
    }

    @Override
    public String dance() {
        System.out.println("开心的马骝");
        return "跳完";
    }

}
```

- **创建代理类，继承InvocationHandler**  

```
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class StarProxy implements InvocationHandler {
    // 目标类，也就是被代理对象
    private Object target;

    public void setTarget(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 这里可以做增强
        System.out.println("收钱");

        Object result = method.invoke(target, args);

        return result;
    }

    // 生成代理类
    public Object CreatProxyObj() {
        return Proxy.newProxyInstance(target.getClass().getClassLoader(), target.getClass().getInterfaces(), this);
    }

}

```

- **测试**
```
public static void main(String[] args) {
    // 创建被代理对象
    Star ldh = new LiuDeHua();
    // 创建代理对象
    StarProxy proxy = new StarProxy();
    /// 设置被代理对象
    proxy.setTarget(ldh);
    // 生成代理类
    Star obj = (Star) proxy.CreatProxyObj();
    obj.dance();
}
```
- **运行结果**  

![image](https://note.youdao.com/yws/api/personal/file/6645929B2D5E4D32B736BDC1E7A5F2C0?method=download&shareKey=e9dbdf3b18804407e0c84db2118a7c11)

#### cglib动态代理  
> **CGLib全称为Code Generation Library**，是一个强大的高性能，高质量的代码生成类库， 可以在**运行期扩展 Java 类与实现 Java 接口**，CGLib 封装了 asm，可以再**运行期动态生成新的 class**。和 JDK 动态代理相比较：**JDK 创建代理有一个限制，就是只能为接口创建代理实例，而对于没有通过接口定义业务方法的类，则可以通过CGLib创建动态代理。**   

- **创建Cglib代理类**

```
import java.lang.reflect.Method;
import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class CglibProxy implements MethodInterceptor {
    // 根据一个类型产生代理类，此方法不要求一定放在MethodInterceptor中
    public Object CreatProxyedObj(Class<?> clazz) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(clazz);
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        System.out.println("收钱");
        return proxy.invokeSuper(obj, args);
    }

}
```

- **测试**

```
public static void main(String[] args) {
    Star ldh=new LiuDeHua();
    CglibProxy proxy=new CglibProxy();
    Star star= (Star) proxy.CreatProxyedObj(ldh.getClass());
    star.sing();
}
```

- **运行结果**  

![image](https://note.youdao.com/yws/api/personal/file/C9A50BC9B71E49BD90732DA7CD2CC81D?method=download&shareKey=bb1df5e9312f2e8435a0795be64f8752)  

#### cglib动态代理与jdk动态代理区别
> jdk创建对象的速度远大于cglib，这是由于cglib创建对象时需要操作字节码。cglib执行速度略大于jdk，所以比较适合单例模式。另外由于CGLIB的大部分类是直接对Java字节码进行操作，这样生成的类会在Java的永久堆中。如果动态代理操作过多，容易造成永久堆满，触发OutOfMemory异常。spring默认使用jdk动态代理，如果类没有接口，则使用cglib。

