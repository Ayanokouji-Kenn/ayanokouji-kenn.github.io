---
title: 错误[ObjectBox] Relation target class 'Long' defined in class 'LocalMessage' could not be foun...
date: 2020-02-28 19:07:27
tags: 
- Android
- ObjectBox
categories: Android
---

使用`ObjectBox`时遇到的迷之问题，看到这个提示，怎么检查那个类都是加了`@Entity`注解的。
发生原因是：类里定义了`ObjectBox`不能理解的字段，比如`var atIds: MutableList<Long> = mutableListOf()`，要写个转换类加上去
```kotlin 
data class DataX（
    @Convert(converter = LongList2StringConverter::class, dbType = String::class)
    var atIds: MutableList<Long> = mutableListOf()
)

class LongList2StringConverter : PropertyConverter<MutableList<Long>, String> {
    override fun convertToDatabaseValue(entityProperty: MutableList<Long>?): String {
        return if (entityProperty.isNullOrEmpty()) {
            "[]"
        } else {
            val jsa = JSONArray()
            entityProperty.forEach {
                jsa.put(it)
            }
            jsa.toString()
        }
    }

    override fun convertToEntityProperty(databaseValue: String?): MutableList<Long> {
        val result = mutableListOf<Long>()
        return if (databaseValue.isNullOrEmpty()) {
            result
        } else {
            val jsa = JSONArray(databaseValue)
            for (i in 0 until jsa.length()) {
                result.add(jsa[i] as Long)
            }
            result
        }
    }

}
```
法预料返回的是个什么，那不妨就从LinkedTreeMap中直接取数据。