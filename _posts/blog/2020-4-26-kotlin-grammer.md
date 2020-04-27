---
layout: post
title: Kotlin小知识
categories: Blog
description:  Kotlin小知识
keywords:  Kotlin,小知识

---



## 1. 变量的初始化，与空类型判断

var a = "string" 定义就初始化可以自动类型推断

var view:View? 允许空

View?.name 使用时要这样

view!!.name 强制转换未非空类型



lateinit var view:View

可以稍后自己初始化



局部变量要自己初始化



var 变量

val 常量（类似java final)

## 2. 函数

返回值：Unit、Void



## 3. get set自动



## 4. Int Float，语言层面不在有基本类型了



## 5. 类

class A :constructor(var name:String)，主构造函数，次函数必须要调用一下主构造函数

constructor 构造函数

Init 初始化代码块

val name 可以通过getter动态

Compaion object {

​	//类似静态对象

}



单例（所有方法相当于静态，可以类名访问）

Object Single {

}

package级别的顶层定义属性，方法即可inline也可以



const val 编译时常量



## 6. 集合

Range,sequence流控制

Array convarient协变泛型，失去了原生数组，为了扩展方法和功能,基本类型数组intArray

List

Map ("1" to "value",)

Set

默认不可变(支持协变)，可变需要Mutable，



Switch-->when



==equals   ===引用



## 7. 泛型

数组没有擦出，泛型擦出了

? Extends 上界 out 或者全部类

？ super 下界  in

? *

Any Object



## 8. lambda

匿名函数，都是函数对象

& : 多重上述类型 where t XX,

Inline reifieid 泛型到具体类型

::a 函数赋值，传递，函数对象

(::a).invoke()可以调用

（String）->Unit

匿名函数

val d = fun(s:Int):String {

}

View:View ->

It

最后一行就是return，不要写




## 9. 其他
密封类：类似于常量或者枚举 sealed class Expr
数据类： data class User(var name:String)

内联函数：inline


单例: object/compaion object

高阶函数:返回的时函数



委托 类似动态代理

扩展函数：类编译优化

反射
注解




















