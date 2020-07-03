[TOC]

## 引入

在 app 的 build.gradle 中的 android 闭包下配置

```groovy
android {
    ...
    dataBinding {
        enabled true
    }
}
```

## 使用

在需要使用 DataBinding 的布局文件中，使用 `<layout></layout>` 包裹 xml 布局文件，里边的根布局只能有一个，布局文件相关的参数信息包裹在 `<data></data>` 中

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools">
    <data>
      	// 引入此页面要使用的变量
        <variable
                name="viewModel"
                type="com.okay.viewdemo.databinding.DataBindingViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            tools:context=".databinding.DatabindingActivity">

        <TextView
                android:id="@+id/tv_name"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@{viewModel.name}"
                app:layout_constraintLeft_toLeftOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                tools:text="Hello World" />

        <Button
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:onClick="@{()->viewModel.onClick()}"
                android:text="Button"
                android:textAllCaps="false"
                app:layout_constraintLeft_toLeftOf="parent"
                app:layout_constraintTop_toBottomOf="@id/tv_name" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

## 绑定数据

> 系统会为每个布局文件生成一个绑定类，默认的名称基于布局文件的名称，它会转换为 Pascal 大小写形式并在末尾添加 Binding 后缀。如 `activity_main.xml`，对应的 `ViewDataBinding` 类为 `ActivityMainBinding`，此类包含从布局属性（例如，`user` 变量）到布局视图的所有绑定，并且知道如何为绑定表达式指定值。

实例代码如下：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: ActivityMainBinding = DataBindingUtil.setContentView(
            this, R.layout.activity_main)

    binding.user = User("Test", "User")
}
```

也可以先 bind 相关 View 再 add 到 Activity 容器

```kotlin
val binding = ActivityDatabindingBinding.inflate(layoutInflater)
setContentView(binding.root)
```

`inflate()` 方法还有另外一个版本，即使用 `LayoutInflater` 对象，还需要 `ViewGroup` 参数

```kotlin
val binding: ActivityDatabindingBinding = ActivityDatabindingBinding.inflate(getLayoutInflater(), viewGroup, false)
```

如果 View 是使用其他方法 `inflate` 出来的，支持单独绑定

```kotlin
val view = LayoutInflater.from(context).inflate(R.layout.test,root,false)
val binding : MyLayoutBinding = MyLayoutBinding.bind(view)
```

如果需要在 `Fragment`、`ListView` 或者 `RecyclerView` 适配器中使用数据绑定，可以使用 `inflate` 方法

```kotlin
val listItemBinding = ListItemBinding.inflate(layoutInflater,viewGroup,false)
// or 使用 DataBindingUtil 生成
val listItemBinding = DataBindingUtil.inflate(layoutInfalter,R.layout.list_item,viewGroup,flase)
```



## 表达式语言

### 属性引用

表达式可以使用 `@{表达式}` 格式来引用属性，对于字段、getter 和 ObservableField 对象都一样

```xml
android:text="@{user.name}"
```

### Null 合并运算符

```xml
android:text="@{user.displayname ?? user.lastname}"
```

等价于

```xml
android:text="@{user.displayname!=null?user.displayname:user.lastname}"
```

### 避免 Null 指针异常

对于字符串，如果为空，则默认值为 `null`，如果为 int，则默认值为 `0`

### 访问集合

可使用 `[]` 运算符访问常见集合，例如数组、列表、稀疏列表和映射。

> ⭐️ **注意**：要使 XML 不含语法错误，您必须转义 `<` 字符。例如：不要写成 `List` 形式，而是必须写成 `List<String>`。

```xml
<data>
        <import type="android.util.SparseArray"/>
        <import type="java.util.Map"/>
        <import type="java.util.List"/>
        <variable name="list" type="List&lt;String>"/>
        <variable name="sparse" type="SparseArray&lt;String>"/>
        <variable name="map" type="Map&lt;String, String>"/>
        <variable name="index" type="int"/>
        <variable name="key" type="String"/>
    </data>
    …
    android:text="@{list[index]}"
    …
    android:text="@{sparse[index]}"
    …
    android:text="@{map[key]}"
