---

layout: post

title: 使用 Kotlin 进行 Android 开发

date: 2018-05-11 08:30:00.000000000 +09:00

tags: Android Kotlin

---

[TOC]

# Kotlin

**Kotlin**是一种在Java虚拟机上运行的静态类型编程语言，它也可以被编译成为JavaScript源代码。它主要是由俄罗斯圣彼得堡的JetBrains开发团队所发展出来的编程语言，其名称来自于圣彼得堡附近的科特林岛。2012年1月，著名期刊《Dr. Dobb's Journal》中Kotlin被认定为该月的最佳语言。虽然与Java语法并不兼容，但Kotlin被设计成可以和Java代码相互运作，并可以重复使用如Java集合框架等的现有Java类库。

> 摘自 [Kotlin维基百科](https://zh.wikipedia.org/wiki/Kotlin)

# 历史

2011年7月，JetBrains推出Kotlin项目，这是一个面向JVM的新语言，它已被开发一年之久。JetBrains负责人Dmitry Jemerov说，大多数语言没有他们正在寻找的特性，Scala除外。但是，他指出了Scala的编译时间慢这一明显缺陷。Kotlin的既定目标之一是像Java一样快速编译。2012年2月，JetBrains以Apache 2许可证开源此项目。

Jetbrains希望这个新语言能够推动IntelliJ IDEA的销售。

Kotlin v1.0于2016年2月15日发布。这被认为是第一个官方稳定版本，并且JetBrains已准备从该版本开始的长期向后兼容性。

在Google I/O 2017中，Google宣布在Android上为Kotlin提供最佳支持。

> 摘自 [Kotlin维基百科](https://zh.wikipedia.org/wiki/Kotlin)

# 使用 Kotlin 进行 Android 开发

Kotlin 非常适合开发 Android 应用程序，将现代语言的所有优势带入 Android 平台而不会引入任何新的限制：

- **兼容性**：Kotlin 与 JDK 6 完全兼容，保障了 Kotlin 应用程序可以在较旧的 Android 设备上运行而无任何问题。Kotlin 工具在 Android Studio 中会完全支持，并且兼容 Android 构建系统。
- **性能**：由于非常相似的字节码结构，Kotlin 应用程序的运行速度与 Java 类似。 随着 Kotlin 对内联函数的支持，使用 lambda 表达式的代码通常比用 Java 写的代码运行得更快。
- **互操作性**：Kotlin 可与 Java 进行 100％ 的互操作，允许在 Kotlin 应用程序中使用所有现有的 Android 库 。这包括注解处理，所以数据绑定和 Dagger 也是一样。
- **占用**：Kotlin 具有非常紧凑的运行时库，可以通过使用 ProGuard 进一步减少。 在实际应用程序中，Kotlin 运行时只增加几百个方法以及 .apk 文件不到 100K 大小。
- **编译时长**：Kotlin 支持高效的增量编译，所以对于清理构建会有额外的开销，增量构建通常与 Java 一样快或者更快。
- **学习曲线**：对于 Java 开发人员，Kotlin 入门很容易。包含在 Kotlin 插件中的自动 Java 到 Kotlin 的转换器有助于迈出第一步。Kotlin 心印通过一系列互动练习提供了语言主要功能的指南。

# 语法

## 定义包

包的声明应处于源文件顶部：

```Kotlin
package my.demo

import java.util.*

// ……
```



## 函数

```Kotlin
// 带有两个 Int 参数、返回 Int 的函数
fun sum(a: Int, b: Int): Int {
    return a + b
}

// 将表达式作为函数体、返回值类型自动推断的函数
fun sum(a: Int, b: Int) = a + b

// 函数返回无意义的值
fun printSum(a: Int, b: Int): Unit {
    println("sum of $a and $b is ${a + b}")
}

// Unit 返回类型可以省略
fun printSum(a: Int, b: Int) {
    println("sum of $a and $b is ${a + b}")
}
```

## 变量

```kotlin
// 使用 val 关键字定义的为常量
val a: Int = 1  // 立即赋值
val b = 2   	// 自动推断出'Int'类型
val c: Int  	// 如果没有初始值类型不能省略
c = 3       	// 明确赋值

// 使用 var 关键字定义的为变量
var c = 5		// 定义一个'Int'类型值为5
c = 3			// 可修改值为3
```

## 注释

正如 Java 和 JavaScript，Kotlin 支持行注释及块注释

> // 这是一个行注释
>
>  /* 这是一个多行的块注释。 */

与 Java 不同的是，Kotlin 的块注释可以嵌套。

## 使用字符串模板

```kotlin
var a = 1
// 模板中的简单名称：
val s1 = "a is $a" 

a = 2
// 模板中的任意表达式：
val s2 = "${s1.replace("is", "was")}, but now is $a"
```

## 使用条件表达式

```kotlin
fun maxOf(a: Int, b: Int): Int {
    if (a > b) {
        return a
    } else {
        return b
    }
}

// 等同于
fun maxOf(a: Int, b: Int) = if (a > b) a else b
```

## 使用可空值及`null`检测

当某个变量的值可以为 *null* 的时候，必须在声明处的类型后添加 `?` 来标识该引用可为空。

```Kotlin
// 变量空值检测，这样在使用的时候就必须要判断是否为空
private var a: String? = null

// 如果有一些变量局部设置，在onCreat()中初始化
private lateinit var b: String

// 如果 str 的内容不是数字返回 null
fun parseInt(str: String): Int? {
    // ……
}
```

## 使用类型检测及自动类型转换

*is* 运算符检测一个表达式是否某类型的一个实例。

如果一个不可变的局部变量或属性已经判断出为某类型，那么检测后的分支中可以直接当作该类型使用，无需显式转换。

```Kotlin
fun getStringLength(obj: Any): Int? {
    if (obj is String) {
        // 'obj'在该条件分支内自动转换成'String'
        return obj.length
    }
    // 在离开类型检测分支后，'obj'仍然是'Any'类型
    return null
}

// 或者
fun getStringLength(obj: Any): Int? {
    if (obj !is String) return null
    // 'obj'在这一分支自动转换为'String'
    return obj.length
}

// 甚至
fun getStringLength(obj: Any): Int? {
    // 'obj'在 '&&' 右边自动转换成'String'类型
    if (obj is String && obj.length > 0) {
        return obj.length
    }
    return null
}
```

## 使用 `for` 循环

```Kotlin
val items = listOf("apple", "banana", "kiwifruit")
for (item in items) {
    println(item)
}

// 或者
val items = listOf("apple", "banana", "kiwifruit")
for (index in items.indices) {
    println("item at $index is ${items[index]}")
}
```

## 使用 `when` 表达式

```Kotlin
fun describe(obj: Any): String =
when (obj) {
    1          -> "One"
    "Hello"    -> "Greeting"
    is Long    -> "Long"
    !is String -> "Not a string"
    else       -> "Unknown"
}

/* result输出：
item at 0 is apple
item at 1 is banana
item at 2 is kiwifruit
*/
```

## 使用区间`range`

使用 *in* 运算符来检测某个数字是否在指定区间内

```Kotlin
val x = 10
val y = 9
if (x in 1..y+1) {
    println("fits in range")
}

// result输出：
// fits in range
```

> 如果想检测某个数字是否在指定区间外：可在`in`前面加上`!`

## 使用集合

```Kotlin
// 对集合进行迭代
val items = listOf("apple", "banana", "kiwifruit")
for (item in items) {
    println(item)
}
/* 输出：
apple
banana
kiwifruit
*/

// 使用 in 运算符来判断集合内是否包含某实例
val items = listOf("apple", "banana", "kiwifruit")
when {
    "orange" in items -> println("juicy")
    "apple" in items -> println("apple is fine too")
}
/* 输出：
apple is fine too
*/

// 使用 lambda 表达式来过滤（filter）和映射（map）集合
val fruits = listOf("banana", "avocado", "apple", "kiwifruit")
items
.filter { it.startsWith("a") }
.sortedBy { it }
.map { it.toUpperCase() }
.forEach { println(it) }
/* 输出：
APPLE
AVOCADO
*/
```

## 创建基本类及其实例

```Kotlin
val rectangle = Rectangle(5.0, 2.0) // 不需要“new”关键字
val triangle = Triangle(3.0, 4.0, 5.0)
```

## 创建 `DTOs`

```Kotlin
data class Customer(val name: String, val email: String)
```

会为 `Customer` 类提供以下功能：

- 所有属性的 `getters` （对于 *var* 定义的还有 setters）
- `equals()`
- `hashCode()`
- `toString()`
- `copy()`
- 所有属性的 `component1()`、 `component2()`……等等
