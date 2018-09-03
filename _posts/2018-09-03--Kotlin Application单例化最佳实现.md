---
layout: post

title: Application单例化和属性的Delegated

date: 2018-09-03 08:00:00.000000000 +09:00

tags: Android Kotlin

---
[TOC]

我们很快要去实现一个数据库，如果我们想要保持我们代码的简洁性和层次性（而不是把所有代码添加到Activity中），我们就要需要有一个更简单的访问application context的方式。

# java方式实现
按照我们在Java中一样创建一个单例最简单的方式：

```kotlin
class App : Application() {
    companion object {
        private var instance: Application? = null
        fun instance() = instance!!
    }
    override fun onCreate() {
        super.onCreate()
        instance = this
    }
}
```

为了这个Application实例被使用，要记得在AndroidManifest.xml中增加这个App：

```kotlin
<application
        android:name=".App"
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
    ...
</application>
```

Android有一个问题，就是我们不能去控制很多类的构造函数。比如，我们不能初始化一个非null属性，因为它的值需要在构造函数中去定义。所以我们需要一个可null的变量，和一个返回非null值的函数。我们知道我们一直都有一个 **App** 实例，但是在它调用 **onCreate** 之前我们不能去操作任何事情，所以我们为了安全性，我们假设 **instance()** 函数将会总是返回一个非null的 **app** 实例。

但是这个方案看起来有点不自然。我们需要定义个一个属性（已经有了getter和setter），然后通过一个函数来返回那个属性。我们有其他方法去达到相似的效果么？是的，我们可以通过委托这个属性的值给另外一个类。这个就是我们知道的 **委托属性** 。

# 委托属性

我们可能需要一个属性具有一些相同的行为，使用 **lazy** 或者 **observable** 可以被很有趣地实现重用。而不是一次又一次地去声明那些相同的代码，Kotlin提供了一个委托属性到一个类的方法。这就是我们知道的委托属性。

当我们使用属性的get或者set的时候，属性委托的**getValue**和**setValue**就会被调用。
属性委托的结构如下：

```kotlin
class Delegate<T> : ReadWriteProperty<Any?, T> {
    fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return ...
    }

    fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        ...
    }
}
```

这个T是委托属性的类型。**getValue**函数接收一个类的引用和一个属性的元数据。**setValue**函数又接收了一个被设置的值。如果这个属性是不可修改（val），就会只有一个**getValue**函数。

下面展示属性委托是怎么设置的：

```kotlin
class Example {
    var p: String by Delegate()
}
```

它使用了 **by** 这个关键字来指定一个委托对象。

# 标准委托

在Kotlin的标准库中有一系列的标准委托。它们包括了大部分有用的委托，但是我们也可以创建我们自己的委托。

## Lazy

它包含一个lambda，当第一次执行**getValue**的时候这个lambda会被调用，所以这个属性可以被延迟初始化。之后的调用都只会返回同一个值。这是非常有趣的特性， 当我们在它们第一次真正调用之前不是必须需要它们的时候。我们可以节省内存，在这些属性真正需要前不进行初始化。

```kotlin
class App : Application() {
    val database: SQLiteOpenHelper by lazy {
        MyDatabaseHelper(applicationContext)
    }

    override fun onCreate() {
        super.onCreate()
        val db = database.writableDatabase
    }
}
```

在这个例子中，database并没有被真正初始化，直到第一次调用**onCreate**时。在那之后，我们才确保applicationContext存在，并且已经准备好可以被使用了。**lazy**操作符是线程安全的。

如果你不担心多线程问题或者想提高更多的性能，你也可以使用**lazy(LazyThreadSafeMode.NONE){ ... }**。

## Observable

这个委托会帮我们监测我们希望观察的属性的变化。当被观察属性的**set**方法被调用的时候，它就会自动执行我们指定的lambda表达式。所以一旦该属性被赋了新的值，我们就会接收到被委托的属性、旧值和新值。

```kotlin
class ViewModel(val db: MyDatabase) {
    var myProperty by Delegates.observable("") {
        d, old, new ->
        db.saveChanges(this, new)
    }
}
```

这个例子展示了，一些我们需要关心的ViewMode，每次值被修改了，就会保存它们到数据库。

## Vetoable

这是一个特殊的**observable**，它让你决定是否这个值需要被保存。它可以被用于在真正保存之前进行一些条件判断。

```kotlin
var positiveNumber = Delegates.vetoable(0) {
    d, old, new ->
    new >= 0
}
```

上面这个委托只允许在新的值是正数的时候执行保存。在lambda中，最后一行表示返回值。你不需要使用return关键字（实质上不能被编译）。

## Not Null

有时候我们需要在某些地方初始化这个属性，但是我们不能在构造函数中确定，或者我们不能在构造函数中做任何事情。第二种情况在Android中很常见：在Activity、fragment、service、receivers……无论如何，一个非抽象的属性在构造函数执行完之前需要被赋值。为了给这些属性赋值，我们无法让它一直等待到我们希望给它赋值的时候。我们至少有两种选择方案。

