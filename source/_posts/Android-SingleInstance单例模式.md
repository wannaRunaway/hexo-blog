---
title: Android-SingleInstance单例模式
date: 2021-12-04 20:31:58
tags: Android

---

单例模式在Android非常常见，例如user、networkManager、数据库操作等等。一般使用dcl或者静态内部类单例，带参数选择dcl，不带参数静态内部类。

1、java单例5种写法。

```java
//java 5种单例模式
public class SingleInstance {

    //加载时创建单例
    static class SingleInstanceAdd {
        private SingleInstanceAdd() {
        }

        private static SingleInstanceAdd instanceAdd = new SingleInstanceAdd();

        public static SingleInstanceAdd getInstance() {
            return instanceAdd;
        }

        @Override
        public String toString() {
            return super.toString() + " add completed";
        }
    }

    //懒加载
    static class SingleInstanceLazy {
        private SingleInstanceLazy() {
        }

        private static SingleInstanceLazy singleInstanceLazy = null;

        public static SingleInstanceLazy getInstance() {
            if (singleInstanceLazy == null) {
                singleInstanceLazy = new SingleInstanceLazy();
            }
            return singleInstanceLazy;
        }

        @Override
        public String toString() {
            return super.toString() + " lazy complete";
        }
    }

    //同步锁写法
    static class SingleInstanceSynchronized {
        private SingleInstanceSynchronized() {
        }

        private static SingleInstanceSynchronized singleInstanceSynchronized = null;

        public static synchronized SingleInstanceSynchronized getInstance() {
            if (singleInstanceSynchronized == null) {
                singleInstanceSynchronized = new SingleInstanceSynchronized();
            }
            return singleInstanceSynchronized;
        }
    }

    //dcl
    static class SingleInstanceDCL {
        private SingleInstanceDCL() {
        }

        private static SingleInstanceDCL singleInstanceDCL = null;

        public static SingleInstanceDCL getInstance() {
            if (singleInstanceDCL == null) {
                synchronized (SingleInstanceDCL.class) {
                    if (singleInstanceDCL == null) singleInstanceDCL = new SingleInstanceDCL();
                }
            }
            return singleInstanceDCL;
        }
    }

    //静态内部类写法
    static class SingleInstanceStatic{
        private SingleInstanceStatic(){}
        private static class SingleStatic{
            private static SingleInstanceStatic singleInstanceStatic = new SingleInstanceStatic();
        }
        public SingleInstanceStatic getInstance(){
            return SingleStatic.singleInstanceStatic;
        }
    }

    public static void main(String[] args) {
        print(SingleInstanceAdd.getInstance().toString());
        print(SingleInstanceLazy.getInstance().toString());
    }

    static void print(String message) {
        System.out.println(message);
    }
}
```



2、kotlin单例5种写法。

```kotlin
class SingleInstanceDCL {
}

//单例加载时初始化
private object SingleInstance {
    override fun toString(): String {
        println(super.toString() + " " + javaClass.name + " complete")
        return super.toString()
    }
}

//延迟加载、懒加载
private class SingleInstanceLazy {
    companion object {
        val instanceLazy by lazy(LazyThreadSafetyMode.NONE) {
            SingleInstanceLazy()
        }
    }

    override fun toString(): String {
        println(super.toString() + " " + javaClass.name + " complete")
        return super.toString()
    }
}

//同步锁写法
private class SingleInstanceSynchronized{
    companion object {
        private var singleInstanceSynchronized:SingleInstanceSynchronized?=null
        @Synchronized
        fun get():SingleInstanceSynchronized{
            if (null== singleInstanceSynchronized) singleInstanceSynchronized = SingleInstanceSynchronized()
            return singleInstanceSynchronized as SingleInstanceSynchronized
        }
    }
}

//DCL写法
private class SingleInstanceDCLkt{
    companion object{
        val instanceDCL by lazy(LazyThreadSafetyMode.SYNCHRONIZED){
            SingleInstanceDCL()
        }
    }
}

//静态内部类写法
private class SingleInstanceStatic{
    private object SingleStatic{
        val instanceStatic = SingleInstanceStatic()
    }
    companion object {
        fun getInstance() = SingleStatic.instanceStatic
    }
}

private fun main() {
    SingleInstance.toString()
    SingleInstanceLazy.instanceLazy.toString()
    SingleInstanceSynchronized.get().toString()
}
```

