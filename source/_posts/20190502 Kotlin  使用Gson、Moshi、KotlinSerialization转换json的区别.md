---
title: Kotlin使用Gson、Moshi、KotlinSerialization转换json的区别
date: 2019-05-02 17:21:27
tags: 
- Android
- kotlin
- Gson
categories: Android
---

>**简介：**
`Gson`是google出的，`Moshi`是square出的，KotlinSerializatin（以下简称KS）是kotlin官方的。

测试用数据类
```kotlin
    @JsonClass(generateAdapter = true) //这个注解是moshi用
    @Serializable                       //这个注解KS用
    data class User(
            val age :Int=11,
            val name :String= "lucy"
    )
```
测试代码
```kotlin
        // Moshi
        try {
            val moshi = Moshi.Builder().build()
            val user1 = moshi.adapter(User::class.java).fromJson(json)
            Log.d("moshi",user1.toString())
        } catch (e: Exception) {
            Log.e("moshi",e.message)
        }
        // Gson
        try {
            val user2 = Gson().fromJson(json, User::class.java)
            Log.d("Gson", user2.toString())
        } catch (e: Exception) {
            Log.e("Gson",e.message)
        }
        // KS
        try {
            val userJson = Json.parse(User.serializer(), json)
            Log.d("KS",userJson.toString())
        } catch (e: Exception) {
            Log.e("KS",e.message)
        }
```
## 1. 测试一，参数空缺时，三家的处理。测试json为
```
 val json = """
            {

            }
        """.trimIndent()
```
结果：

![](https://user-gold-cdn.xitu.io/2019/5/2/16a78f8f57828b2b?w=363&h=50&f=png&s=5827)
`Moshi`和`Gson`可以正常解析，并且可以将默认值赋值，`KS`抛了个参数缺失的异常。。

## 2.测试二，多给了些数据类没有定义的参数
```
val json = """
            {
                "name":"tom",
                "isFemale":false,
                "age":24
            }
        """.trimIndent()
```
结果：

![](https://user-gold-cdn.xitu.io/2019/5/2/16a78fd143925fbf?w=709&h=68&f=png&s=16319)
KS报了个未知key，但可以取消严格模式来过滤。在改为`val userJson = Json.nonstrict.parse(User.serializer(), json)`后，三家都能正常解析。

## 3. 测试三，从测试一得知，不给参数是可以正常拿到默认值的，那么给定null呢？
```
val json = """
            {
                "name":null
            }
        """.trimIndent()
```
结果：
![](https://user-gold-cdn.xitu.io/2019/5/2/16a79024c38c7d68?w=455&h=57&f=png&s=8938)
三家都有点问题。
`moshi`和`KS`一样，都能认得出来`kotlin`有定义了`name`是`notnull`的，json给定了null都报错了，`Gson`看起来是正常解析的，但是它没有认出kotlin的非空定义。

## 4.测试四 上述用例中User都是用默认值的，现在我们去除掉
```
data class User(
            val age :Int,
            val name :String
    )
```
给一个空串去解析，理所当然的，moshi和KS都报错了，Gson能正常解析，然而与测试一不同，同样是空串去解析，这次Gson并没有拿到默认值。
![](https://user-gold-cdn.xitu.io/2019/5/2/16a79095eb08068e?w=363&h=53&f=png&s=7772)

> 从结果来看，Gson还是比较稳的，至少在java层面来说，它没有太大问题。只是在对kotlin的支持不够。一个是空类型认不出；一个是它只取空构造去创建对象，kotlin在每个构造参数都有默认值的时候，会有空构造函数，然后会调用带参构造将默认值初始化，所以Gson能够正常解析。但如果没有默认值的时候，kotlin生成的字节码中是没有空构造的，此时Gson就拿不到默认值了。而moshi和ks能够认出kotlin的一些特点，但是区区一个解析json都要我去处理异常，我是不高兴的。。

设置成所有参数都可空，且给一个默认值，那么moshi和Gson就都没有问题了。但是这样就放弃了kotlin的非空特性

谁的好的方法，比如过滤掉Gson对null的赋值，欢迎留言。







