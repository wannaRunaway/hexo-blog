---
title: Android-Build模式
date: 2021-12-03 21:19:37
tags: Android
---



Build模式在Android中非常常见，例如dialog创建、okhttp配置等等。build模式是将复杂对象和表示分离，一步步构造对象。

1、静态内部类Build中function setConponent()将各个属性设置进去并返回this

2、build()创建Product(build), 参数构造时将build属性设置给product

3、Product.Build().setConponent().build创建product instance

```kotlin
//Build模式
class Product() {
    private lateinit var head: String
    private lateinit var body: String
    private lateinit var foot: String

    constructor(build: Build) : this() {
        this.head = build.head
        this.body = build.body
        this.foot = build.foot
    }

    override fun toString(): String {
        return ("Product: head $head body$body foot$foot")
    }


    class Build {
        lateinit var head: String
        lateinit var body: String
        lateinit var foot: String

        fun setHead(head: String): Build {
            this.head = head
            return this
        }

        fun setBody(body: String): Build {
            this.body = body
            return this
        }

        fun setFoot(foot: String): Build {
            this.foot = foot
            return this
        }

        fun build(): Product {
            return Product(this)
        }
    }
}

fun main() {
    var product = Product.Build()
        .setHead("yeah I'm head").setBody(" now is body").setFoot(" atlast foot").build()
    println(product.toString())
}
```
