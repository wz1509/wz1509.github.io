---
layout: post
title: Kotlin SharedPreferences最佳实现
date: 2018-09-03 08:30:00.000000000 +09:00
tags: Android Kotlin
---
[TOC]

# 关于

用于访问和修改返回的首选项数据的接口**Context.getSharedPreferences(String, int)**。对于任何特定的首选项集，所有客户端共享此类的单个实例。对首选项的修改必须通过SharedPreferences.Editor对象来确保首选项值保持一致状态并在提交存储时进行控制。从各种get方法返回的对象必须被应用程序视为不可变。

注意：此类提供强大的一致性保证。它使用昂贵的操作可能会减慢应用程序的速度。经常改变可以容忍损失的属性或属性应该使用其他机制。有关详细信息读取上的评论 **SharedPreferences.Editor.commit()**和**SharedPreferences.Editor.apply()**。

> 注意：此类不支持跨多个进程使用。

# 概括
嵌套类：

接口 | 方法 | 注释 |
---|---|---
interface | SharedPreferences.Editor | 用于修改SharedPreferences 对象中的值的接口。
interface | SharedPreferences.OnSharedPreferenceChangeListener | 更改共享首选项时要调用的回调的接口定义。 

公共方法：

返回值 | 函数名
---|---
abstract boolean | contains(String key)<br>检查Preferences中是否包含此key 
abstract SharedPreferences.Editor | edit()<br>为这些首选项创建一个新的编辑器，通过该编辑器可以对首选项中的数据进行修改，并将这些更改原子地提交回SharedPreferences对象。
abstract Map | getAll()<br>从首选项中检索所有值。
abstract boolean | getBoolean(String key, boolean defValue)<br>从首选项中检索布尔值。
abstract float | getFloat(String key, float defValue)<br>从首选项中检索浮点值。
abstract int | getInt(String key, int defValue)<br>从首选项中检索int值。
abstract long | getLong(String key, long defValue)<br>从首选项中检索长值。
abstract String | getString(String key, String defValue)<br>从首选项中检索String值。
abstract Set<String> | getStringSet(String key, Set<String> defValues)<br>从首选项中检索一组String值。
abstract void | registerOnSharedPreferenceChangeListener(SharedPreferences.OnSharedPreferenceChangeListener listener)<br>注册在首选项发生更改时调用的回调。
abstract void | unregisterOnSharedPreferenceChangeListener(SharedPreferences.OnSharedPreferenceChangeListener listener)<br>取消注册先前的回调。

> 上面表格乱版的 --->>> [Jokey's blog](https://blog.csdn.net/Jokey_wz/article/details/82350759)
> https://developer.android.com/reference/android/content/SharedPreferences

# kotlin中对SharedPreferences的封装

封装类：
```kotlin
import android.content.Context
import android.content.SharedPreferences
import android.preference.PreferenceManager
import kotlin.properties.ReadWriteProperty
import kotlin.reflect.KProperty

open class SharedPreferencesUtils(context: Context) {

    private val preferences: SharedPreferences = PreferenceManager.getDefaultSharedPreferences(context)

    var username by SharedPreferenceDelegates.string(defaultValue = "this is username")

    var age by SharedPreferenceDelegates.int()

    var flag by SharedPreferenceDelegates.boolean()

    var currentDateTime: Long by SharedPreferenceDelegates.long(System.currentTimeMillis())

    var money by SharedPreferenceDelegates.float()

    var setString by SharedPreferenceDelegates.setString()

    private object SharedPreferenceDelegates {

        fun int(defaultValue: Int = 0) = object : ReadWriteProperty<SharedPreferencesUtils, Int> {

            override fun getValue(thisRef: SharedPreferencesUtils, property: KProperty<*>): Int {
                return thisRef.preferences.getInt(property.name, defaultValue)
            }

            override fun setValue(thisRef: SharedPreferencesUtils, property: KProperty<*>, value: Int) {
                thisRef.preferences.edit().putInt(property.name, value).apply()
            }
        }

        fun long(defaultValue: Long = 0L) = object : ReadWriteProperty<SharedPreferencesUtils, Long> {

            override fun getValue(thisRef: SharedPreferencesUtils, property: KProperty<*>): Long {
                return thisRef.preferences.getLong(property.name, defaultValue)
            }

            override fun setValue(thisRef: SharedPreferencesUtils, property: KProperty<*>, value: Long) {
                thisRef.preferences.edit().putLong(property.name, value).apply()
            }
        }

        fun boolean(defaultValue: Boolean = false) = object : ReadWriteProperty<SharedPreferencesUtils, Boolean> {
            override fun getValue(thisRef: SharedPreferencesUtils, property: KProperty<*>): Boolean {
                return thisRef.preferences.getBoolean(property.name, defaultValue)
            }

            override fun setValue(thisRef: SharedPreferencesUtils, property: KProperty<*>, value: Boolean) {
                thisRef.preferences.edit().putBoolean(property.name, value).apply()
            }
        }

        fun float(defaultValue: Float = 0.0f) = object : ReadWriteProperty<SharedPreferencesUtils, Float> {
            override fun getValue(thisRef: SharedPreferencesUtils, property: KProperty<*>): Float {
                return thisRef.preferences.getFloat(property.name, defaultValue)
            }

            override fun setValue(thisRef: SharedPreferencesUtils, property: KProperty<*>, value: Float) {
                thisRef.preferences.edit().putFloat(property.name, value).apply()
            }
        }

        fun string(defaultValue: String? = null) = object : ReadWriteProperty<SharedPreferencesUtils, String?> {
            override fun getValue(thisRef: SharedPreferencesUtils, property: KProperty<*>): String? {
                return thisRef.preferences.getString(property.name, defaultValue)
            }

            override fun setValue(thisRef: SharedPreferencesUtils, property: KProperty<*>, value: String?) {
                thisRef.preferences.edit().putString(property.name, value).apply()
            }
        }

        fun setString(defaultValue: Set<String>? = null) = object : ReadWriteProperty<SharedPreferencesUtils, Set<String>?> {
            override fun getValue(thisRef: SharedPreferencesUtils, property: KProperty<*>): Set<String>? {
                return thisRef.preferences.getStringSet(property.name, defaultValue)
            }

            override fun setValue(thisRef: SharedPreferencesUtils, property: KProperty<*>, value: Set<String>?) {
                thisRef.preferences.edit().putStringSet(property.name, value).apply()
            }
        }
    }
}
```

使用：

```kotlin
import android.annotation.SuppressLint
import android.support.v7.app.AppCompatActivity
import android.os.Bundle
import com.wazing.room.utils.SharedPreferencesUtils
import kotlinx.android.synthetic.main.activity_main.*

class MainActivity : AppCompatActivity() {

    private val preferences by lazy { SharedPreferencesUtils(this) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        preferences.age = 56
        preferences.username = "刘德华"
        preferences.flag = true
        preferences.money = 9.30F
        preferences.currentDateTime = System.currentTimeMillis()
        preferences.setString = setOf("Android", "Ios")

        tv_content.text = StringBuffer().append(preferences.username)
                .append("\n").append(preferences.age)
                .append("\n").append(preferences.flag)
                .append("\n").append(preferences.money)
                .append("\n").append(preferences.currentDateTime)
                .append("\n").append(preferences.setString.toString())
    }
}
```

可以看出采用kotlin的委托属性更加方便，直接在**SharedPreferencesUtils**中定义属性，该属性作为key就可以开始用。

效果图：

<div align=center><img width="480" height="800" src="http://p8i9mda7f.bkt.clouddn.com/18-9-3/17194331.jpg"/></div>
