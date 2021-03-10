---
title: ObjectBox增加新字段的问题
date: 2020-07-24 10:04:27
tags: 
- Android
- ObjectBox
categories: Android
---
新加字段后，数据库新加一列，老数据为null，但kotlin定义为非null，此时若字段为number则ObjectBox查询后会给一个默认的0，但若是string就会报错了。
在2.6.0之后，官方有了报错提示，可以查看到具体报错信息。
若新加字段为string的话，在字段上增加`@DefaultValue("")`即可

[NativeCrash Cursor.nativeGetEntity](https://github.com/objectbox/objectbox-java/issues/876)

[Default values for new properties](https://github.com/objectbox/objectbox-java/issues/157)