通过反射可以访问或操作`类的私有变量和方法`

`setAccessible()` 获取方法/变量的访问权（private 修饰的方法和字段）



#### getDeclaredMethod

返回类中的所有自身声明方法，包含 public、protected 和 `private` 方法。

#### getMethod

获取类的所有`公共方法`，包括自身的所有 `public` 方法和从基类继承的、从接口实现的所有 `public` 方法

#### class#newInstance() /Constructor#newInstance()方法 

此方法会调用 Class 的默认无参构造函数，`若函数声明了构造函数的重载，则必须手动声明无参构造函数，否则报错`

#### Field

```c
field.getType() // 获取 field 类型
field.getType().getSimpleName() // 获取 field 类型名
Modifier.toString(field.getModifiers()) // 获取 field 修饰符
Field#setInt(Object,Int) // 为指定 object 对象的 field设置 int 值
Field#getInt(Object) // 获取指定 object 对象的指定 field 的值
```

#### Method

* method.getParameterTypes() // 获取一个 Class<?>[] 数组，存储该  method 的参数类型
* method.getReturnType()
* method.getParameterCount()
* method.getParameters() // 获取此对象代表的底层可执行文件需要的所有参数信息