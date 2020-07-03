[TOC]

## 遇到的问题

### extern 关键字的使用

  > extern 关键字声明一个外部变量/函数。
  >
  > ```c
  > int add(){
  >   // 函数内声明变量 x 和 y 为外部变量
  >   extern int x;
  >   extern int y;
  >   // 给外部变量(全局变量) x 和 y 赋值
  >   // 如下这样写会报错，等于在函数内重新声明并定义了 x、y。
  >   // extern 声明的变量必须在全局实例化，或者像下边这样写 x = 1
  >   // int x = 1;
  >   // int y = 2;
  >   x = 1;
  >   y = 2;
  >   return x+y;
  > }
  > ```
  >
  > 

### 两个文件的链接(一般使用头文件 `.h`，还有一种是使用 `CMakeLists.txt` 配置)

![](https://ws1.sinaimg.cn/large/006tNc79ly1g2cl781sbyj30x00bqjt6.jpg)

其中，`add_executable()`方法里将各个文件链接起来了，编译器为我们完成。

C 语言规定，定义函数时，省略 `extern` 关键字，则默认为外部函数。在需要调用此函数的其他文件中，需要对此函数做声明。

### 不同文件中定义相同的全局变量/常量

在学习过程中遇到一个问题，在 `.h` 文件中定义了一个常量 `const int WIDTH = 100`。然后在两个文件中都引用了这个 `.h` 文件，结果报错了，`duplicate symbol 声明`。查找了原因，发现是在头文件中定义了全局常量引起的。

> C 语言中，不允许在不同类中定义相同名字的**「全局变量/常量」**



[C/C++ 中基本数据类型所占内存大小](https://blog.csdn.net/zcyzsy/article/details/77935651)





Clion 怎么查看具体错误信息：

1. 数组越界无提示（exit code 6、11）
2. 函数无返回值无提示