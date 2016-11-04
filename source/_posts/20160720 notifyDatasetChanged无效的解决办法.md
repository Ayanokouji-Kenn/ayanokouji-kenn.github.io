---
title: notifyDatasetChanged无效的解决办法
date: 2016-07-20 10:33:27
tags: problems
---
- 出现问题 
比如这样给适配器传一个集合
```java
        List datas;
        datas = DBDao.selectMyClock(); //此句是返回一个List
        clocksAdapter = new AlarmClockListAdapter(getContext(), datas);
        myClocks.setAdapter(clocksAdapter);
```
数据库刷新了，然后想这样刷新数据
```java
    private void refresh() {
        datas = DBDao.selectMyClock();
        clocksAdapter.notifyDataSetChanged();
    }
```
是无效的，因为datas指向了两个对象

- 解决办法：清空集合，再将新的数据添加进去即可
```java
    public void refresh() {
        datas.clear();
        datas.addAll(DBDao.selectMyClock());
        clocksAdapter.notifyDataSetChanged();
    }
```