```

`@{map[key]}` 可以替换为 `@{map.key}`

## 事件处理

DataBinding 中的时间处理可以使用两种机制

- [方法引用](https://developer.android.google.cn/topic/libraries/data-binding/expressions#method_references)：在表达式中，您可以引用符合监听器方法签名的方法。当表达式求值结果为`方法引用`时，数据绑定会将方法引用和所有者对象封装到监听器中，并在目标视图上设置该监听器。如果表达式的求值结果为 `null`，则数据绑定不会创建监听器，而是设置 `null` 监听器。
- [监听器绑定](https://developer.android.google.cn/topic/libraries/data-binding/expressions#listener_bindings)：这些是在事件发生时进行求值的 lambda 表达式。数据绑定始终会创建一个要在视图上设置的监听器。事件被分派后，监听器会对 lambda 表达式进行求值。

> 方法引用和监听器绑定之间的主要区别在于实际监听器实现是在绑定数据时创建的，而不是在事件触发时创建的。如果您希望在事件发生时对表达式求值，则应使用[监听器绑定](https://developer.android.google.cn/topic/libraries/data-binding/expressions#listener_bindings)。

### 方法引用

> android:onClick="@{viewHandler::click}"

方法引用的方式类似于自己实现了监听器的匿名函数，将此函数的引用传递给具体方法，方法调用时，通过传递进来的方法引用调用函数。

事件可以直接绑定到处理脚本方法，类似于使用 `android:onClick` ，将其值指定为 Activity 中的某一方法的方式。与使用 `View` 的 `onClick` 特性相比，主要优点是`表达式在编译时进行处理，所以如果方法不存在或其签名不正确，会收到编译时错误。`

要将事件分配给其处理脚本，请使用常规绑定表达式，并以要调用的方法名称作为值。如下所示：

### 监听器绑定

> android:onCheckedChanged="@{(view,isChecked)->viewHandler.onCheckChanged(view,isChecked)}"
>
> android:onClick="@{()->viewHandler.onClick()}"
>
> android:onLongClick="@{(view)->viewHandler.onLongClick(view,viewModel.name)}"

监听器绑定是在事件发生时运行的绑定表达式。类似于方法引用，但是允许运行任意的数据绑定表达式。此功能适用于 `Gradle 2.0 版及更高版本`的 Android Gradle 插件。

> `返回值必须与监听器的预期返回值（比如 onLongClick 返回 boolean，则你引用的函数引用也必须范湖 boolean 类型的返回值）相匹配（与其返回值无效除外）`

在定义表达式是，有两种写法，一种是不定义参数的，一种是定义传递给方法的参数的写法

1. 忽略方法的所有参数
2. 命名所有参数，甚至后边可以加上自己需要的参数，但是方法返回值需要与监听器预期返回值相匹配

#### 无参数的

```kotlin
// 点击事件，无参数
fun onClick(){
		Log.i(TAG,"onClick")
}
```

将点击事件绑定到 `onClick` 方法。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
                name="viewHandler"
                type="com.okay.viewdemo.databinding.ViewUtil" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            tools:context=".databinding.DatabindingActivity">
        <Button
                android:id="@+id/btn"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:onClick="@{()->viewHandler.onClick()}"/>
    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```

#### 有参数的

```kotlin
// 长按事件，返回值必须与监听的事件的返回值类型一致
// 可以在 xml 中的 lambda 表达式中传递多个参数，如下传递了 ViewModel 中的 name 参数
fun onLongClick(view: View, name: String?): Boolean {
    Log.i(TAG, "onLongClick | [view, name]：[$view, $name]")
    return true
}

// CheckChanged 事件，返回值类型与监听的时间的返回值类型一致
fun onCheckChanged(view:View,isChecked:Boolean) {
		Log.i(TAG, "onCheckChanged | [view, isChecked]：[$view, $isChecked]")
}
```

然后可以将长按事件绑定到 `onLongClick` 方法，将 `CheckBox` 的选中事件绑定到 `onCheckChanged` 方法。

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
                name="viewModel"
                type="com.okay.viewdemo.databinding.DataBindingViewModel" />

        <variable
                name="viewHandler"
                type="com.okay.viewdemo.databinding.ViewUtil" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            tools:context=".databinding.DatabindingActivity">

        <CheckBox
                android:id="@+id/cb"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:onCheckedChanged="@{(view,isChecked)->viewHandler.onCheckChanged(view,isChecked)}"
                app:layout_constraintLeft_toLeftOf="parent"
                app:layout_constraintTop_toBottomOf="@id/tv_name" />

        <Button
                android:id="@+id/btn"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:onLongClick="@{(view)->viewHandler.onLongClick(view,viewModel.name)}"
                android:text="Button"
                android:textAllCaps="false"
                app:layout_constraintLeft_toLeftOf="parent"
                app:layout_constraintTop_toBottomOf="@id/cb" />
    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>
```



## 在 xml 中对 `setXXX` 类型的方法的引用

在 `TextView`、`ImageView`等控件中，会存在类似于 `setText`、`setImageResource`这样设置数据源的方法，DataBinding 机制会自动将类中的 `setXXX`方法转换为 `XXX`，可以在 xml 中直接调用 `app:XXX=""`。方法 set 之后的名字首字母小写，类似：

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools">
  <data>
        <variable
                name="viewModel"
                type="com.okay.viewdemo.databinding.DataBindingViewModel" />
    </data>
	<ImageView
  	  android:layout_width="50dp"
    	android:layout_height="50dp"
    	app:imageResource="@{viewModel.imgResource}"
	/>
  <-- 也可以使用 android:imageResource = "@{viewModel.name}" -->

	<TextView
  	  android:layout_width="50dp"
    	android:layout_height="50dp"
    	android:text="@{viewModel.name}"/>
  <-- 也可以使用 app:text = "@{viewModel.name}" -->
          
</layout>
```

> `ImageView` 中的 `setImageResource` 方法被 `imageResource` 替代，`TextView` 中的 `setText` 方法被 `text` 替代。前面的限定符使用 `app` 还是 `android` 都可以。
>
> android:text="@{viewModel.name}"  <==>  app:text = "@{viewModel.name}"
>
> android:imageResource = "@{viewModel.name}"  <==>  app:imageResource="@{viewModel.imgResource}"  

## Import、Variable 和 include

### Import

通过 `import` 功能可以在布局文件中引用类，就像在 java 代码中一样，引入之后可以使用类中的属性?方法

```xml
...
<data>
    <import type="android.view.View"/>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List&lt;User>"/>
</data>
...
<TextView
       android:text="@{user.lastName}"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

#### 类型别名

当类名有冲突时，可以为导入的类设置别名，比如将 `com.okay.viewdemo.databinding` 包下的 `View` 重命名为 `Vista`

```xml
<import type="android.view.View"/>
    <import type="com.okay.viewdemo.databinding.View"
            alias="Vista"/>
    
