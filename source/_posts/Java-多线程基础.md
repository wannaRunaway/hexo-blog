---
title: Java-多线程基础
date: 2022-03-08 14:04:47
tags: Java

---

> 一个进程包括多个线程，进程包括由操作系统分配出来的内存空间，线程不能独立存在，必须存在进程之上。一个进程一直运行，知道所有的非守护线程结束运行后才停止。

# 线程状态

## 1、新建状态

使用了new关键字和thread或其子类建立一个线程后，就处于新建状态；

## 2、就绪状态

start之后线程进入了就绪状态。进入了就绪队列中，等待jvm线程调度器的调用。

## 3、运行状态

就绪状态的线程获取到了cpu资源，就可以执行run，此时，它可以变为阻塞、就绪、死亡状态。

## 4、阻塞状态

线程执行了sleep、suspend等方法，失去了cpu资源，该线程就从运行状态进入阻塞状态。在睡眠时间已到或者获取到了资源后就可以重新进入就绪状态，三种：

1、等待阻塞，线程执行wait方法；

2、同步阻塞，线程获取synchronized同步锁失败；

3、其他阻塞，线程通过sleep()或者join()发出io请求时，线程就会进入阻塞状态，sleep()超时，join()等待线程终止或超时，线程重新进入就绪状态；

# 线程的优先级

线程优先级是一个整数，在1...10之间，Thread.MIN_PRIORITY---Thread.MAX_PRIORITY，默认情况下，每一个线程分配一个优先级 NORM_PRIORITY=5

# 创建线程的3种方式

## 1、实现runnable接口创建线程

```java
public class RunnalbeThreadType implements Runnable{
    private Thread thread;
    private String threadName;
    RunnalbeThreadType(String threadName){
        this.threadName = threadName;
        System.out.println(threadName+" creating");
    }
    @Override
    public void run() {
        System.out.println("start running");
        for (int i = 0; i < 5; i++) {
            System.out.println(i+" is cycling");
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
    public void start(){
        System.out.println("starting");
        if (thread == null){
            thread = new Thread(this, threadName);
            thread.start();
        }
    }
}
    public static void main(String[] args) {
        var runnalbeThreadType = new RunnalbeThreadType(RunnalbeThreadType.class.getCanonicalName());
        runnalbeThreadType.start();
    }
```

## 2、继承thread类创建线程

```java
public class ExtendThread extends Thread{
    private Thread thread;
    private String threadName;
    ExtendThread(String threadName){
        this.threadName = threadName;
        System.out.println("creating");
    }

    @Override
    public void run() {
        super.run();
        System.out.println("running");
        for (int i = 0; i < 5; i++) {
            System.out.println(i+" cycling");
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void start(){
        System.out.println("starting");
        if (thread == null){
            thread = new Thread(this, threadName);
            thread.start();
        }
    }
}
        var thread = new ExtendThread(ExtendThread.class.getName());
        thread.start();
```

# Join()使用

```java
public class JavaBase {
    public static void main(String[] args) throws InterruptedException {
//        var runnalbeThreadType = new RunnalbeThreadType(RunnalbeThreadType.class.getCanonicalName());
//        runnalbeThreadType.start();
//        var extendThread = new ExtendThread(ExtendThread.class.getName());
//        extendThread.start();
        var thread = new Thread(()->{
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread()+"starting");
        });
        System.out.println("start...");
        thread.start();
        thread.join();
        System.out.println("end...");
    }
}

start...
Thread[Thread-0,5,main]starting
end...
```

Thread.join挂起当前线程，等待线程执行完毕。

# 线程同步

## 1、多线程本身不同步的问题

```java
package threadbase;

class Counter {
    public static int count = 0;
}

class ThreadAdd implements Runnable {
    Thread thread;

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            Counter.count = Counter.count + 1;
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(this.getClass().getName() + " " + Counter.count);
        }
    }

    public void start() {
        if (thread == null) {
            thread = new Thread(this);
            thread.start();
        }
    }
}

class ThreadDec implements Runnable {
    Thread thread;

    @Override
    public void run() {
        for (int i = 0; i < 10000; i++) {
            Counter.count = Counter.count - 1;
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(this.getClass().getName() + " " + Counter.count);
        }
    }

    public void start() {
        if (thread == null) {
            thread = new Thread(this);
            thread.start();
        }
    }
}

public class Threadsynchronize {
    public static void main(String[] args) throws InterruptedException {
        var threadAdd = new ThreadAdd();
        var threaddec = new ThreadDec();
        threadAdd.start();
        threaddec.start();
        threadAdd.thread.join();
        threaddec.thread.join();
        System.out.println("final result is "+Counter.count);
    }
}

outputs: 
final result is -2
```

