---
title: kotlin协程在android的应用
date: 2019-11-13 17:21:27
tags: 
- Android
- kotlin
- 协程
categories: Android
---

1. 可以让View层继承`CoroutineScope by MainScope`，就可以直接使用`launch`调用协程
```kotlin
abstract class BaseFragment:Fragment() ,CoroutineScope by MainScope(){
      fun xxx(){
          launch{
          //...业务逻辑，默认在主线程执行
          }
      }  
}
```
2. 如果使用了ViewModle的话，可以在vm内直接使用`viewModelScope.launch {  }`开启协程，也是在主线程内执行。
3. 那么协程是怎么协的呢，举个栗子，retrofit网络请求，可以这样（***注意这是举个栗子，实际开发用retrofit一般不会这么做，返回要么是Call，要么是Response，要么是Rxjava的Observable，否则服务端给个响应码404就异常了，捕获异常也只能拿到异常信息 ，拿不到errorBody中的信息***）
```
    interface Api{
        @GET("xxx/xxx")
        suspend fun getData(): BaseDTO<*>
    }

    suspend fun getData() : BaseDTO<*> = withContext(Dispatchers.IO){
        Retrofit.Builder().baseUrl("").build().create(Api::class.java).getData()
    }
    
    fun test(){
        launch { 
            val data = getData() // getData是个suspend方法，withContext指定了运行在IO线程池
            refeshUI(data)// 回到launch下面，这里是运行在主线程的，可以拿到data后刷新UI
        }
    }
```

3.如果suspend不能直接返回结果呢，比如回调的时候怎么办？还是用retrofit举栗子
```kotlin
    interface Api{
        @GET("xxx/xxx")
        suspend fun getData(): BaseDTO<*>

        @GET("xxx/xxx")
        suspend  fun getData2():Call<BaseDTO<*>>
    }

   // 这个suspend方法一样能返回BaseDTO的数据
    suspend fun getData2() :BaseDTO<*>? = suspendCoroutine {continuation->
        val call = Retrofit.Builder().baseUrl("").build().create(Api::class.java).getData2()
        call.enqueue(object:Callback<BaseDTO<*>>{
            override fun onFailure(call: Call<BaseDTO<*>>, t: Throwable) {
                continuation.resume(null)
            }

            override fun onResponse(call: Call<BaseDTO<*>>, response: Response<BaseDTO<*>>) {
                continuation.resume(response.body())
            }

        })
    }
    fun test(){
        launch { 
            val data2 = getData2() // getData2是个suspend方法，withContext指定了运行在IO线程池
            refeshUI(data2)// 回到launch下面，这里是运行在主线程的，可以拿到data后刷新UI
        }
    }
```
> 协程的优势就是可以直接在一个花括号内，像写同步代码一样写异步代码，线程协作运行。一些不必要的LiveData、EventBus、Handler之类跳来跳去的通知可以省去了。

