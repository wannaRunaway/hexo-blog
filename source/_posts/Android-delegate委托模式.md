---
title: Android-delegate委托模式
date: 2021-11-24 01:13:54
tags: Android

---



委托模式，与java继承不同的是，它体现的是代码组合的思想。解决问题思路是将委托对象中的某些操作交给其他被委托对象。可以将代码分离，使得某个对象不是那么臃肿。



这是最简单的委托模式：

Print 中的 fun print()被委托给RealPrint，fun print()打印。

```kotlin
fun main() {
    var print = Print()
    print.print()
}
//委托模式。将某个对象中的操作委托给其他对象，是代码组合的思想

public class RealPrint() {
    //被委托者
    fun print() {
        print("yeah, it is realPrint")
    }
}

public class Print() {
    //委托者
    var realPrint: RealPrint = RealPrint()
    fun print() {
        realPrint.print()
    }
}
```



使用接口实现委托是很常见的一种模式。

1、委托类实现接口方法，然后创建接口引用去掉用接口方法。

2、委托类中function创建被委托类instance.

3、被委托类实现接口，接口function中具体执行代码。

4、委托类切换interface引用instance, 执行function.

```kotlin
fun main(){
    var executeMe:ExecuteMe = ExecuteMe()
    executeMe.toDelegateFirst()
    executeMe.delegateFirst()
    executeMe.delegateSecond()
    executeMe.toDelegateSecond()
    executeMe.delegateFirst()
    executeMe.delegateSecond()
}

//使用接口的委托模式
interface Delegateinterface{
    fun delegateFirst()
    fun delegateSecond()
}

//被委托类
class DelegateFirst:Delegateinterface{
    override fun delegateFirst() {
        print("yeah! it's ${javaClass.name} delegateFirst()")
    }

    override fun delegateSecond() {
        print("yeah! it's ${javaClass.name} delegateSecond()")
    }

}

//被委托类
class DelegateSecond:Delegateinterface{
    override fun delegateFirst() {
        print("yeah! it's ${javaClass.simpleName} delegateFirst()")
    }

    override fun delegateSecond() {
        print("yeah! it's ${javaClass.name} delegateSecond()")
    }

}

//委托类
class ExecuteMe: Delegateinterface{
    lateinit var delegateinterface:Delegateinterface
    override fun delegateFirst() {
        delegateinterface.delegateFirst()
    }

    override fun delegateSecond() {
        delegateinterface.delegateSecond()
    }

    fun toDelegateFirst(){
        delegateinterface = DelegateFirst()
    }

    fun toDelegateSecond(){
        delegateinterface = DelegateSecond()
    }
}
```