多线程模式下，要保证逻辑正确，对共享变量必须进行原子化操作：即某一个线程执行时，其他线程等待。

## 2、synchronized锁

```java
package threadbase;

class Counter {
    public static int count = 0;
    public static final Counter counter = new Counter();
}

class ThreadAdd implements Runnable {
    Thread thread;

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            synchronized (Counter.counter) {
                Counter.count = Counter.count + 1;
            }
            System.out.println(this.getClass().getName() + " " + Counter.count);
        }
    }

    public void start() {
        if (thread == null) {
            thread = new Thread(this);
            thread.start();
        }
    }
}

class ThreadDec implements Runnable {
    Thread thread;

    @Override
    public void run() {
        for (int i = 0; i < 100; i++) {
            synchronized (Counter.counter) {
                Counter.count = Counter.count - 1;
            }
            System.out.println(this.getClass().getName() + " " + Counter.count);
        }
    }

    public void start() {
        if (thread == null) {
            thread = new Thread(this);
            thread.start();
        }
    }
}

public class Threadsynchronize {
    public static void main(String[] args) throws InterruptedException {
        var threadAdd = new ThreadAdd();
        var threaddec = new ThreadDec();
        threadAdd.start();
        threaddec.start();
        threadAdd.thread.join();
        threaddec.thread.join();
        System.out.println("final result is "+Counter.count);
    }
}

final result is 0

```

synchronized锁保证了只能有一个线程某段时间在执行，获取锁要用同一个对象才有效，但是synchronized问题是性能下降，1是加锁释放锁需要消耗资源，2是代码块无法并发执行。

synchronized锁加在方法上就是同步代码块，synchronized方法加锁对象就是this，静态方法就是同步锁加到类上面去。

# 死锁

## 1、可重入锁

jvm允许同一个线程重复获取同一个锁，能被一个线程重复获取的锁，就叫做可重入锁。

每次获取一次锁，就+1；退出synchronized代码块就-1。减到0的时候，才会释放锁。

```java
/**
 * 可重入锁
 * */
public class RepeatBolck {
    private int count;
    public synchronized void add(int number){
        count = count+number;
        System.out.println("add "+count);

    }

    public synchronized void des(int number){
        count = count - number;
        System.out.println("des "+count);
    }

    public static class MainBolck {
        public static void main(String[] args) {
            RepeatBolck repeatBolck = new RepeatBolck();
            repeatBolck.add(10);
            repeatBolck.des(2);
            System.out.println("result "+repeatBolck.count);
        }
    }
}
```

## 2、死锁

获取多个锁的时候，不同线程获取不同锁对象可能导致死锁，例如下面代码：

Thread1.oneblock()获取到了deadblock1,再去获取deadblock2;

Thread2.secondblock()获取到了deadblock2,再去获取deadblock1;

这样可能会导致thread1得到了deadblock1等待获取deadblock2;

thread2得到了deadblock2等待获取deadblock1。

```java
package threadbase;

public class DeadBolck {
    private DeadBolck deadBolck1 = new DeadBolck();
    private DeadBolck deadBolck2 = new DeadBolck();
    private int count = 0;

    public void onebolck(){
        synchronized (deadBolck1){
            count = count + 10;
            System.out.println("oneblock1 "+count);
            synchronized (deadBolck2){
                count = count + 10;
                System.out.println("oneblock2 "+count);
            }
        }
    }

    public void secondblock(){
        synchronized (deadBolck2){
            count = count - 20;
            System.out.println("secondblock1 "+count);
            synchronized (deadBolck1){
                count = count -20;
                System.out.println("secondblock2 "+count);
            }
        }
    }


    static class MainRun{
        public static void main(String[] args) throws InterruptedException {
            DeadBolck deadBolck = new DeadBolck();
            var thread1 = new Thread(deadBolck::onebolck);
            var thread2 = new Thread(deadBolck::secondblock);
            thread1.start();
            thread2.start();
            thread1.join();
            thread2.join();
            System.out.println("main end");

        }
    }
}

```

死锁发生后，只能强制结束jvm进程。

避免死锁：获取锁的顺序要一致。

**总结：**
**java的synchronized是可重入锁**
**死锁发生的条件是多线程持有不同的锁，并试图获取对方已持有的锁，导致互相等待。**

**避免死锁的方法：多线程获取锁顺序要保持一致性。**

