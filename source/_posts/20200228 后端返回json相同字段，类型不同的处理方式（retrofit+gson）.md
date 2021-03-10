---
title: 后端返回json相同字段，类型不同的处理方式（retrofit+gson）
date: 2020-02-28 14:13:27
tags: 
- Android
- Gson
categories: Android
---

方法有很多，你可以自己拿`responseBody`里的`json`，一个个字段自己解析；也可以给`okhttp`添加拦截器来处理response内容。
这里提供一种简便的方法。
> `Gson`在反序列化的时候，默认是将`{}`转成`LinkedTreeMap`，`[]` 转成`ArrayList`，value是数字的全部定义为了double。
 如果后端返回格式不规矩或者会变化的时候，将bean里定义的是`Any`或`*`即可

比如有时返回的是这样
![1.png](https://i.loli.net/2020/07/27/uIr83PATClWknd5.png)
有时是这样
![2.png](https://i.loli.net/2020/07/27/JyHfgh8mn1ltK7G.png)
可以看到`event`虽然都是个数组，可里面的内容完全就不一样，于是我就定义俩个`data class`，分别为`ZulipMessage`和`DelMsgEventDTO`
```kotlin 
data class ZulipEventDTO(
    val events: List<LinkedTreeMap<String,Any>>
)
```
`event`定义是一个集合，泛型就是`Gson`默认的`LinkedTreeMap<String,Any>`,这样不管是那种数据都可以接受到。并且存入map中。
解析的时候，根据`event.type`的类型，解析成不同的类。
```kotlin 
                          
private val parseMapGson = GsonBuilder().enableComplexMapKeySerialization().create() //重点行  
eventDTO.events.forEach { event ->
    when (event["type"]) {
       //根据type类型，解析成不同的类
        ZulipConst.EVENT_TYPE_MESSAGE -> {
            val messageJson = parseMapGson.toJson(event[ZulipConst.EVENT_TYPE_MESSAGE]) // 先将LinkedList转成json
            val zulipMessage = parseMapGson.fromJson<ZulipMessage>(messageJson, ZulipMessage::class.java) //再根据类型转成bean对象
            saveMsg2Database(zulipMessage)
        }
        ZulipConst.EVENT_TYPE_DELETE_MESSAGE -> {
            val messageJson = parseMapGson.toJson(event[ZulipConst.EVENT_TYPE_DELETE_MESSAGE])
            val delMsgEventDTO = parseMapGson.fromJson<ZulipMessage>(messageJson, DelMsgEventDTO::class.java)
            delMsgFromDb(delMsgEventDTO)
        }
        else -> {
        }
    }
    // Any被转成LinkedTreeMap时，所有数字都是double
    val eventId = (event["id"] as Double).toLong()
    if (eventId > lastEventId) {
        lastEventId = eventId
    }
}
```
---
总结：有`type`能帮忙判断返回数据类型的，那就根据类型定义不同的类，先用`LinkedTreeMap`接住数据，再根据类型转就是了。
如果没有`type`这种字段，完全无法预料返回的是个什么，那不妨就从LinkedTreeMap中直接取数据。