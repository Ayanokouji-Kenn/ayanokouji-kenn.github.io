---
title: google sample-android-architecture看源码学姿势
date: 2018-01-04 16:23:27
tags: 
- Android
- 源码
categories: Android
cover: http://upload-images.jianshu.io/upload_images/2524531-255d01b3ae072217.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240
---

1、在util包下看到了几个文件，是文件不是class，打开一看
```java
/*
 * Copyright (C) 2017 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */
package com.example.android.architecture.blueprints.todoapp.util

/**
 * Various extension functions for AppCompatActivity.
 */

import android.app.Activity
import android.arch.lifecycle.ViewModel
import android.arch.lifecycle.ViewModelProviders
import android.support.annotation.IdRes
import android.support.v4.app.Fragment
import android.support.v4.app.FragmentManager
import android.support.v4.app.FragmentTransaction
import android.support.v7.app.ActionBar
import android.support.v7.app.AppCompatActivity
import com.example.android.architecture.blueprints.todoapp.ViewModelFactory


const val ADD_EDIT_RESULT_OK = Activity.RESULT_FIRST_USER + 1
const val DELETE_RESULT_OK = Activity.RESULT_FIRST_USER + 2
const val EDIT_RESULT_OK = Activity.RESULT_FIRST_USER + 3

/**
 * The `fragment` is added to the container view with id `frameId`. The operation is
 * performed by the `fragmentManager`.
 */
fun AppCompatActivity.replaceFragmentInActivity(fragment: Fragment, frameId: Int) {
    supportFragmentManager.transact {
        replace(frameId, fragment)
    }
}

/**
 * The `fragment` is added to the container view with tag. The operation is
 * performed by the `fragmentManager`.
 */
fun AppCompatActivity.addFragmentToActivity(fragment: Fragment, tag: String) {
    supportFragmentManager.transact {
        add(fragment, tag)
    }
}

fun AppCompatActivity.setupActionBar(@IdRes toolbarId: Int, action: ActionBar.() -> Unit) {
    setSupportActionBar(findViewById(toolbarId))
    supportActionBar?.run {
        action()
    }
}

fun <T : ViewModel> AppCompatActivity.obtainViewModel(viewModelClass: Class<T>) =
        ViewModelProviders.of(this, ViewModelFactory.getInstance(application)).get(viewModelClass)

/**
 * Runs a FragmentTransaction, then calls commit().
 */
private inline fun FragmentManager.transact(action: FragmentTransaction.() -> Unit) {
    beginTransaction().apply {
        action()
    }.commit()
}
```
里面存放着一些扩展函数，供activity直接方便的调用。还有最后的inline内联函数等都是kotlin的特点。这个文件可以看成是java的一个工具类了。
如果是对view进行操作的工具类 ，那就建立ViewExt.kt，里面是
```
/**
 * Transforms static java function Snackbar.make() to an extension function on View.
 */
fun View.showSnackbar(snackbarText: String, timeLength: Int) {
    Snackbar.make(this, snackbarText, timeLength).show()
}

/**
 * Triggers a snackbar message when the value contained by snackbarTaskMessageLiveEvent is modified.
 */
fun View.setupSnackbar(lifecycleOwner: LifecycleOwner,
        snackbarMessageLiveEvent: SingleLiveEvent<Int>, timeLength: Int) {
    snackbarMessageLiveEvent.observe(lifecycleOwner, Observer {
//        这个it是singleLiveEvent.value，存放的是snackbar要展示的文本的resId
//        如果it不是null，才做showSnackbar的操作
        it?.let { showSnackbar(context.getString(it), timeLength) }
    })
}

/**
 * Reloads the data when the pull-to-refresh is triggered.
 *
 * Creates the `android:onRefresh` for a [SwipeRefreshLayout].
 */
@BindingAdapter("android:onRefresh")
fun ScrollChildSwipeRefreshLayout.setSwipeRefreshLayoutOnRefreshListener(
        viewModel: TasksViewModel) {
    setOnRefreshListener { viewModel.loadTasks(true) }
}
```
除了扩展函数，还包含了DataBinding的一些自定义的BindingAdapter比如上面的那个SwipeRefreshLayout的刷新监听。

2、 viewmodle在生成完之后，可以直接通过.apply{}对vm存放的一些LiveData的监听放在里面。

3、fragment的onCreateView里设置`setHasOptionsMenu(true)`之后，可以对`onOptionsItemSelected` `onCreateOptionsMenu`进行重写来操作toolbar.