```

这样在 xml 文件中 `View` 使用 `android.view.View`，`Vista` 引用 `com.okay.viewdemo.databinding`

> **注意**：Android Studio 尚不处理导入，因此导入变量的自动填充功能可能无法在您的 IDE 中使用。您的应用仍可以编译，并且您可以通过在变量定义中使用完全限定名称来解决这个 IDE 问题。

### Variable

您可以在 `data` 元素中使用多个 `variable` 元素。每个 `variable` 元素都描述了一个可以在布局上设置、并将在布局文件中的绑定表达式中使用的属性

```xml
...
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user" type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note" type="String"/>
</data>
...
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{user.name,default=`HelloWorld`}"
    android:textSize="24sp"
    app:layout_constraintLeft_toLeftOf="parent"
    app:layout_constraintTop_toTopOf="parent" />
          
```

如果想要变量是可观察的（可以动态更新），则变量类型应该声明为可观察类型。如果该变量是不实现 `Observable` 接口的基类或接口，则变量是`不可观察的`。

在生成的绑定类中，每个变量都会生成一个 `setter` 和一个 `getter` 方法，如果没有调用 `setter` 为其赋值，则这些变量使用默认值，如：引用类型采用 `null`，`int` 类型为 `0`，`boolean` 为 `false`。

如

```xml
android:text="@{user.name,default= `HelloWorld`}" 
// android:text='@{user.name,default= "HelloWorld"}'
```

所示，可以为表达式赋一个默认值，使用 `default` 表示，双引号里边用反单引号表示引用，如果用外层用单引号，内层使用双引号

### include

开发者可以向界面中引入的 `include` 标签透传数据，要在跟标签引入 `xmlns:app="http://schemas.android.com/apk/res-auto"`命名空间，透传的 key 是引入的布局中定义好的 `variable` 的 `name`。如下为 `model`。

`activity_databinding.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto"
        xmlns:tools="http://schemas.android.com/tools">
    <data>
        <variable
                name="viewModel"
                type="com.okay.viewdemo.databinding.DataBindingViewModel" />
    </data>
      
      <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            tools:context=".databinding.DatabindingActivity">
        <include
                layout="@layout/test"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintLeft_toLeftOf="parent"
                app:model="@{viewModel}" />
    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
  
```

`test.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">

    <data>

        <variable
                name="model"
                type="com.okay.viewdemo.databinding.DataBindingViewModel" />
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent">

        <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="@{model.name,default=`HelloWorld`}"
                android:textSize="24sp"
                app:layout_constraintLeft_toLeftOf="parent"
                app:layout_constraintTop_toTopOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

数据绑定不支持 include 作为 merge 元素的直接子元素。如下布局不受支持

```xml
<?xml version="1.0" encoding="utf-8"?>
    <layout xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:bind="http://schemas.android.com/apk/res-auto">
       <data>
           <variable name="user" type="com.example.User"/>
       </data>
       <merge><!-- Doesn't work -->
           <include layout="@layout/name"
               bind:user="@{user}"/>
           <include layout="@layout/contact"
               bind:user="@{user}"/>
       </merge>
    </layout>
```

## 实现 Observability

Data Binding 可以允许开发者使用可观察的对象绑定到 `attributes` 上，当数据改变的时候 UI 会自动更新。实现可观察有三种途径

1. 使用 Observable fields

