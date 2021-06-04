---
title: onStartCommand中的intent为null
date: 2016-08-24 10:33:27
tags: problems
---
- 当需要在服务中，对intent做什么事情的时候，先加入这个判断，特别是那个getAction不能忘了
~~~java
if (intent!=null && intent.getAction()!=null) { 
            //do something
}
~~~