4、SwipeRefreshLayout的第一个child如果不是list的话会不能触发refresh的监听，可以继承他来设置监听具体哪个childView
```
class ScrollChildSwipeRefreshLayout @JvmOverloads constructor(
        context: Context,
        attrs: AttributeSet? = null
) : SwipeRefreshLayout(context, attrs) {

    var scrollUpChild: View? = null

    override fun canChildScrollUp() =
            scrollUpChild?.canScrollVertically(-1) ?: super.canChildScrollUp()
}
```
直接绑定上刷新监听需要先弄两个静态方法，标上@BindingAdapter
```
object TestBindingAdapter {
    @BindingAdapter("android:items")
    @JvmStatic
    fun setItems(rv: RecyclerView, items: List<String>) {
        with(rv.adapter as BaseQuickAdapter<String,BaseBindingViewHolder>){
            notifyDataSetChanged()
        }
    }

    @BindingAdapter("android:onRefresh")
    @JvmStatic fun SwipeRefreshLayout.setOnRefreshListener(
            viewModel: Vm) {
        setOnRefreshListener { viewModel.refresh() }
    }
}
```
然后在xml里
```
<android.support.v4.widget.SwipeRefreshLayout
        android:id="@+id/srl"
        android:onRefresh="@{vm}" //就直接可以这么用了
        app:refreshing="@{vm.dataLoading}"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toBottomOf="@id/appbar" >
        <android.support.v7.widget.RecyclerView
            android:items="@{vm.datas}"
            android:id="@+id/rv"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
    </android.support.v4.widget.SwipeRefreshLayout>
```
如果要自动更新刷新状态`app:refreshing="@{vm.dataLoading}"`，那么vm.dataLoading是个ObservableBoolean(),且在`attrs.xml`中加上下面的
```
<resources>
    <declare-styleable name="SwipeRefreshLayout">
        <attr name="refreshing" format="boolean" />
    </declare-styleable>
</resources>
```

5、在fragment中对activity里的fab（floatActionButton）进行操作，就直接通过
```
activity.findViewById<FloatingActionButton>(R.id.fab_add_task).run {
            setImageResource(R.drawable.ic_add)
            setOnClickListener {
                viewDataBinding.viewmodel?.addNewTask()
            }
        }
```
去设置图标和点击事件，不要写在activity里再传进frag。

6、Toolbar用得少。基本都是自定义，头一次看见除了popupWindow还有个东西叫popupMenu。。用来在ActionBar上显示一个窗口

7、对于listview和databinding的使用：
首先申明一个单例TaskListBindings
```

/**
 * Contains [BindingAdapter]s for the [Task] list.
 */
object TasksListBindings {

    @BindingAdapter("app:items")
    @JvmStatic fun setItems(listView: ListView, items: List<Task>) {
        with(listView.adapter as TasksAdapter) {
            replaceData(items)
        }
    }
}
```
在xml里配置上
```
  <ListView
                android:id="@+id/tasks_list"
                app:items="@{viewmodel.items}"
                android:layout_width="match_parent"
                android:layout_height="wrap_content" />
```
注意这个items是一个ObservableArrayList
`val items: ObservableList<Task> = ObservableArrayList()`
当他发生变化的时候，会去上面的bindingadapter里调用tasksAdapter.replaceData(items)方法进行刷新

8、因为binding与xml是绑定的，所以可以直接FragmentXXXBinding.inflate(),不用写R.layout.xxx了。

9、事件的流转通过调用vm的方法，改变vm.liveData，从而通知到注册了liveData的地方进行处理事件。比如页面的跳转，弹个snackbar，弹个dialog，弹个popupWindow，都从liveData通知到Activity，让activity去做。

10、intent包含的key 以及requestCode，往哪里去，就把这些常量写在哪里。

11、设置actionbar左上角返回
```
//这个方法在第一个代码段中定义的，往上翻↑
setupActionBar(R.id.toolbar) {
            setDisplayHomeAsUpEnabled(true) //显示返回图标
            setDisplayShowHomeEnabled(true) //显示左上图标
        }

 override fun onSupportNavigateUp(): Boolean {
        onBackPressed()
        return true
    }
```

12、接受单个事件或数据变动
```
class SingleLiveEvent<T> : MutableLiveData<T>() {

    private val pending = AtomicBoolean(false)

    @MainThread
    override fun observe(owner: LifecycleOwner, observer: Observer<T>) {
//        判断是否曾经注册过监听了
        if (hasActiveObservers()) {
            Log.w(TAG, "Multiple observers registered but only one will be notified of changes.")
        }

        // Observe the internal MutableLiveData
        super.observe(owner, Observer<T> { t ->
            if (pending.compareAndSet(true, false)) {
                observer.onChanged(t)
            }
        })
    }

    @MainThread
    override fun setValue(t: T?) {
        pending.set(true)
        super.setValue(t)
    }

    /**
     * Used for cases where T is Void, to make calls cleaner.
     */
    @MainThread
    fun call() {
        value = null
    }

    companion object {
        private const val TAG = "SingleLiveEvent"
    }
}
```
在vm中建立一个对象 `val newTaskEvent= SingleLiveEvent<Void>()`
在某个Activity或Fragment中注册监听。
```
viewModel = obtainViewModel().apply {
            // Subscribe to "new task" event
            newTaskEvent.observe(this@TasksActivity, Observer<Void> {
                this@TasksActivity.addNewTask()
            })
        }
```
13、![image.png](http://upload-images.jianshu.io/upload_images/2524531-f5d65d2b6b278195.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到分包是根据PBF（package by feature）的，而不是PBL （package by layer），根据业务特征来分类。而其中有一个data包是这样的
![image.png](http://upload-images.jianshu.io/upload_images/2524531-1f89abb7c19b6ac1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

分为了本地数据和远程数据。
在TasksDataSource中定义了操作数据的接口。在LocalDataSource和RemoteDataSource中具体实现。比如本地就用room去数据库拿，remote就去操作retrofit去网络获取。
TaskRepository数据仓库 也是继承自TasksDataSource，且同时持有本地和远程的DataSource，综合获取、处理并对外提供数据。
可以看到层次非常清晰。我想起了一张图，等我去google官网找下。。
找到了，是这个
![image.png](http://upload-images.jianshu.io/upload_images/2524531-255d01b3ae072217.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)





