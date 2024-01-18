---
title: Android-proxy代理模式
date: 2021-12-02 17:01:00
tags: Android

---

代理模式是一种非常常见的模式，在java或者kotlin中，当我们去扩展某个对象也需要保护这个对象的时候，就可以使用代理模式。代理模式是一种控制模式。

1、静态代理。委托类和代理类实现同一个接口，代理类初始化时候将委托类也初始化，然后就可以使用委托类对象调用方法等，而代理类新加入方法属性可以对属性类做扩展，这样非常安全。

```kotlin
//静态代理
private interface ISubject{
    fun init()
    fun doAction()
}

//真实对象委托类
class RealSubject: ISubject{
    override fun init() {
        println("${javaClass.name} function init()")
    }

    override fun doAction() {
        println("${javaClass.name} function doAction")
    }
}

//代理类
class ProxySubject(var realSubject: RealSubject) : ISubject{
    override fun init() {
        println("${javaClass.name} function init")
    }

    override fun doAction() {
        println("${javaClass.name} doAction start")
        realSubject.doAction()
        println("${javaClass.name} doAction end")
    }
}

fun main(){
    var proxySubject = ProxySubject(RealSubject())
    proxySubject.init()
    proxySubject.doAction()
}
```



2、动态代理。java动态代理例如jdk动态代理，proxyHandle实现InvocationHandler, 使用invoke()中的method.invoke(obj, args)。使用Proxy.newProxyInstance(classload, class, new ProxyHandle(myObject))利用反射创建的class加载进入内存中，然后调用doAction()。

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class JDKProxyJavaPattern {

    //动态代理
    private interface Isubject {
        void init();

        void doAction();
    }

    public static class ProxySubject1 implements Isubject {

        @Override
        public void init() {
            print(getClass().getName() + " init");
        }

        @Override
        public void doAction() {
            print(getClass().getName() + " doAction");
        }
    }

    public static class ProxySubject2 implements Isubject {
        @Override
        public void init() {
            print(getClass().getName() + " init");
        }

        @Override
        public void doAction() {
            print(getClass().getName() + " doAction");
        }
    }

    private static class ProxyHandle implements InvocationHandler {
        private Object subject;

        public ProxyHandle(Object subject) {
            this.subject = subject;
        }

        @Override
        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            print(getClass().getName() + " start");
            Object result = method.invoke(subject, args);
            print(getClass().getName() + " end");
            return result;
        }
    }

    public static void main(String[] args) {
        Isubject isubject1 = (Isubject) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class<?>[]{Isubject.class}, new ProxyHandle(new ProxySubject1()));
        isubject1.init();
        isubject1.doAction();
        Isubject isubject2 = (Isubject) Proxy.newProxyInstance(Thread.currentThread().getContextClassLoader(), new Class<?>[]{Isubject.class}, new ProxyHandle(new ProxySubject2()));
        isubject2.init();
        isubject2.doAction();
    }

    static void print(String message) {
        System.out.println(message);
    }

}
```

3、kotlin代理模式。通过by关键字实现。

```kotlin
//kotlin中的动态代理，使用by关键字。效率比java反射高很多
interface ProxyKotlinPatternInterface {
    fun doAction()
}

class ProxyKotlinPatternSubject : ProxyKotlinPatternInterface {
    override fun doAction() {
        println("${javaClass.name}  ${Thread.currentThread().stackTrace[1].methodName}")
    }
}

class ProxyKotlinPatternExecute(proxyKotlinPatternInterface: ProxyKotlinPatternInterface) : ProxyKotlinPatternInterface by proxyKotlinPatternInterface{
    fun execute(){
        println("${javaClass.name}  ${Thread.currentThread().stackTrace[1].methodName}")
    }
}

fun main(){
    val proxyKotlinPatternExecute = ProxyKotlinPatternExecute(ProxyKotlinPatternSubject())
    proxyKotlinPatternExecute.execute()
    proxyKotlinPatternExecute.doAction()
}
```

