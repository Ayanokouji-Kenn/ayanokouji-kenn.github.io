---
title: ExpandableListView中childview的数据出现了2次
date: 2016-08-10 10:33:27
tags: problems
---
- 问题：给child的集合个数是3个，但是打开折叠，出现了6个数据，debug发现getChildView执行了groupCount×childCount×2次，这显然是不合理的
- 原因：
虽然ExpandableListView提供了点击、展开、折叠的监听
```java
listview.setOnGroupClickListener()
listview.setOnGroupExpandListener()
listview.setOnGroupCollapseListener()
```
但ExpandableListView自己就实现了点击group展开，再次点击折叠，不需要手动去写这个操作。
除非需要同一时间只允许打开一个group，那么可以
~~~java
//展开监听，展开的时候遍历所有组，将其他的折叠起来
        listview.setOnGroupExpandListener(new ExpandableListView.OnGroupExpandListener() {
            @Override
            public void onGroupExpand(int groupPosition) {
                for (int i = 0; i < ringAdapter.getGroupCount(); i++) {
                    if (groupPosition != i) {
                        listview.collapseGroup(i);
                    }
                }
            }
        });
~~~
***
**前方高能**
如果**手欠**再次去实现了一遍展开和折叠的操作的话，比如这样，
~~~java
listview.setOnGroupClickListener(new ExpandableListView.OnGroupClickListener() {
            @Override
            public boolean onGroupClick(ExpandableListView parent, View v, int groupPosition, long id) {
                //如果是展开的，就合上，如果合上的点击展开
                if (listview.isGroupExpanded(groupPosition)) {
                    listview.collapseGroup(groupPosition);
                } else {
                    listview.expandGroup(groupPosition);
                }
                return false;
            }
        });
~~~
那么就会被认为展开了2次操作，会多调一遍getChildView()，造成数据重复！！