第一种就是使用可null类型并且赋值为null，直到我们有了真正想赋的值。但是我们就需要在每个地方不管是否是null都要去检查。如果我们确定这个属性在任何我们使用的时候都不会是null，这可能会使得我们要编写一些必要的代码了。

第二种选择是使用**notNull**委托。它会含有一个可null的变量并会在我们设置这个属性的时候分配一个真实的值。如果这个值在被获取之前没有被分配，它就会抛出一个异常。

这个在单例App这个例子中很有用：

```kotlin
class App : Application() {
    companion object {
        var instance: App by Delegates.notNull()
    }

    override fun onCreate() {
        super.onCreate()
        instance = this
    }
}
```

## 从Map中映射值

另外一种属性委托方式就是，属性的值会从一个map中获取value，属性的名字对应这个map中的key。这个委托可以让我们做一些很强大的事情，因为我们可以很简单地从一个动态地map中创建一个对象实例。

如果我们 **import kotlin.properties.getValue**，我们可以从构造函数映射到val属性来得到一个不可修改的map。如果我们想去修改map和属性，我们也可以 **import kotlin.properties.setValue**。类需要一个MutableMap作为构造函数的参数。

想象我们从一个Json中加载了一个配置类，然后分配它们的key和value到一个map中。我们可以仅仅通过传入一个map的构造函数来创建一个实例：

```kotlin
import kotlin.properties.getValue

class Configuration(map: Map<String, Any?>) {
    val width: Int by map
    val height: Int by map
    val dp: Int by map
    val deviceName: String by map
}
```

作为一个参考，这里我展示下对于这个类怎么去创建一个必须要的map：

```kotlin
conf = Configuration(mapOf(
    "width" to 1080,
    "height" to 720,
    "dp" to 240,
    "deviceName" to "mydevice"
))
```

# 创建一个自定义的委托

先来说说我们要实现什么，举个例子，我们创建一个**notNull**的委托，它只能被赋值一次，如果第二次赋值，它就会抛异常。

Kotlin库提供了几个接口，我们自己的委托必须要实现：**ReadOnlyProperty**和**ReadWriteProperty**。具体取决于我们被委托的对象是**val**还是**var**。

我们要做的第一件事就是创建一个类然后继承**ReadWriteProperty**：

```kotlin
private class NotNullSingleValueVar<T>() : ReadWriteProperty<Any?, T> {

        override fun getValue(thisRef: Any?, property: KProperty<*>): T {
            throw UnsupportedOperationException()
        }

        override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        }
}
```

这个委托可以作用在任何非null的类型。它接收任何类型的引用，然后像getter和setter那样使用T。现在我们需要去实现这些函数。

- Getter函数 如果已经被初始化，则会返回一个值，否则会抛异常。
- Setter函数 如果仍然是null，则赋值，否则会抛异常。

```kotlin
private class NotNullSingleValueVar<T>() : ReadWriteProperty<Any?, T> {
    private var value: T? = null
    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value ?: throw IllegalStateException("${property.name} " +
                "not initialized")
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        this.value = if (this.value == null) value
        else throw IllegalStateException("${property.name} already initialized")
    }
}
```

现在你可以创建一个对象，然后添加函数使用你的委托：

```kotlin
object DelegatesExt {
    fun notNullSingleValue<T>():
            ReadWriteProperty<Any?, T> = NotNullSingleValueVar()
}
```

以下是自定义委托全部代码：

```kotlin
import kotlin.properties.ReadWriteProperty
import kotlin.reflect.KProperty

object DelegatesExt {

    fun <T : Any> notNullSingleValue(): ReadWriteProperty<Any?, T> = NotNullSingleValueVar()
}

private class NotNullSingleValueVar<T> : ReadWriteProperty<Any?, T> {

    private var value: T? = null

    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value ?: throw IllegalStateException("${property.name} not initialized")
    }

    override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        this.value = if (this.value == null) value
        else throw IllegalStateException("${property.name} already initialized")
    }
}
```

# 重新实现Application单例化

在这个情景下，委托就可以帮助我们了。我们直到我们的单例不会是null，但是我们不能使用构造函数去初始化属性。所以我们可以使用**notNull**委托：

```kotlin
class App : Application() {
    companion object {
        var instance: App by Delegates.notNull()
    }

    override fun onCreate() {
            super.onCreate()
            instance = this
    }
}
```

这种情况下有个问题，我们可以在app的任何地方去修改这个值，因为如果我们使用**Delegates.notNull()**，属性必须是var的。但是我们可以使用刚刚创建的委托，这样可以多一点保护。我们只能修改这个值一次：

```kotlin
class App : Application() {

    companion object {
        var instance: App by DelegatesExt.notNullSingleValue()
    }

    override fun onCreate() {
        super.onCreate()
        instance = this
    }
}
```
尽管，在这个例子中，使用单例可能是最简单的方法，但是我想用代码的形式展示给你怎么去创建一个自定义的委托。
