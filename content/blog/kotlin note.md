+++
title = "Kotlin Note"
date = 2022-11-16
[taxonomies]
  tags = ["note", "kotlin", "Android"]

+++

# sealed class 怎么用？
sealed class = 枚举类+普通类 = 密封类🐝
示例：
```kotlin
//密封类要跟子类写在一个kt文件里

sealed class Result
class Success(val code: Int) : Result()
class Exception(val code: Int, val message: String) : Result()
```
🐝类跟 when 表达一起使用的时候，不需要补充 else-> 分支
[参考](https://juejin.cn/post/6844904163835396103)

- 为了简化使用，sealed class 常搭配 data class 一起用，再看看 data class 怎么用呢？
用了 data class 就不用自己写这些方法了：equals 、hashCode 、toString（数据类应该实现的方法）
示例：
```kotlin
//密封类要跟子类写在一个kt文件里

sealed class Result
data class Success(val code: Int) : Result()
data class Exception(val code: Int, val message: String) : Result()
```
Notice: 可变参数 vararg 不能用在data class里 
*Primary constructor vararg parameters are forbidden for data classes*

# inner class
inner class = nested class
示例：
```kotlin
class BaseAccessibilityConfig {
    inner class AccessibilityNode   
}
```
AccessibilityNode只能被BaseAccessibilityConfig/其子类 实例化。有了inner修饰，AccessibilityNode就能访问到BaseAccessibilityConfig中的其他成员，没有则不能访问，权限相当于一个普通fun。

    Note:
    Constructor of inner class AccessibilityNode can be called only with receiver of containing class.

    About 'inner':
    With 'inner' declared, AccessibilityNode can access the outer BaseAccessibilityConfig's mActivity,
    while it can't do this without the 'inner'.