2. 使用 LiveData

   与使用 Observable fields 相比，LiveData 提供了 [Transformations](https://developer.android.com/reference/android/arch/lifecycle/Transformations) 机制，并且可以与其他架构组件（Room、  WorkManager 等）兼容使用，同时还具有 [LiveData](https://developer.android.google.cn/topic/libraries/architecture/livedata#the_advantages_of_using_livedata) 自身的一些优点。使用时需要传递一个 `lifecycle owner` 实例给 Data binding 库

   ```kotlin
   binding.setLifecycleOwner(this)
   ```

3. 实现 Observable class

   要得到更大的灵活性和控制性，可以实现 `Observable` 接口（或者 BaseObservable），之后可以由开发者自己决定什么时候更新属性。这样可以在属性之间创建依赖关系，并且可以实现部分 UI 更新。见 [使用可观察的 ViewModel 更好的控制 Binding Adapter](#使用可观察的 ViewModel 更好的控制 Binding Adapter)

## 可观察的数据对象（Observable fields）

使用普通的 Java 对象可以实现数据绑定，但是修改对象不会自动是界面更新，要想让数据对象在其数据发生更改的时候通知其他对象，则需要使用`可观察的数据对象`。可观察类有三种类型：`对象、字段和集合`。

### 可观察字段

* `ObservableBoolean`
* `ObservableByte`
* `ObservableChar`
* `ObservableShort`
* `ObservableInt`
* `ObservableLong`
* `ObservableFloat`
* `ObservableDouble`
* `ObservableParcelable`

String 类型的变量包裹在 `ObservableField` 中，如下所示

要使用可观察的字段

1. 使用 `Java` 编写的话需要为字段声明 `public final` 属性  ```java

```java
private static class User{
		public final ObservableField<String> firstName = new ObservableField<>();
  	public final ObservableField<String> lastName = new ObservableField<>();
  	public final ObservableInt age = new ObservableInt();
}
```

2. 使用 `Kotlin` 编写的话需要创建为只读属性

```kotlin
class User{
		val firstName = ObservableField<String>()
		val lastName = ObservableField<String>()
    val age = ObservableInt()
}
```

要访问字段值，需要使用 `set()` 和 `get()` 方法

```kotlin
user.firstName = "John"
val age = user.age
```

> 使用 `Android Studio 3.1` 及更高版本允许跟 Android 架构组件 `LiveData` 对象结合使用，替换可观察字段。详情见下边所述。

### 可观察集合

#### 可观察 Map，ObservableArrayMap

```kotlin
ObservableArrayMap<String,Any>().apply{
		put("firstName","Xin")
  	put("lastName","Lin")
  	put("age",18)
}
```

```xml
<data>
        <import type="android.databinding.ObservableMap"/>
        <variable name="user" type="ObservableMap<String, Object>"/>
    </data>
    …
    <TextView
        android:text="@{user.lastName}"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <TextView
        android:text="@{String.valueOf(1 + (Integer)user.age)}"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
```

#### 可观察 List，ObservableArrayList

```kotlin
ObservableArrayList<Any>().apply {
		add("Google")
		add("Inc.")
		add(17)
}
```

在布局中可以通过索引访问

```xml
<data>
        <import type="android.databinding.ObservableList"/>
        <import type="com.example.my.app.Fields"/>
        <variable name="user" type="ObservableList<Object>"/>
    </data>
    …
    <TextView
        android:text='@{user[Fields.LAST_NAME]}'
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>
    <TextView
        android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"/>

// Fields.LAST_NAME = 0
// Fields.AGE = 2
```

### 可观察对象

实现 [`Observable`](https://developer.android.google.cn/reference/androidx/databinding/Observable) 接口的类可以注册属性监听器，以便接收可观察对象的属性更改的通知。

[`Observable`](https://developer.android.google.cn/reference/androidx/databinding/Observable) 接口具有添加和移除监听器的机制，但何时发送数据变化的通知则由开发者决定。为便于开发，数据绑定库提供了用于实现监听器注册机制的 [`BaseObservable`](https://developer.android.google.cn/reference/androidx/databinding/BaseObservable) 类。实现 `BaseObservable` 的数据类负责在属性更改时发出通知。具体操作过程是向 getter 分配 [`Bindable`](https://developer.android.google.cn/reference/androidx/databinding/Bindable) 注释，然后在 setter 中调用 [`notifyPropertyChanged()`](https://developer.android.google.cn/reference/androidx/databinding/BaseObservable#notifyPropertyChanged(int)) 方法，如以下示例所示：

```kotlin
class User : BaseObservable() {

        @get:Bindable
        var firstName: String = ""
            set(value) {
                field = value
                notifyPropertyChanged(BR.firstName)
            }

        @get:Bindable
        var lastName: String = ""
            set(value) {
                field = value
                notifyPropertyChanged(BR.lastName)
            }
    }
```

数据绑定在模块包中生成一个名为 `BR` 的类，该类包含用于数据绑定的`资源的 ID`，如上例中的 `firstName`、`lastName` 和 `age` 等。在编译期间，[`Bindable`](https://developer.android.google.cn/reference/androidx/databinding/Bindable) 注释会在 `BR` 类文件中生成一个条目。如果数据类的基类无法更改，则 [`Observable`](https://developer.android.google.cn/reference/androidx/databinding/Observable) 接口可以使用 [`PropertyChangeRegistry`](https://developer.android.google.cn/reference/androidx/databinding/PropertyChangeRegistry) 对象实现，以便有效地注册和通知监听器。

## 将布局视图绑定到架构组件（DataBinding 与 ViewModel 和 LiveData 的一起使用）

Data Binding 库可以与架构组件无缝使用来进一步简化 UI 的开发。app 中的 layout 可以绑定到架构组件中的 Data，帮助管理 UI controllers 的生命周期并且可以通知数据更改。

> 要使用 `LiveData` 替换 `observable fields`，需要 Android Studio 3.1 and higher

### 数据改变时使用 LiveData 通知 UI 

可以使用 `LiveData` 对象作为 data binding 的数据源，这样可以实现数据改变的时候自动通知 UI。

与实现 `Observable` 对象（例如 `ObservableInt 等`）不同的是，`LiveData` 对象知道订阅者对数据更改的生命周期，这可以带来很多使用 [LiveData](https://developer.android.google.cn/topic/libraries/architecture/livedata#the_advantages_of_using_livedata) 的一些自带好处。如，不会导致内存泄漏等。

要将 `LiveData` 对象与绑定类一起使用，你需要指定一个 `lifecycle owner` 来定义 `LiveData` 对象的 scope。如下所示：

```kotlin
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Inflate view and obtain an instance of the binding class.
        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // Specify the current activity as the lifecycle owner.
        binding.setLifecycleOwner(this)
    }
}
```

### 使用 ViewModel 管理与 UI 相关的数据

可以将 ViewModel、LiveData 与 Data Binding 一起使用，在 ViewModel 中负责处理与 UI 数据相关的逻辑，在 Activity 中实例化 ViewModel 并获取 DataBinding 对象，将 ViewModel 对象设置给 DataBinding 对象，在 layout 中就可以使用 viewMode 对象了。

```kotlin
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // Obtain the ViewModel component.
        val userModel: UserModel by viewModels()

        // Inflate view and obtain an instance of the binding class.
        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // Assign the component to a property in the binding class.
        binding.viewmodel = userModel
    }
}
```

在布局中使用 ViewModel 中提供的属性和表达式。

```xml
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@{viewmodel.rememberMe}"
    android:onCheckedChanged="@{() -> viewmodel.rememberMeChanged()}" />
