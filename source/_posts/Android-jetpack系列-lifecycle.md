---
title: Android-jetpack系列-lifecycle
tags: Android
---



# Lifecycle的使用

Android jetpack系列有很多组件，lifecycle属于感知生命周期的组件，下面通常代码：

```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var myLocationListener: MyLocationListener
    val DITAG = "debug"
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        myLocationListener = MyLocationListener(applicationContext){
        }
    }

    override fun onStart() {
        super.onStart()
        myLocationListener.start()
    }

    override fun onStop() {
        super.onStop()
        myLocationListener.stop()
    }

   inner class MyLocationListener(
        private val context: Context,
        private val callback: (Location) -> Unit
    ) {
        fun start() {
            //connect to locationListener
            Log.d(DITAG, "locationListener start")
        }

        fun stop() {
            //disconnect to locationListener
            Log.d(DITAG, "locationListener stop")
        }
    }
}
```

这是Android很常见的listener定义，activity onstart中listener start, activity onstop时listener stop, callback中的function可以通过start和stop来控制。	



问题在于：

1、如果activity中有很多listener start or stop的function, 不止一个listener, 那在activity生命周期中方法就会特别多非常难以维护。这在产品后期的迭代开发中时致命的。

2、如果activity onstart需要长时间的启动，而onstop在onstart之前执行，那就会导致组件留存下来。



```kotlin
class MainActivityLifecycle : AppCompatActivity() {

    private lateinit var binding: ActivityMainLifecycleBinding
    private lateinit var myLocationListener: MyLocationListener
    private val DITAG = "debug"

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        binding = ActivityMainLifecycleBinding.inflate(layoutInflater)
        setContentView(binding.root)
        myLocationListener = MyLocationListener(applicationContext, lifecycle){
            //update ui
        }
        lifecycle.addObserver(myLocationListener)
        myLocationListener.enable()
    }

    override fun onStart() {
        super.onStart()
        Log.d(DITAG, "activity onstart")
    }

    override fun onStop() {
        super.onStop()
        Log.d(DITAG, "activity onstop")
    }

    inner class MyLocationListener(
        private val context: Context,
        private val liftcycle: Lifecycle,
        private val callback: (Location) -> Unit
    ) :DefaultLifecycleObserver{

        private var enabled = false
        override fun onStart(owner: LifecycleOwner) {
            super.onStart(owner)
            Log.d(DITAG, "onstart ")
            if (enabled){
                //connect

            }
        }

        fun enable(){
            enabled = true
            if (liftcycle.currentState.isAtLeast(Lifecycle.State.STARTED)){
                //connect if not connected
            }
        }

        override fun onStop(owner: LifecycleOwner) {
            super.onStop(owner)
            //disconnect if connected
            Log.d(DITAG, "onstop ")

        }
    }
}
```

使用lifecycle就只需要lifecycle.addObserver(instance implement defautlifecycleObserver),然后override onstart onstop, 就可以初始化和结束关闭资源回收了，不需要在activity中去写很多代码，直接从view中代码独立出来，非常容易维护。
