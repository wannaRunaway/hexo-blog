---
title: Android-源码中的单例模式
tags: Android
---



# 单例模式介绍

> 介绍：单例模式属于项目必用模式之一，因为项目中肯定要有user、database、network这些对象，这些对象一是创建起来耗费大量资源，而是网络请求中需要队列去处理请求（因为会有多个请求的状况），所以我们必须保证只有一个对象。
>
> 定义：确保一个类只有一个实例，并且自行实例化并向整个系统提供这个实例。
>
> 使用场景：创建一个对象消耗的资源过多。
>
> 关键点：
>
> · 构造函数不对外开放，一般为private;
>
> · 通过一个静态方法或者枚举返回单例类对象；
>
> · 确保单例类对象有且仅有一个，尤其在多线程环境下；
>
> · 确保单例类对象在反序列化时不会重新构建对象。



# 几种常用的单例模式

> 饿汉模式，不建议

```java
public class SingletonHungryMan {
    //构造函数私有
    private SingletonHungryMan(){
    }
    //直接创建对象
    private static SingletonHungryMan singletonHungryMan = new SingletonHungryMan();
    //静态public提供外部访问
    public static SingletonHungryMan getInstance(){
        return singletonHungryMan;
    }
}
```



> 懒汉模式，不建议
>
> 每次使用的时候都有synchronized同步，造成资源的浪费

```java
public class SingletonLazyMan {
    //构造函数private,无法new
    private SingletonLazyMan(){
    }
    private static SingletonLazyMan singletonLazyMan;
    public synchronized static SingletonLazyMan getInstance(){
        if (singletonLazyMan == null){
            singletonLazyMan = new SingletonLazyMan();
        }
        return singletonLazyMan;
    }
}
```



> Double Check Lock
>
> 用的做多的单例模式，volatile防止指令重拍，避免dcl失效(给对象分配内存--使用构造函数，初始化成员字段，---对象指向内存空间，2、3具有不确定性)

```java
public class SingletonDCL {
    //构造函数private,无法new
    private SingletonDCL(){
    }
    private static volatile SingletonDCL singletonDCL = new SingletonDCL();
    public static SingletonDCL getInstance(){
        //第一次判空，为了检验当前是否有对象
        if (singletonDCL==null){
            synchronized (SingletonDCL.class){
                //第二次判空，第一次初始化的时候，因为多个线程都跑到了这里，所以为了避免重复创建，会再次判空
                if (singletonDCL == null){
                    singletonDCL = new SingletonDCL();
                }
            }
        }
        return singletonDCL;
    }
}

```



> 静态内部类单例，dcl优化是丑陋的，不建议使用，建议如下
>
> 第一次加载类并不会初始化对象，只有第一次调用才会导致对象初始化
>
> 加入readResolve() 反序列化也不会生成新的实例

```java
public class SingletonInnerClass {
    private SingletonInnerClass(){}
    /*
    静态内部类
    * */
    private static class SingletonHolder{
        private static final SingletonInnerClass instance = new SingletonInnerClass();
    }
    public static SingletonInnerClass getInstance(){
        return SingletonHolder.instance;
    }
      private Object readResolve(){
        return SingletonHolder.instance;
    }
}
```



> 枚举单例，反序列化也不会生成新的实例
>
> 容器单例等



# Android源码中的单例模式

> 接下来通过Android源码中的context和layoutinflater来认识Android中的单例模式，因为篇幅较长和很多源码，所以放到下篇。