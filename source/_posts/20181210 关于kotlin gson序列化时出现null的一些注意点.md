---
title: 关于kotlin gson序列化时出现null的一些注意点
date: 2018-12-10 17:21:27
tags: 
- Android
- kotlin
- Gson
categories: Android
---

现在前后端基本都用json来传输数据，kotlin因为有空校验比如这个类：
```kotlin

data class XXXDataBean(
        var code:Int = 0,
        var data: DataBean = DataBean()) {
    class DataBean {
    }
}
```
在kotlin中定义的都是非空，可是如果给的json是这样：
```
{
    "code":1,
    "data":null
}
```
在`Gson().fromJson(json,XXXDataBean::class.java)`拿到的对象中，data会为空。如果json不给data的话，那么data依然是初始化时的那个`DataBean()`

---

5.1 更新
发现一个新的问题

数据类：
```
data class User(
    var age :Int,
    var name :String= "lucy"
)
```
在AndroidStudio中点击`Tools-Kotlin-Show Kotlin Bytecode`会出现字节码，再点击`Decompile`能看到反编译的java文件
![](https://user-gold-cdn.xitu.io/2019/5/1/16a72b5aebf780dc?w=1177&h=433&f=png&s=67604)
上面那个`User`的构造中，可以看到`age`没有默认值，`name`给了个默认值，此时，java中的构造为
```java
public User(int age, @NotNull String name) {
      Intrinsics.checkParameterIsNotNull(name, "name");
      super();
      this.age = age;
      this.name = name;
   }

   // $FF: synthetic method
   public User(int var1, String var2, int var3, DefaultConstructorMarker var4) {
      if ((var3 & 2) != 0) {
         var2 = "lucy";
      }
      this(var1, var2);
   }
```
如果`User`中的参数都给一个默认值，如
```
data class User(
    var age :Int=13,
    var name :String= "lucy"
)
```
则对应java的构造会变成3个，多了个无参构造。
```
public User(int age, @NotNull String name) {
      Intrinsics.checkParameterIsNotNull(name, "name");
      super();
      this.age = age;
      this.name = name;
   }

   // $FF: synthetic method
   public User(int var1, String var2, int var3, DefaultConstructorMarker var4) {
      if ((var3 & 1) != 0) {
         var1 = 13;
      }

      if ((var3 & 2) != 0) {
         var2 = "lucy";
      }

      this(var1, var2);
   }

   public User() {
      this(0, (String)null, 3, (DefaultConstructorMarker)null);
   }
```
那么问题来了：
在构造都有默认值的情况下，Gson是能够正常解析的。但如果有非默认值的时候，也就是java文件中没有空参构造时，
```
        val json =""
        val user2 = Gson().fromJson(json, User::class.java)
        println("gson: $user2")
```
结果为：`gson: User(age=0, name=null)`，即使我们给了`name`一个初始值，即使我们标明了`name`是`@NotNull`，解析的结果依然是个null。
> 结论：Gson解析需要一个空参构造，而kotlin如果构造中每个参数都有默认值的时候，会有一个空参，并且会正确的将默认值赋值。所以Gson可以正常解析。如果kotlin的构造有参数没有默认值，就会出问题。