```

### 使用可观察的 ViewModel 更好的控制 Binding Adapter

当数据发生变化的时候，开发者可以使用实现了 `Observable` 的 `ViewModel` 组件来通知 app 中的其他组件。

使用实现 `Observable` 的 `ViewModel` 可以让开发者更随意的控制 Binding Adapter。比如，可以在数据更改时的通知部分，开发者可以加入自己的逻辑，还可以指定自定义方法来设置双向数据绑定中的属性值。

创建一个类，继承 `ViewModel`，同时实现 `Observable` 接口。开发者可以在使用 `addOnPropertyChangedCallback()` 、`removeOnPropertyChangedCallback()` 提供观察器订阅或取消订阅通知时，加入一些自己的逻辑。还可以在 `notifyPropertyChanged()` 提供属性更改时加入一些自定义逻辑。如下所示，是一个可观察的 `ViewModel`：

```kotlin
/**
 * A ViewModel that is also an Observable,
 * to be used with the Data Binding Library.
 */
open class ObservableViewModel : ViewModel(), Observable {
    private val callbacks: PropertyChangeRegistry = PropertyChangeRegistry()

    override fun addOnPropertyChangedCallback(
            callback: Observable.OnPropertyChangedCallback) {
        callbacks.add(callback)
    }

    override fun removeOnPropertyChangedCallback(
            callback: Observable.OnPropertyChangedCallback) {
        callbacks.remove(callback)
    }

    /**
     * Notifies observers that all properties of this instance have changed.
     */
    fun notifyChange() {
        callbacks.notifyCallbacks(this, 0, null)
    }

    /**
     * Notifies observers that a specific property has changed. The getter for the
     * property that changes should be marked with the @Bindable annotation to
     * generate a field in the BR class to be used as the fieldId parameter.
     *
     * @param fieldId The generated BR id for the Bindable field.
     */
    fun notifyPropertyChanged(fieldId: Int) {
        callbacks.notifyCallbacks(this, fieldId, null)
    }
}
```

系统提供的 `BaseObservable`类就是一个简单的实现了 `Observable`接口的实现类

```kotlin
class ProfileObservableViewModel : ObservableViewModel() {
    val likes = ObservableInt(0)

    fun onLike() {
        likes.increment()
        notifyPropertyChanged(BR.popularity)
    }

    @Bindable
    fun getPopularity(): Popularity {
        return likes.get().let {
            when {
                it > 9 -> Popularity.STAR
                it > 4 -> Popularity.POPULAR
                else -> Popularity.NORMAL
            }
        }
    }
}
```

当调用了 `onLike()` 方法的时候，like 字段会发生变化，此时由于 `popularity` 属性与 like 属性有关联，所以 `popularity` 属性会被更新，又调用了 ` notifyPropertyChanged(BR.popularity)` 方法，所以更新会被其观察者接收到。

## 双向数据绑定

使用单向绑定的时候，开发者可以为一个 `attribute A` 设置 `value`，并且为另一个 `attribute B` 设置属性，来对 `A` 属性的值的变化做出反应（就是监听，当 A 属性发生变化的时候，在 `B` 中处理一些逻辑）。

## 生成的绑定类

### 创建绑定类

见[绑定数据部分](#绑定数据)

### 获取 Binding 中的 View

DataBinding 会为 xml 中的每个具有 id 的 View 生成一个对应 id 的对象，使用时如下：

```kotlin
...
val binding = DataBindingUtil.setContentView<Main>(
            this,
            R.layout.activity_databinding
        )
// get TextView who's id is tv
Log.i("textView",binding.tv)
```

### 即时绑定

当可变或可观察对象发生更改时，绑定会按照计划在下一帧之前发生更改。但有时必须立即执行绑定。要强制执行，请使用 [`executePendingBindings()`](https://developer.android.google.cn/reference/androidx/databinding/ViewDataBinding#executePendingBindings()) 方法。

### 高级绑定

当不知道具体的绑定类的时候，比如在任意布局的 `RecyclerView.Adapter` 中，不知道特定绑定类。在调用 `onBindViewHolder()` 时，需要指定绑定值。

如下所示，`RecyclerView` 绑定到的所有布局都有 `item` 变量。`BindingHolder` 对象具有一个 `getBinding()` 方法，这个方法返回 `ViewDataBinding` 基类。

```kotlin
 override fun onBindViewHolder(holder: BindingHolder, position: Int) {
      item: T = items.get(position)
   		// 所有布局文件中都存在数据变量 item
      // <variable name = "item" type = "xxx" />
      holder.binding.setVariable(BR.item, item);
      holder.binding.executePendingBindings();
}
```

### 自定义绑定名称

默认情况下，绑定类是根据布局文件的名称生成的，以大写字母开头，移除下划线 ( _ )，将后一个字母大写，最后添加后缀 **Binding**。该类位于模块包下的 `databinding` 包中。例如，布局文件 `contact_item.xml` 会生成 `ContactItemBinding` 类。如果模块包是 `com.example.my.app`，则绑定类放在 `com.example.my.app.databinding` 包中。

通过调整 `data` 元素的 `class` 特性，绑定类可重命名或放置在不同的包中。

```xml
<data class = "MainPage">
  ...
</data>
```

则生成的绑定类名字为 `MainPage`

若在类前面加 `.`则在 App 包下生成绑定类

```xml
<data class = ".MainPage">
  ...
