---
title: Gson解析成list、map
date: 2017-09-19 12:46:27
tags: 
- Android
- Gson
categories: Android
---

新建一个type传入即可
```
Type type = new TypeToken<HashMap<String, Integer>>() {}.getType();
   HashMap<String,Integer> map   = new Gson().fromJson(json, type);
```


