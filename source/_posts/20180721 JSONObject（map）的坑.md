---
title: JSONObject（map）的坑
date: 2018-07-13 17:21:27
tags: 
- Android
- json
categories: Android
---

 1. bug出现：在一个EditText中输入文字，拿到输入的文字`view.text.trim()`，存入map，继而使用了`JSONObject（map）`之后，`JSONObject.toString()`发现存进去个“null”。

2. bug原因：debug后发现`view.text.trim()`存进map的是个`SpannableStringBuilder`，
JSONObject的构造是
```
 public JSONObject(Map copyFrom) {
        this();
        Map<?, ?> contentsTyped = (Map<?, ?>) copyFrom;
        for (Map.Entry<?, ?> entry : contentsTyped.entrySet()) {
            /*
             * Deviate from the original by checking that keys are non-null and
             * of the proper type. (We still defer validating the values).
             */
            String key = (String) entry.getKey();
            if (key == null) {
                throw new NullPointerException("key == null");
            }
            nameValuePairs.put(key, wrap(entry.getValue()));
        }
    }
```
遍历map，存起来。但是注意，并不是直接存`entry.getValue()`的。而是进行了wrap包裹，那么包裹了啥呢？
```
public static Object wrap(Object o) {
        if (o == null) {
            return NULL;
        }
        if (o instanceof JSONArray || o instanceof JSONObject) {
            return o;
        }
        if (o.equals(NULL)) {
            return o;
        }
        try {
            if (o instanceof Collection) {
                return new JSONArray((Collection) o);
            } else if (o.getClass().isArray()) {
                return new JSONArray(o);
            }
            if (o instanceof Map) {
                return new JSONObject((Map) o);
            }
            if (o instanceof Boolean ||
                o instanceof Byte ||
                o instanceof Character ||
                o instanceof Double ||
                o instanceof Float ||
                o instanceof Integer ||
                o instanceof Long ||
                o instanceof Short ||
                o instanceof String) {
                return o;
            }
            if (o.getClass().getPackage().getName().startsWith("java.")) {
                return o.toString();
            }
        } catch (Exception ignored) {
        }
        return null;
    }
```
`NULL`是个Object，可以看成就是个“null”
这个`wrap()`对`null `、`JSONArray`、 `JSONObject`、集合以及各种基本类型进行了处理，但是不能处理`SpannableStringBuilder`啊~
3.  bug解决
在获取到EditText的内容的时候 还是`view.text.toString().trim()`