</data>
```

则生成的绑定类在 `com.okay.viewdemo` 下

还可以使用完整软件包名来生成绑定类，生成的绑定类在指定的包下边

```xml
<data class = "com.okay.viewdemo.text.MainPage">
               ...
</data>
```

则生成的绑定类在 `com.okay.viewdemo.text` 下，名字为 `MainPage`

![image-20200420171204887](https://tva1.sinaimg.cn/large/007S8ZIlly1ge0ckp29n1j30fk0ofaca.jpg)

## 绑定适配器

Binding adapers 帮助框架调用 `setxxx` 方法为属性设置值，类似 `setText()` 和 `setOnClickListener()` 方法。同时它允许你自定义/创建布局属性，在自定义的方法中提供绑定逻辑，返回特定类型的值。使用 `Binding Adapter` 可以把 `Activity` 中对 UI 的操作转换为一个个的静态方法，有利于封装。

> 在 `androidx.databinding:databinding-adapters:versionCode` aar 包中， `androidx.databinding.adapters` 包下，有系统预设的一些 Adapter 的实现。可供参考。

### 设定属性值（set attribute）

无论何时绑定的 `value` 发生变化的时候，生成的 Binding class 都应该通过绑定表达式调用一个 `setter` 方法更新 view 对应的状态。

1. 让 Data Binding 库决定调用哪个方法
2. 通过指定一个自定义的方法名显示声明方法
3. 使用 Binding Adapter 实现自定义逻辑来选择一个方法

#### 自动选择方法

假设有个属性叫 `test`，Data Binding 库会自动寻找名为 `setTest(arg)` 的，参数类型为表达式计算之后的返回结果类型的方法，属性的命名空间无所谓，只根据属性名、参数类型寻找方法。

```xml
app:test="@{10}"  <==>  android:test="@{10}"
```

则会寻找 `setTest(Int int)` 的 方法。

即使在 xml 中某个控件下边不存在对应的 `attribute` 属性，Data binding 同样可以正常绑定数据。开发者可以使用 Data binding 为任何 setter 方法创建属性。比如， `DrawerLayout` 中的 `setScrimColor(int)` 和 `setDrawerListneer(DrawerListener)` 方法，可以如下使用

```xml
<DrawerLayout
    app:scrimColor="@{@color/scrim}"
    app:drawerListener="@{fragment.drawerListener}"
/>
```

#### 通过指定一个自定义的方法名

这个注解的作用，是将指定的 attrubute 关联到某个类的某个方法上，如下代码所示，将 `android:tint` 属性关联到了 `ImageView` 的 `setImageTintList` 方法上。

```kotlin
@BindingMethods(
    BindingMethod(
        type = android.widget.ImageView::class,
        attribute = "android:tint",
        method = "setImageTintList"
    )
)
class MyBindingMethods
```

> 大多数时候，开发者不需要重命名 Android Framework 中提供的类中的 `setter` 方法。Data binding 会根据 attribute 对应的转换规则去寻找匹配的 `setter` 方法。而且我自己实验时候也没有见到应有的效果。

#### :star:**使用 Binding Adapter 实现自定义逻辑**

> 要使用 `@BindingAdapter` 注解，需要在 App 的 `build.gradle` 中配置使用 kotlin-kapt 编译 `apply plugin: 'kotlin-kapt'`

利用 Binding adapter，开发者可以在方法中实现各种逻辑，是非常方便的。比如，`android:paddingLeft` 属性没有对应的 `setter`，系统提供了 `setPadding(left,top,right,bottom)` 方法。此时使用 `BindingAdapter` 注解，就可以实现为一个 `view` 的属性实现自定义的设置逻辑。

> @BindingAdapter 注解必须设置在静态方法上

Data binding 库为 `android:paddingLeft` 方法创建了 `BindingAdapter` 注解，在 `androidx.databinding.adapters.ViewBindingAdapter` 类中

```java
@BindingAdapter({"android:paddingLeft"})
public static void setPaddingLeft(View view, float paddingFloat) {
    final int padding = pixelsToDimensionPixelSize(paddingFloat);
    view.setPadding(padding, view.getPaddingTop(), view.getPaddingRight(),
            view.getPaddingBottom());
}
```

> 第一个参数决定了与 `attribute` 关联的 `View` 的类型，第二个参数决定了 `attribute` 中设置的 binding 表达式计算后返回值类型。
>
> :star:注意：当开发者自定义的 `Binding adapter` 方法 与系统定义的发生了冲突时，开发者定义的方法会覆盖系统的方法。

##### 接收多个参数的 Binding adapter 方法

```kotlin
@JvmStatic
@BindingAdapter(value = ["name", "age"], requireAll = false)
// 下面是省略 value key 的写法
//@BindingAdapter("name", "age", requireAll = false)
fun test(view: View, name: String?, age: Int?) {
    Log.i(TAG, "test | [view, name, age]：[$view,$name,$age]")
}
```

多个参数在 xml 文件中对应为多个属性，如上定义了 `name`、`age` 两个参数，则在 xml 中作为两个 `attribute` 使用

```xml
<TextView
		app:age="@{viewModel.age}"
		app:name="@{viewModel.name}"
