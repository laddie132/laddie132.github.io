---
title: "Spark程序主函数区别"
date: "2017-12-04 23:13:23"
categories:
- Code

tags: 
- spark
---

在scala程序中，主函数有两种启动方式：
``` scala
object Test extends App {
  //ToDo
}
 
object Test{
  def main(args: Array[String]): Unit = {
    //ToDo
  }
}
```
<!-- more -->
其中第一种方法在Spark中不被兼容，可能会出现空指针异常，如下代码：

```Scala
object Test extends App{
  val conf = new SparkConf()
                .setAppName("Test")
                .set("spark.sql.tungsten.enabled", "true")

  val sc = new SparkContext(conf)
  
  val data = sc.parallelize(List(1, 2, 3, 4), 4)
  val test_print = 111
  val test_print2 = List("this", "is")
  data.foreach(_ =>{
    println("test1:", test_print)
    println("test2:", test_print2)
  })
}
```

运行后，对于第二个print出现如下空指针异常错误`caused by: java.lang.NullPointerException`，且第一个print输出始终为0。

这是由于继承App后，里面所有变量都被认为是单个类的field了，并不会被初始化，无法被Spark访问到；而如果是以main函数初始化，所有变量都是被当作局部变量，这样一来，Spark程序封装闭包后便可以访问到该变量。

另外，Spark官方文档也有如下提示：

> Note that applications should define a `main()` method instead of extending `scala.App`.Subclasses of `scala.App` may not work correctly.











