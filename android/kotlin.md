## 集合

```kotlin
constructor = aClass.getConstructor(*mConstructorSignature) as Constructor<out View>?
```

`*array`把数组展开作为可变参传入



## 变量

val（value的简写）用来声明一个不可变的变量，这种变量在初始赋值之后就再也不能重新赋值，对应Java中的final变量。

var（variable的简写）用来声明一个**可变的变量**

## 循环

for-in

1. `1..10` `[1,10]`
2. `1 until 10` `[1,10)`
3. `10 downTo ` `[10,1]`

在Kotlin中任何一个非抽象类默认都是不可以被继承的

`open`开启继承

允许只使用次构造函数

继承使用次构造函数不需要声明`()`

单继承，多实现

## 面向接口编程

```kotlin
fun main() {
    val student = Student("Jack", 19)
    doStudy(student)
}

fun doStuy(stu: Student) {
    stu.readBooks()
    stu.doHomework()
}
```

允许对接口中定义的函数进行默认实现



类-接口-类





## 修饰符

public、private、protected（当前类和子类）、internal（模块可见）

Kotlin默认所有的参数和变量都不可为空

那么可为空的类型系统是什么样的呢？很简单，就是在类名的后面加上一个问号。比如，Int表示不可为空的整型，而Int?就表示可为空的整型；String表示不可为空的字符串，而String?就表示可为空的字符串。



## 函数式参数

```kotlin
var labelsFormatter: (Float) -> String = { it.toString() }
```

接收一个 float 转化为 String

# 协程