/>
```

`BindingAdapter` 中 `value` 参数的顺序，对应方法中参数的顺序。`requireAll` 字段默认为 `true`，此时表示必须两个参数在 xml 中都设置了才可以编译运行，当 `requireAll` 为 `false` 时，则可以值设置`参数中的某一个`。

##### 在 Binding adapter 方法中持有 old value

这个说的是自定义的 Binding adapter 方法中，可以同时持有 oldValue 和 newValue，但是需要先为 `attribute` 声明一个 old value。

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:paddingLeft="@{viewModel.age??10}"
/>
```

```kotlin
/**
  * 自定义 adapter 可以选择性的处理旧值
  * 必须先声明一个旧值，然后再声明新值
  * 回调到方法里的时候，会得到两个值，一个是之前设置的旧值，一个是后来设置的新值
*/
@JvmStatic
@BindingAdapter("android:paddingLeft")
fun setPaddingLeft(view: View, oldPaddingleft: Int?, newPaddingLeft: Int?) {
		Log.i(TAG,"setPaddingLeft | [view, oldPaddingleft, newPaddingLeft]：[$oldPaddingleft,$newPaddingLeft]")
		if (oldPaddingleft != newPaddingLeft) {
    		Toast.makeText(view.context,"newPaddingLeft is:$newPaddingLeft",
                    Toast.LENGTH_SHORT).show()
       }
}
```

##### 使用 Binding Adapter 处理 Event

Binding Adapter 在事件处理上默认支持接口或者抽象类中只有一个抽象方法的情况，如下系统自带的

```java
@BindingAdapter({"android:onClick", "android:clickable"})
public static void setOnClick(View view, View.OnClickListener clickListener,
        boolean clickable) {
    view.setOnClickListener(clickListener);
    view.setClickable(clickable);
}
```

在 layout 中使用 onClick 事件

```xml
<Button android:onClick="@{()->handler.onClick()}" />
```

在官方的介绍中，就可以调用到 `handler` 中对应的 `onClick()` 方法，且默认有一个参数 `View(Button)`。不过经过我的试验，这样写会报错，找不到 `onClick()` 方法。

```
activity_databinding.xml:87:44: cannot find method onClick() in class com.okay.viewdemo.databinding.BindAdapter
```

> `实测需要将所有参数补齐，才可以通过编译并且调用到正确的方法`

```xml
<-- 参数名无所谓，前后一致即可 -->
<Button android:onClick="@{(view)->handler.onClick(view)}" />
```

还可以在表达式中传递函数的引用来调用对应方法，`此时不需要传递 view 参数`

```xml
<Button android:onClick="@{handler::click}" />
```

如下 `onLayoutChange` 方法

```kotlin
@BindingAdapter("android:onLayoutChange")
fun setOnLayoutChangeListener(
        view: View,
        oldValue: View.OnLayoutChangeListener?,
        newValue: View.OnLayoutChangeListener?
) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        if (oldValue != null) {
            view.removeOnLayoutChangeListener(oldValue)
        }
        if (newValue != null) {
            view.addOnLayoutChangeListener(newValue)
        }
    }
}
```

在 layout 中使用，需要将参数按顺序传递进去

```xml
<View android:onLayoutChange="@{(view,left,top,right,bottom,oldLeft,oldTop,oldRight,oldBottom)->bindAdapter.onLayoutChange(view,left,top,right,bottom,oldLeft,oldTop,oldRight,oldBottom)}"/>
```

还可以为 `attribute` 的值传递一个 `OnLayoutChangeListener` 对象

```kotlin
val mLayoutChanged =
        View.OnLayoutChangeListener { v, left, top, right, bottom, oldLeft, oldTop, oldRight, oldBottom ->
            Log.i(
                TAG,
                "onLayoutChanged | [v, left, top, right, bottom, oldLeft, oldTop, oldRight, oldBottom]：[$v,$left,$top,$right,$bottom,$oldLeft,$oldTop,$oldRight,$oldBottom]"
            )
        }
```

在 `layout` 中如下使用

```xml
<View android:onLayoutChange="@{bindAdapter.mLayoutChanged}"/>
```

###### 含有多个方法的 listener 

当一个 `listener` 含有多个方法的时候，处理方法其实就是多个参数的 `Binding adapter`，需要将多个方法拆分成多个 `listener` 。比如，Android 在  `View.OnAttachStateChangeListener` 接口中定义了两个抽象方法：`onViewAttachedToWindow(View)` 和 `onViewDetachedFromWindow(View)`。要想在 layout 中使用 `OnAttachStateChangeListener`，则需要将接口中的两个方法抽象成另外两个独立的接口，接口中提供唯一方法来处理逻辑：

```kotlin
// Translation from provided interfaces in Java:
@TargetApi(Build.VERSION_CODES.HONEYCOMB_MR1)
interface OnViewDetachedFromWindow {
    fun onViewDetachedFromWindow(v: View)
}

@TargetApi(Build.VERSION_CODES.HONEYCOMB_MR1)
interface OnViewAttachedToWindow {
    fun onViewAttachedToWindow(v: View)
}
```

因为有两个参数（listener 方法），所以需要一个可以同时应用两个方法或只应用其中一个方法的适配器，将 `requireAll` 字段设置为 `false`，则

