# Kotlin 中的类

Kotlin 中的`变量、类`默认都是 `final` 的，且都是继承自 `Any` 基类，如果想要类可以被继承，需要声明关键字 `open`

```kotlin
open class User{
		open var name:String = "John" // 可被重写
    var age:Int = 18
  
    open fun foo() = "foo"  // 可被重写
  
  	fun bar() = "bar"
}
```

## Kotlin 中的继承

Kotlin 中的类的构造函数与 Java 中有一些不同

> - 在`Kotlin`中，允许有一个主构造函数和多个二级构造函数（辅助构造函数）。其中主构造函数是类头的一部分。
> - 关键字或者构造函数名：`constructor(参数)`

### 构造函数

#### 存在主构造函数

> 主构造函数是函数头的一部分，主构造函数使用默认可见性修饰符修饰，或者不具有注释符的时候，可省略 `constructor` 关键字
>
> // 类似下面两种情况的，都必须存在constructor关键字，并且在修饰符或者注释符后面。 
>
> ```kotlin
> class Test private constructor(num: Int){ } 
> class Test @Inject constructor(num: Int){ }
> ```
>
> 

当存在主构造函数时，一般在主构造函数中实现基类中参数最多的构造函数，其他`二级构造函数`调用 `this` 关键字将方法委派到本类的其他构造函数。最终直接或间接的委派给了主构造函数。在 `init` 构造代码块中，可以访问到主构造函数中的参数。

```kotlin
class CustomView(context: Context?, attrs: AttributeSet?, defStyleAttr: Int, defStyleRes: Int) :
    View(context, attrs, defStyleAttr, defStyleRes) {

   init{
    	Log.d("tag",context)
   }
   
    constructor(context: Context?, attrs: AttributeSet?) : this(
        context,
        attrs,
        0
    )

    constructor(context: Context?, attrs: AttributeSet?, defStyleAttr: Int) : this(
        context,
        attrs,
        defStyleAttr,
        0
    )

}
```

#### 无主构造函数

若不存在主构造函数，所有的二级构造函数需要调用 `super` 关键字委派到父类的构造函数

```kotlin
class CustomView : View {

    constructor(context: Context) : super(context)

    constructor(context: Context, attributeSet: AttributeSet) : super(context, attributeSet) 

    constructor(context: Context, attributeSet: AttributeSet, defStyleAttr: Int) : super(
        context,
        attributeSet,
        defStyleAttr
    ) 

    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int, defStyleRes: Int) : super(
        context,
        attrs,
        defStyleAttr,
        defStyleRes
    )
}
```

### 重载与重写

#### 函数的重写和重载

> Kotlin 基类中的函数，若没有加 `open` 关键字，则其子类中不得定义函数名与基类中相同的名字，编译会报错

```kotli
open class People {
    fun test() {

    }
}

class User : People() {
		// 编译报错
    //fun test() {
		//}
		
    // 编译同样报错
    //override fun test(){
    //}
}
```

> 子类重写基类中的方法时，加了 `final` 修饰，则此方法到这个类为止，下一个子类不允许再重写此方法。

```
open class People {
    open fun test() {
        println("people test")
    }
}

open class User1 : People() {
    final override fun test() {
        println("user test")
    }
}
```

函数的重载与 Java 中一样

```kotlin
class KotlinTest3 {
    companion object {
        @JvmStatic
        fun main(args: Array<String>) {
            val user = User1()
            user.test()
            user.test("John")
        }
    }
}

open class People {
    open fun test() {
        println("people test")
    }
}

open class User1 : People() {
    final override fun test() {
        println("user test")
    }

    fun test(name: String) {
        println("name:$name")
    }

  	/** 
  	  * java 重载注解，加此注解之后，有的参数必须有 default value
  	  * 在转换为 JVM 代码是，会生成三个方法，带有不同的参数
  	**/
    @JvmOverloads
    fun hello(name: String, age: Int = 0, sex: String = "") {

    }
}
```

#### 重写属性

基类中用 `val` 修饰的属性，在子类中可以用 `val` 或者 `var` 重写，反之不可以。重写的属性若是调用了 `super.name` ，则不管 `setter` 中传入了什么，最后都是返回基类中的值。

```kotlin
open class People {
    open val name: String = "John"
}

open class User1 : People() {
//    override val name: String
//        get() = "Marry"

//    override var name: String = "abc"
//        set(value) {
//            field = value
//        }

    override var name: String ="abc"
        // 此处这么写，不管给 name set 什么值，最后都是返回基类中的值
        get() = super.name
        set(value) {
            field = value
        }
}
```

#### 在主构造函数中重写属性

```kotlin
class KotlinTest3 {
    companion object {
        @JvmStatic
        fun main(args: Array<String>) {
            val user = User1("Marry")
            println("name:${user.name}")
        }
    }
}

open class People {
    open val name: String = "John"
}

open class User1(override val name: String) : People() {
}
```

最后输出 `name:Marry`

> 继承代价太高，用的时候需要考虑好

### 覆盖规则

> 这里的覆盖规则，是指实现类继承了一个基类，并且实现了一个接口类，当我的基类中的方法、属性和接口类中的函数重名的情况下，怎样去区分实现类到底实现哪一个中的属性或属性。 这一点和一个类同时实现两个接口类，而两个接口都用同样的属性或者函数的时候是一样的。

```kotlin
open class A{
	open fun test1(){println("基类 A 中的 test1() 函数")}
}

interface B {
	fun test1(){println("接口 B 中的 test1() 函数")}
}

class C:A(),B{
	override fun test1(){
  	super<A>.test1()
    super<B>.test1()
  }
}
```

