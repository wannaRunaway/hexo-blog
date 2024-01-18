---
title: Android-ChainOfResponsibility责任链模式
date: 2021-12-07 00:12:57
tags: Android

---



责任链在Android中特别常用的设计模式，例如okhttp的intercepter等等.

```kotlin
//责任链模式
enum class Topic {
    NONE, BUTTON, DIALOG, APPLICATION
}

private abstract class HelpHandler(var helpHandler: HelpHandler?, var topic: Topic) {
    open fun handleHelp() {
        helpHandler?.handleHelp()
    }

    open fun hasHelp(): Boolean {
        return topic != Topic.NONE
    }
}

private abstract class Widget(helpHandler: HelpHandler, topic: Topic) : HelpHandler(helpHandler, topic) {
    init {
        println(javaClass.name+" init()")
    }
}

private class Button(helpHandler: HelpHandler, topic: Topic) : Widget(helpHandler, topic) {
    override fun handleHelp() {
        if (hasHelp()) {
            println(javaClass.name + " " + Thread.currentThread().stackTrace[1].methodName)
        } else {
            super.handleHelp()
        }
    }
}

private class Dialog(helpHandler: HelpHandler, topic: Topic) : Widget(helpHandler, topic) {
    override fun handleHelp() {
        if (hasHelp()) {
            println(javaClass.name + " " + Thread.currentThread().stackTrace[1].methodName)
        } else {
            super.handleHelp()
        }
    }
}

private class Application(helpHandler: HelpHandler?, topic: Topic) : HelpHandler(helpHandler, topic) {
    constructor(topic: Topic) : this(null, topic) {
    }
}

fun main(){
    var application = Application(Topic.APPLICATION)
    var dialog = Dialog(application, Topic.DIALOG)
    var button = Button(dialog, Topic.BUTTON)
    button.handleHelp()

    var button1 = Button(dialog, Topic.NONE)
    button1.handleHelp()
}
```