```kotlin
@JvmStatic
@BindingAdapter(
    "android:onViewAttachedToWindow",
    "android:onViewDetachedFromWindow",
    requireAll = false
)
fun setListener(
    view: View,
    onViewAttachedToWindow: ViewBindingAdapter.OnViewAttachedToWindow?,
    onViewDetachedFromWindow: ViewBindingAdapter.OnViewDetachedFromWindow?
) {
		if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB_MR1) {
            val newListener: View.OnAttachStateChangeListener?
            newListener = if (detach == null && attach == null) {
                null
            } else {
                object : View.OnAttachStateChangeListener {
                    override fun onViewAttachedToWindow(v: View) {
                        attach.onViewAttachedToWindow(v)
                    }

                    override fun onViewDetachedFromWindow(v: View) {
                        detach.onViewDetachedFromWindow(v)
                    }
                }
            }

            val oldListener: View.OnAttachStateChangeListener? =
                    ListenerUtil.trackListener(view, newListener, R.id.onAttachStateChangeListener)
            if (oldListener != null) {
                view.removeOnAttachStateChangeListener(oldListener)
            }
            if (newListener != null) {
                view.addOnAttachStateChangeListener(newListener)
            }
}
```

`android.databinding.adapters.ListenerUtil` 类可以帮助追踪以前的 `Listener`，以便可以在 binding adapter 中将其 remove 掉。

通过用 `@TargetApi(VERSION_CODES.HONEYCOMB_MR1)` 注释接口 `OnViewDetachedFromWindow` 和 `OnViewAttachedToWindow`，数据绑定代码生成器知道只应在运行 Android 3.1（API 级别 12）及更高级别（`addOnAttachStateChangeListener()` 方法支持的相同版本）时生成监听器。

使用的时候传递一个参数类型的实例对象 `ViewBindingAdapter.OnViewAttachedToWindow`/`ViewBindingAdapter.OnViewDetachedFromWindow`或者其对应的方法

声明参数类型的对象：

```kotlin
val mAttach =
    ViewBindingAdapter.OnViewAttachedToWindow { v ->
        Log.i(
            TAG,
            "onViewAttachedToWindow | [v]：[$v]"
        )
    }
```

在 layout 中使用：

```xml
<View android:onViewAttachedToWindow="@{bindAdapter.mAttach}"/>
```

或者声明一个参数类型对应的方法

```kotlin
fun onViewAttachedToWindow(view: View) {
    Log.i(
        TAG,
        "onViewAttachedToWindow | [v]：[$view]"
    )
}
```

在 layout 中使用

```xml
<View android:onViewAttachedToWindow="@{bindAdapter::onViewAttachedToWindow}"/>
```

如果有多个参数，则分别将参数声明在调用中

```xml
<View android:onViewAttachedToWindow="@{(view,param1,param2)->bindAdapter.onViewAttachedToWindow(view,param1,param2)}/>
```

### 对象转换

#### 自动转换

当一个 binding 表达式返回了一个 `Object` 的时候，binding 库会自动选择一个方法来设置属性的 value。表达式返回的 `Object`会被转换为选择的方法的参数类型。对于使用 [`ObservableMap`](https://developer.android.google.cn/reference/androidx/databinding/ObservableMap) 类存储数据的应用，这种行为非常便捷。

```xml
<TextView
   android:text='@{userMap["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content" />
```

> 可以使用 ` object.key` 的方式访问 Map 中的值。如下所示：
>
> @{userMap["lastName"]}  <==>  @{userMap.lastName}

表达式中的 `userMap` 对象会返回一个值，该值会自动转换为用于设置 `android:text` 特性值的 `setText(CharSequence)` 方法中的参数类型。如果参数类型不明确，则必须`在表达式中强制转换返回类型。`

#### 自定义转换（`BindingConversion`）

有些情况下自动转换不能满足需求，此时可以自定义转换，比如 `android:background` 属性接收一个 `Drawable` 对象作为参数，但当我们想用一个 `color` 值（`integer`）作为参数的时候，就需要做一个转换。

```xml
<View
   android:background="@{isError ? @color/red : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

**可以通过为一个 public 的静态方法添加 `@BindingConversion` 注解来实现**

当满足如下条件时，Convert 会自动调用：

> 表达式参数类型需要的不是 `integer`，同时属性中 value 传入的表达式返回类型是 `integer`的时候。下面的 Converter 函数会自动调用，将这个 `int` 值转换为 `ColorDrawable`。

```kotlin
// convertColorToDrawable 函数定义在 androidx.databinding.adapters.Converters类中。
@BindingConversion
fun convertColorToDrawable(color: Int) = ColorDrawable(color)
```

上述 `View` 中表达式返回的参数类型需要一致，三目运算符返回的两种情况参数类型需要一致。如下这种情况不可以：

```xml
<View
   android:background="@{isError ? @drawable/error : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

将 `Boolean` 值转换为对 View 可见性的值

```kotlin
@BindingConversion
@JvmStatic fun booleanToVisibility(isVisible: Boolean): Int {  
    return if (isVisible) View.VISIBLE else View.GONE
}
```

> :star:注意：Converter 函数的调用是不安全的，它不会被限定为只有某个方法才可以调用，它可能会在任何符合条件的地方被调用，如上所示的 `booleanToVisibility` 方法，如果在 xml 的 `attribute` 接收的参数类型不为 `boolean` 的属性中传入了 `boolean` 值，则改转换也会执行。比如在 `TextView` 中如下使用： 
>
> ```xml
> <TextView
> 		android:width="200dp"
> 		android:height="100dp"
> 		android:text="@{true}"
> 		/>
> ```
>
> 则该 Conversion 同样会被调用
>
> 推荐使用 `Binding Adapter`来在自定义属性中实现这些逻辑，而不是在 xml 表达式中添加一些逻辑，比如三目运算符之类的。

## 参考文章

[Android DataBinding](https://developer.android.google.cn/topic/libraries/data-binding)