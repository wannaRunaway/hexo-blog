---
title: Android-observer观察者模式
date: 2021-12-01 17:33:49
tags: Android

---

观察者模式是一种一对多的对象行为模式，当被观察者改变时，所有依赖于他的观察者都会自动更新。常见例子比如Android中的button.setOnclickListener()，broadcastReceiver, rxJava等等。这里通过一个自定义观察者模式来看：

```kotlin
interface Observer {
    //接受被观察者的信息
    fun update(observable: Observable, objects: Any)
}

open class Observable {
    //存储所有的观察者对象
    private var list: ArrayList<Observer> = ArrayList<Observer>()

    //添加观察者对象
    fun addObserver(observe: Observer) {
        observe?.apply {
            list.add(observe)
        }
    }

    //移除观察者
    fun deleteObserver(observe: Observer) {
        list?.apply {
            if (list.size > 0) {
                list.remove(observe)
            }
        }
    }

    //有活动行为通知观察者
    fun notifyObservers(obj: Any) {
        for (element in list) {
            element.update(this, obj)
        }
    }
}

//观察者
class FishMan(var name: String) : Observer {
    override fun update(observable: Observable, objects: Any) {
        println("$name observed this fish is eating $objects")
    }
}

class Shark(var name: String) : Observer {
    override fun update(observable: Observable, objects: Any) {
        println("$name observed this fish is eating $objects")
    }
}

//被观察者
class Fish(var name: String) : Observable() {

    fun eating(food: String) {
        notifyObservers(food)
    }

    fun toStrings(): String {
        return "Fish"
    }
}

fun main() {
    println("hi is me")
    var fish = Fish("smallFish")
    var fishManJack = FishMan("Jack")
    var fishManTonny = FishMan("Tonny")
    var fishManMash = FishMan("Mash")
    var shark = Shark("Onil")
    fish.addObserver(fishManJack)
    fish.addObserver(fishManTonny)
    fish.addObserver(fishManMash)
    fish.addObserver(shark)
    fish.deleteObserver(fishManJack)
    fish.eating("smallFish")
}
```

1、抽象的observer接口，方便多个观察者调用；

2、class observable存储所有的观察者对象，有list存储观察者，addObserver()添加观察者，deleteObservable()删除观察者，notifyObserver()将list中的observers.notify(), notify()是抽象后的接口, 实现的方法体在每个具体的观察者如fishMan、shark等类中。

3、init Observable Fish, init Observers Fishman and Shark, 将观察者add进去被观察者父类中，然后调用被观察者eating方法，到Observable中的notifyObservers(), 再到Observer中的update(), 最后到达FishMan和Shark中的update()。
