

[TOC]

要在项目中使用 AndroidX 的话，

1. compile SDK 版本大于等于 Android9.0（API 28）

2. 在 `gradle.properties` 中设置

   ```groovy
   android.useAndroidX=true // 设置为 true，Android 插件会使用合适版本的 AndroidX library 代替 Support Library 库，默认是 false
   android.enableJetifier=true // 设置为 true，Android 插件使用 AndroidX 依赖重写已经存在的第三方 libraries 完成自动迁移，默认 false
   ```

## 实现一个自定义的 `LifeCycleObserver`

### 通过实现 `LifeCycleObserver` 接口，使用注解的方式

```kotlin
class TestObserver constructor(private val mContext: Context) : LifecycleObserver, {
	private val TAG = "TestObserver" + "："

	/**
 	* 使用注解
 	*/
	@OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
	fun onCreate() {
  	  Toast.makeText(mContext, "onStart invoked", Toast.LENGTH_SHORT).show()
    	Log.i(TAG, "onCreate")
	}
}
```



### 通过实现 `DefaultLifecycleObserver` 接口的方式

如果使用 Java8，推荐使用实现 `DefaultLifecycleObserver` 接口的方式，而不是注解的方式

`DefaultLifecycleObserver` 必须在大于等于 `Android N(24)` 以上的版本上才可以使用，并且需要指定 kotlin JVM 编译版本为 `1.8`，不然会报错：

>  Cannot inline bytecode buit with JVM target 1.8 into bytecode that is being built with JVM target 1.6.Please specify proper '-jvm-target' option

解决方法是在 app 目录下的 `build.gradle` 文件里的 `android` 闭包中添加如下代码

```groovy
android{
	...
	kotlinOptions {
  	jvmTarget = "1.8"
	}
	...
 }
```

同时添加依赖

```groovy
 // alternately - if using Java8, use the following instead of compiler
implementation "android.arch.lifecycle:common-java8:$lifecycle_version"
```

###LifeCycleActivity

```kotlin
class LifeCycleActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_life_cycle)
        lifecycle.addObserver(TestObserver(this))
    }
}
```

### TestObserver

```kotlin
class TestObserver constructor(private val mContext: Context) : DefaultLifecycleObserver, LifecycleEventObserver {

    private val TAG = "TestObserver" + "："

    /**
     * 如果一个类同时实现了 DefaultLifecycleObserver 和 LifecycleEventObserver 接口
     * DefaultLifecycleObserver 中的方法会先执行，然后 LifecycleEventObserver#onStateChanged(LifecycleOwner, Lifecycle.Event) 才会执行
     */
    override fun onStateChanged(source: LifecycleOwner, event: Lifecycle.Event) {
        Log.i(TAG, "onStateChanged | [source, event]：[$source, $event]")
    }

    override fun onCreate(owner: LifecycleOwner) {
        super.onCreate(owner)
        Log.i(TAG, "onCreate | [owner]：[$owner]")
    }

    override fun onStart(owner: LifecycleOwner) {
        super.onStart(owner)
        Log.i(TAG, "onStart | [owner]：[$owner]")
    }

    override fun onResume(owner: LifecycleOwner) {
        super.onResume(owner)
        Log.i(TAG, "onResume | [owner]：[]")
    }

    override fun onPause(owner: LifecycleOwner) {
        super.onPause(owner)
        Log.i(TAG, "onPause | [owner]：[$owner]")
    }

    override fun onStop(owner: LifecycleOwner) {
        super.onStop(owner)
        Log.i(TAG, "onStop | [owner]：[]")
    }

    override fun onDestroy(owner: LifecycleOwner) {
        super.onDestroy(owner)
        Log.i(TAG, "onDestroy | [owner]：[]")
    }

    /**
     * 实现 DefaultLifecycleObserver 接口，同时使用了 OnLifecycleEvent 注解，则注解无效
     */
    @OnLifecycleEvent(Lifecycle.Event.ON_CREATE)
    fun onCreate() {
        Toast.makeText(mContext, "onStart invoked", Toast.LENGTH_SHORT).show()
        Log.i(TAG, "onCreate")
    }
}
```

如果一个类同时实现了` DefaultLifecycleObserver` 和 `LifecycleEventObserver` 接口， `DefaultLifecycleObserver` 中的方法会先执行，然后 `LifecycleEventObserver#onStateChanged(LifecycleOwner, Lifecycle.Event)` 才会执行，如日志所示


![](https://tva1.sinaimg.cn/large/00831rSTly1gdj8g69nzsj31kj044dj3.jpg)

> ` DefaultLifecycleObserver` 中的 `onCreate、onStart、onResume` 回调会在 `LifeCycleOwner` 的对应回调之后返回。`onPause、onStop、onDestroy` 回调会在 `LifeCycleOwner` 的对应回调之前返回。

### 各个事件（`LifeCycle Event`）返回后，`LifeCycleOwner`（Activity、Fragment）对应的 `State`

![](https://tva1.sinaimg.cn/large/00831rSTly1gdk2aqhvorj310207j78i.jpg)

### onSaveInstanceState 执行的情况下

![image-20200406151342035](https://tva1.sinaimg.cn/large/00831rSTly1gdk2gxvmkmj314d08oadt.jpg)

官方文档说，AppCompatActivity 的 onStop 方法在 onSaveInstanceState() 方法之后执行，如果在执行完 onSaveInstanceState() 到执行 onStop 之间的这段时间 UI 发生了变动，会导致异常，在 LifeCycle beta2 或比这更低的版本中，将 state 置为了 CREATE 状态，同时不传递 event，这样各处检查 currentState 的时候得到的会是正确的 state，即 CREATE 状态。我使用的 `androidx.appcompat:appcompat:1.1.0@aar` 版本的 AppCompatActivity，当执行了 onSaveInstanceState 方法时，LifeCycle 的 Stop 事件是先一步在 onSaveInstanceState 之前回调的，不过此时状态已经是 CREATE 状态了，执行完 onSaveInstanceState 之后，Activity 的 onStop 方法会被执行。

##  LifeCycle

`LifeCycle` 使用两个主要枚举来跟踪其相关组件的生命周期状态

### Event

### State

![](https://developer.android.com/images/topic/libraries/architecture/lifecycle-states.svg)



## ViewModel

`ViewModel` 的唯一职责就是为 UI 管理数据。他永远不应该访问你的视图层次结构，也不应该保留对 Activity 和 Fragment 的引用



## LiveData

`onActive` 和 `onInActive` 的调用时机，是在 LifeCycle 的 State 为 `START` 和 `RESUME`（Activity/Fragment 为活动状态 ） 时，打印日志发现，`onActive` 方法是在 `onStart` 执行之后，`onResume` 执行之前执行，`onInactive` 方法是在 `onPause` 和 `onStop` 之间

![image-20200407164440771](https://tva1.sinaimg.cn/large/00831rSTly1gdlaq0hq21j327q0acgug.jpg)



map：

observer A

switchMap：

A 变化，导致 B 跟着变化，observer B 的变化