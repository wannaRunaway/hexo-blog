---
title: Android-Factory工厂模式
date: 2021-12-04 11:34:19
tags: Android

---

工厂模式非常常见，一般分为简单工厂模式、工厂模式、抽象工厂

1、简单工厂。类创建型模式，通过参数使用静态方法创建工厂。

```kotlin
//简单工厂模式，对象切换在静态方法完成
interface SimpleFactoryDraw{
    fun draw()
}

private class Circle: SimpleFactoryDraw{
    override fun draw() {
        println(javaClass.name+ " isinvoking " + Thread.currentThread().stackTrace[1].methodName)
    }
}

private class Shape: SimpleFactoryDraw{
    override fun draw() {
        println(javaClass.name+ " isinvoking " + Thread.currentThread().stackTrace[1].methodName)
    }
}

private class Rectangle: SimpleFactoryDraw{
    override fun draw() {
        println(javaClass.name+ " isinvoking " + Thread.currentThread().stackTrace[1].methodName)
    }
}

fun getShape(shape: String): SimpleFactoryDraw {
    return when(shape){
        "Circle" -> Circle()
        "Rectangle" -> Rectangle()
        else -> Shape()
    }
}

fun main(){
    var circle:SimpleFactoryDraw = getShape("Circle")
    var rectangle:SimpleFactoryDraw = getShape("rectangle")
    circle.draw()
    rectangle.draw()
}
```



2、一般工厂模式。创建一个接口，让子类确定实例化哪个类，让对象实例化延迟到了子类。

```kotlin
//一般工厂模式，将选择工厂抽象
private interface IViewDraw {
    fun onDraw()
}

private class Theme(var name: String) {
    @JvmName("getName1")
    fun getName(): String {
        return name
    }
}

private interface ThemeFactory {
    fun createTheme(): Theme
}

private class Scrollbar(var themeFactory: ThemeFactory) : IViewDraw {
    override fun onDraw() {
        println(javaClass.name + " is " + Thread.currentThread().stackTrace[1].methodName + " ${themeFactory.createTheme().name}")
    }
}

private class DarkThemeFactory : ThemeFactory {
    override fun createTheme(): Theme {
        return Theme("DarkTheme")
    }
}

private class LightThemeFactory : ThemeFactory {
    override fun createTheme(): Theme {
        return Theme("LightTheme")
    }
}

fun main() {
    val themeFactory = DarkThemeFactory()
    val scrollbar = Scrollbar(themeFactory)
    scrollbar.onDraw()
    val lightThemeFactory = LightThemeFactory()
    val lightScrollbar = Scrollbar(lightThemeFactory)
    lightScrollbar.onDraw()
}
```



3、抽象工厂。抽象工厂将工厂方法进一步抽象，可以有多个factory。

```kotlin
//抽象工厂模式
private interface Background{
    fun drawBackground()
}

private interface Border{
    fun drawBorder()
}

private interface AdsShape{
    fun draw()
}

private interface AbsThemeFactory{
    fun createBackground():Background
    fun createBorder():Border
}

private class DarkBackground : Background{
    override fun drawBackground() {
        println("draw ${Thread.currentThread().stackTrace[1].methodName}")
    }
}

private class DarkBorder : Border{
    override fun drawBorder() {
        println("draw ${Thread.currentThread().stackTrace[1].methodName}")
    }
}

private class AbScrollbar(var absThemeFactory: AbsThemeFactory): AdsShape{
    override fun draw() {
        absThemeFactory.createBackground().drawBackground()
        absThemeFactory.createBorder().drawBorder()
    }
}

private class AbsDarkThemeFactory : AbsThemeFactory{
    override fun createBackground(): Background {
        return DarkBackground()
    }

    override fun createBorder(): Border {
        return DarkBorder()
    }
}

fun main(){
    AbScrollbar(AbsDarkThemeFactory()).draw()
}
```

