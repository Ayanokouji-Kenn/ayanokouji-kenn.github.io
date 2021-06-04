---
title: Room & Rxjava
date: 2018-01-12 19:27:27
tags: 
- Android
- Room
- Rxjava
categories: Android
---

 [Room](https://developer.android.com/topic/libraries/architecture/room.html)是google在[Architecture Components](https://developer.android.com/topic/libraries/architecture/index.html)
提出的一个持久化库，可以异步查询返回LiveData和Rxjava里的Maybe,Single,Flowable,其中LiveData和Flowable是可观测的查询。它们允许自动从数据库更新数据，从而刷新UI。

可以在UserDao里这么写，
```
@Query(“SELECT * FROM Users WHERE id = :userId”)
User getUserById(String userId);
```
但他有两个缺点：
1、这是同步的，所以会阻塞。
2、当user被修改的时候，我们总是需要手动调用这个方法。

在返回Maybe或Single的时候，确保`subscribeOn`调用了`Scheduler`的不同线程，而不是`AndroidSchedulers.mainThread()`

若想加上room对rxjava的支持，需要在`build.gradle`中添加以下内容
```
// RxJava support for Room
implementation “android.arch.persistence.room:rxjava2:1.0.0-alpha5”
// Testing support
androidTestImplementation “android.arch.core:core-testing:1.0.0-alpha5”
```

#Maybe
```
@Query(“SELECT * FROM Users WHERE id = :userId”)
Maybe<User> getUserById(String userId);
```
来说说发生了啥：
1、当数据库里**没有这条数据**的时候，`Maybe`会直接**`onComplete()`**。
2、当**有这条数据**的时候，`Maybe`会触发**`onSuccess`**，然后会正常的**`onComplete()`**。
3、当数据在`Maybe`的`onCompleted()`后进行了更新，那就**什么都不会发生了**。

#Single
```
@Query(“SELECT * FROM Users WHERE id = :userId”)
Single<User> getUserById(String userId);
```
1、当数据库里**没有这条数据**的时候，`Single`会直接**`onError(EmptyResultSetException.class)`**。
2、当**有这条数据**的时候，`Single`会触发**`onSuccess`**。
3、当数据在`Single.onCompleted()`后进行了更新，那就**什么都不会发生了**,自流被关闭后。

#Flowable
```
@Query(“SELECT * FROM Users WHERE id = :userId”)
Flowable<User> getUserById(String userId);
```
1、当数据库里**没有这条数据**的时候，`Flowable`**不会发射**，当然也就不会有`onNext`和`onError`。
2、当**有这条数据**的时候，`Flowable`会触发**`onNext`**。
3、任何时候**数据被更新**了，`Flowable`会自动发射来让你用最新的数据来更新UI。

Room and RxJava ， google写了个简单的[ 例子](https://github.com/googlesamples/android-architecture-components/tree/master/BasicRxJavaSample)

over~








