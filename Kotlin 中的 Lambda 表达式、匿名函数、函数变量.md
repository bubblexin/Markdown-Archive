## `Kotlin` 中的 `Lambda` 表达式、匿名函数、函数变量

```kotlin
...
fun test(param: ITest) {
    // 声明匿名函数并赋值给一个函数变量
    val a = fun(a: Int) {
        a + 1
    }
    // 声明函数变量 b，将函数 getUsersWithNameLiveData 赋值给它
    val b = ::sum
    // 调用函数变量 b，效果与直接调用 getUsersWithNameLiveData 一致
    b(1, 2)
  
  // kotlin 建议这么写，用 lambda 表达式的形式
    setITest {
       Log.i(TAG, "it:$it")
    }
}

private fun sum(a: Int, b: Int): Int {
    return a + b
}

/**
 * 声明一个包含函数类型参数的函数
 * funParam 真正的实现在调用此方法时传入的 lambda 表达式
 */
fun setITest(funParam: (a: Int) -> Unit) {
    val a = 3
    funParam(a)
}
...
```

### 把函数当做函数的参数传递以及赋值给变量

```kotlin
a(fun (param: Int): String {
  return param.toString()
});
val a = fun (param: Int): String {
  return param.toString()
}
```



## 在 Java 中常用的如下接口回调写法，如点击事件定义如下

```java
// XXView.java
...
private OnClickListener mOnClickListener;
interface OnClickListener{
		public void onClick(View view);
}

public void setOnClickListener(OnClickListener onClickListener(){
	this.mOnClickListener = onClickListener;
}
                               
public void xxx(){
	...
    // 在有需要的地方调用接口的 onClick 方法
	if(this.mOnClickListener !=null){
  		mOnClickListener.onClick(this);
	}
}
...
```

在代码中调用

```java
// xxActivity.java
public void xx(){
		XXView v = XXView(this);
  	v.setOnClickListener(new OnClickListener{
    		@Override
      	public void onClick(View view){
          
        }
    });
}
```

### 在 Kotlin 中建议使用函数的方式

```kotlin
// XXView
class XXView:TextView {
  ...
    fun xxx(funOnClick: (v: View) -> Unit) {
      ...
      funOnClick(v)
    }
  
  	// 设置 IView 对象
	  private var mIView:IView ?= null
  	fun setOnIView(iView:IView){
  		mIview = iView  
    }
  ...
}

interface IView{
		fun onClick(v:View)
}

// xxActivity.kt
...
fun xx(){
    val view = XXView(this) 
  // 传递函数参数，具体实现在这里
  	view.xxx{view->
    		Log.d("test",view)
    }
  
  // ⚠️ 会报错，Kotlin 不能这么使用
  	view.setOnIView{
    
    }
} 
```

> 如上例子所示，当调用 `view.setOnIView` 时会报错，这是因为 `setOnIView` 的参数本质上是一个接口，如果我们传递一个匿名函数过去，无法推断其入参是什么类型的。那为什么调用 `view` 的 `setOnClickListener` 没问题呢？因为在 `Java` 代码和 `Kotlin` 代码之间是可以这么用的，系统会为 `View` 的` setOnClickListener` 方法做一个桥接，靠上下文的推断告诉我们其需要的参数类型。
>
> ```kotlin
> fun setOnClickListener(onClick: (View) -> Unit) {
>   this.onClick = onClick
> }
> ```
>
> Kotlin 是不支持使用 Lambda 的方式来简写匿名类对象的，因为我们有函数类型的参数嘛，所以这种单函数接口的写法就直接没必要了。那你还支持它干嘛？
>
> 不过当和 Java 交互的时候，Kotlin 是支持这种用法的：当你的函数参数是 Java 的单抽象方法的接口的时候，你依然可以使用 Lambda 来写参数。但这其实也不是 Kotlin **增加**了功能，而是对于来自 Java 的单抽象方法的接口，Kotlin 会为它们额外创建一个把参数替换为函数类型的桥接方法，让你可以间接地创建 Java 的匿名类对象。
>
> 这就是为什么，你会发现当你在 Kotlin 里调用 View.java 这个类的 setOnClickListener() 的时候，可以传 Lambda 给它来创建 OnClickListener 对象，但你照着同样的写法写一个 Kotlin 的接口，你却不能传 Lambda。因为 Kotlin 期望我们直接使用函数类型的参数，而不是用接口这种折中方案。

[【码上开学】Kotlin 的高阶函数、匿名函数和 Lambda 表达式](https://mp.weixin.qq.com/s/SsMtsw45dMdr3uI55cFmyQ)

