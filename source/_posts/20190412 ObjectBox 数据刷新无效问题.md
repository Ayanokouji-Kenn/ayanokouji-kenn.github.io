---
title: ObjectBox 数据刷新无效问题
date: 2019-04-12 17:21:27
tags: 
- Android
- ObjectBox
categories: Android
---

1. 实际项目中发现没有自动刷新，于是写了个小demo测试了下，

一开始是这样，tv是用来测试自动刷新的`TextView`，tv2是手动显示`user.age`的，用来对比。
```java
ObjectBox.getUserBox().query().build()
            .subscribe()
            .on(AndroidScheduler.mainThread())
            .observer {
                tv.text = it.firstOrNull()?.age?.toString()
            }
btn_modify.setOnClickListener {
            val singleUser = ObjectBox.getUserBox().all.firstOrNull()
            //数据库空的，就随便插一条数据
            if (singleUser==null) {
                ObjectBox.getUserBox().put(User(0,"lucy",13))
            }else {
            // 数据库有值，就拿出来，年龄自增一次
                singleUser.age+=1
                ObjectBox.getUserBox().put(singleUser)
            }
            tv2.setText(singleUser?.age?.toString())
        }
```
测试发现，一开始tv和tv2是保持同步的，随着不断点击`btn_modify`按钮，tv2还在自增，tv停了。。，我怀疑是不是observer的对象被回收了，于是改成了
```kotlin
val subcribtion  = DataSubscriptionList() //这里是成员变量
 //下面是方法里的内容
 ObjectBox.getUserBox().query().build()
            .subscribe(subcribtion)  //这里传进来
            .on(AndroidScheduler.mainThread())
            .observer {
                tv.text = it.firstOrNull()?.age?.toString()
            }
            
```
将subscribe保存在成员变量里延长生命周期，就可以一直刷新了。

用`DataSubscriptionList`可以保存多个`DataSubscription`,取消监听的时候可以一起`cancel`掉。
如果就一个的话，直接将observer返回的`DataSubscription`放到成员变量位置在一个个取消也是可以的。
