Kotlin 中要求变量初始化时一定要赋初值

## 定义为 Nullable 对象

```kotlin
var tv:TextView ?= null
```

将变量初始化成 Nullable 类型，当得到变量真实值时再重新赋值给它。但是在使用的时候需要加上操作符 `?` 或者 `!!`

```kotlin
tv?.setOnClickListener{/**/}
```

```kotlin
tv!!.setOnClickListener{/**/}
```

> 需要记住的是，使用 `!!` 操作符之后，若在后边的操作中将 tv 置为 null 了，则会报 `NullPointException` crash

## 使用 lateinit 定义

```kotlin
var lateinit str:String
...
fun init(){
 	str = "Hello world" 
}
```

使用 lateinit 关键字定义变量时，不需要给变量赋初值，只需要定义好变量的类型即可，`但是一定要保证初始化变量之后再使用`。不然会报异常。

使用时也可以通过方法判断变量是否初始化

```kotlin
if(::str.isInitialized){
	// has initialized  
}else{
	// not initialized
}
```



## Custom getter

自定义变量的 getter 函数

```kotlin
val textView: TextView
    get() = findViewById(R.id.tv_name)
```

但是使用这种方法时，每次 textView 变量被使用时，都会调用 `findViewById` 函数一次。

## 使用 Lazy 定义

```kotlin
/**
  * 第一种方式，指定模式
  * LazyThreadSafetyMode.SYNCHRONIZED:线程安全的，某个线程操作 test 的初始化时，会将其锁住，其他线程不可操作
  * LazyThreadSafetyMode.PUBLICATION:test 未初始化时，可以在多个线程同时被调用初始化，但是只有第一次返回的值作为其 value
  * LazyThreadSafetyMode.NONE:线程不安全，行为不可控，仅在可以保证 test 不会在多个线程被调用初始化时使用
*/
private val test by lazy(LazyThreadSafetyMode.SYNCHRONIZED) {
        123
}
// 第二种定义方式
val str: String by lazy {
	"Hello World"
}

// 第三种定义方式：
val str1 by lazyOf("Hello world")

/** 第四种定义方式：
	* 默认使用 LazyThreadSafetyMode.SYNCHRONIZED 模式
	* 如果初始化变量时抛出了异常，那么下次使用时会尝试重新初始化
	* 返回的实例使用传递进来的对象 xx 进行同步
**/
val str2 by lazy(xxx:Any?) {
    123 + 5
}
// by lazy 定义 Activity 中的 View
val tv by lazy {
	findViewById<TextView>(R.id.tv_name)
}
```

当加上 `by lazy` 操作符之后，当变量 `x` 第一次使用的时候回执行 lambda 放方块中的初始化代码为变量初始化，lambda 中的最后一行即为返回的变量值，`再次使用 x 时，使用的是上一次赋值的值`，代码如下

> `使用 by lazy 关键字定义的变量，必须声明为 val 类型`
>
> 要注意，Activity 中的 View ，不能在 setContentView() 之前调用，否则还是会 Crash。
>
> Fragment 中的 View，同样不能在 onCreateView() 方法之前调用

调用 by lazy 时，实际走到了如下代码

```kotlin
// LazyJvm.kt
public actual fun <T> lazy(lock: Any?, initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer, lock)

private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    // 如果不传 lock 对象，则使用自己进行同步
    private val lock = lock ?: this

  	// 本质上是做了个代理，在需要初始化的地方调用了 xx.value，若 value 未初始化，就会调用我们传递进来的函数lai
    override val value: T
        get() {
            val _v1 = _value
          // 如果有值，就直接返回
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

           // 根据传递进来的对象 lock 进行同步
            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                } else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }

   ......
}
```

如下，对 ViewModelActivity decomipled 之后，可以看到，声明了 by lazy 的变量会在 Activity 的构造函数中委托给 `LazyKt` 类

```kotlin
public final class ViewModelActivity extends AppCompatActivity {
......
  static final KProperty[] $$delegatedProperties = new KProperty[]{(KProperty)Reflection.property1(new PropertyReference1Impl(Reflection.getOrCreateKotlinClass(ViewModelActivity.class), "test", "getTest()I")), (KProperty)Reflection.property1(new PropertyReference1Impl(Reflection.getOrCreateKotlinClass(ViewModelActivity.class), "str", "getStr()Ljava/lang/String;")), (KProperty)Reflection.property1(new PropertyReference1Impl(Reflection.getOrCreateKotlinClass(ViewModelActivity.class), "str2", "getStr2()Ljava/lang/String;")), (KProperty)Reflection.property1(new PropertyReference1Impl(Reflection.getOrCreateKotlinClass(ViewModelActivity.class), "tvName", "getTvName()Landroid/widget/TextView;"))};
  private final Lazy test$delegate;
   @NotNull
   private final Lazy str$delegate;
   @NotNull
   private final Lazy str2$delegate;
   private final Lazy tvName$delegate;
  
  /*********** 编译生成的方法 ********/
private final int getTest() {
      Lazy var1 = this.test$delegate;
      KProperty var3 = $$delegatedProperties[0];
      boolean var4 = false;
      return ((Number)var1.getValue()).intValue();
   }

   @NotNull
   public final String getStr() {
      Lazy var1 = this.str$delegate;
      KProperty var3 = $$delegatedProperties[1];
      boolean var4 = false;
    	// getValue() 方法其实调用了 lazy lambda 传递过去的表达式
      return (String)var1.getValue();
   }

   @NotNull
   public final String getStr2() {
      Lazy var1 = this.str2$delegate;
      KProperty var3 = $$delegatedProperties[2];
      boolean var4 = false;
         	// getValue() 方法其实调用了 lazy lambda 传递过去的表达式
      return (String)var1.getValue();
   }
  
protected void onCreate(@Nullable Bundle savedInstanceState) {
      super.onCreate(savedInstanceState);
      this.setContentView(-1300009);
      ......
      String var10000 = this.TAG;
      
  		// 在需要调用的地方其实是调用了编译生成的方法
      Log.i(this.TAG, "onCreate | str twice：[" + this.getStr() + ']');
      Log.i(this.TAG, "onCreate | str2：[" + this.getStr2() + ']');
      TextView var2 = this.getTvName();
      Intrinsics.checkExpressionValueIsNotNull(var2, "tvName");
      var2.setText((CharSequence)"HHHHHH");
   }

public ViewModelActivity() {
   this.test$delegate = LazyKt.lazy(LazyThreadSafetyMode.SYNCHRONIZED, (Function0)null.INSTANCE);
   this.str$delegate = LazyKt.lazy((Function0)(new Function0() {
      // $FF: synthetic method
      // $FF: bridge method
      public Object invoke() {
         return this.invoke();
      }

      @NotNull
      public final String invoke() {
         Log.i(ViewModelActivity.this.TAG, "initStr");
         return "Hello World";
      }
   }));
   this.str2$delegate = LazyKt.lazyOf("Hello world");
   this.tvName$delegate = LazyKt.lazy(this.getLifecycle(), (Function0)(new Function0() {
      // $FF: synthetic method
      // $FF: bridge method
      public Object invoke() {
         return this.invoke();
      }

      public final TextView invoke() {
         return (TextView)ViewModelActivity.this.findViewById(-1000019);
      }
   }));
}
}
```

## 参考文章

[Kotlin: Lazy 和 Lateinit 的使用](https://zhuanlan.zhihu.com/p/34257123)

