---
title: Google N年前的Plaid重构了！学习笔记
date: 2019-11-13 17:21:27
tags: 
- Android
- 源码
categories: Android
---

1. recyclerview的`android:clipToPadding="false"`,可以让内容滑到padding所占用的空间。
2. AS里可以直接将svg和psd转成矢量图来用的，这些图是不用按照分辨率提供不同的图了，有些系统图标也挺好用的~
![image](https://upload-images.jianshu.io/upload_images/2524531-e5129e1a39d9bf47?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
3. 应用启动的时候，根据有没有连接网络，会有一些不同的逻辑，所以在Activity的onCreate中有这么一段
```kotlin
        connectivityChecker?.apply {
            lifecycle.addObserver(this)//增加对UI生命周期的监听
            // 这是个LiveData，当发生变化的时候，走自己的业务逻辑
            connectedStatus.observe(this@HomeActivity, Observer<Boolean> {
                if (it) {
                    handleNetworkConnected()
                } else {
                    handleNoNetworkConnection()
                }
            })
        } ?: handleNoNetworkConnection()
```
```kotlin 
class ConnectivityChecker(
    private val connectivityManager: ConnectivityManager
) : LifecycleObserver {

    /* 是否在监听连接状态 */
    private var monitoringConnectivity = false

    /** 这里用两个LiveData的目的应该是只允许私有可变，所以必须有私有的MutableLiveData
      提供给外部的不允许postValue，所以用个LiveData */
    private val _connectedStatus = MutableLiveData<Boolean>()
    val connectedStatus: LiveData<Boolean>
        get() = _connectedStatus

    /** 连接状态回调 */
    private val connectivityCallback = object : ConnectivityManager.NetworkCallback() {
        override fun onAvailable(network: Network) {
            _connectedStatus.postValue(true)
            // 已经连上了，就断掉监听
            connectivityManager.unregisterNetworkCallback(this)
            monitoringConnectivity = false
        }

        override fun onLost(network: Network) {
            _connectedStatus.postValue(false)
        }
    }

    /** 在Activity中注册了生命周期，当发生变化的时候会走到这里 */
    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun stopMonitoringConnectivity() {
        if (monitoringConnectivity) {
            connectivityManager.unregisterNetworkCallback(connectivityCallback)
            monitoringConnectivity = false
        }
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun startMonitoringConnectivity() {
        val activeNetworkInfo: NetworkInfo? = connectivityManager.activeNetworkInfo
        val connected = activeNetworkInfo != null && activeNetworkInfo.isConnected
        _connectedStatus.postValue(connected)
        if (!connected) {
            // we don't have internet connection, so we listen to notifications in connection status
            connectivityManager.registerNetworkCallback(
                NetworkRequest.Builder()
                    .addCapability(NetworkCapabilities.NET_CAPABILITY_INTERNET).build(),
                connectivityCallback
            )
            monitoringConnectivity = true
        }
    }
}
```
4. toolbar的布局是在`res/menu/...`里定义的，里面会有一些item。
那些item可以设置图标`android:icon="@drawable/ic_search_24dp"`，这个search图标就是上面说的矢量图，以后app要用，就直接复制过去好了。

    也可以设置文字`android:title="@string/search"`,还有显示方式`android:showAsAction="always"`这种知道的，这里略微提一下。
    
    还可以指定他是个什么控件，比如
```
 <item
        android:id="@+id/menu_theme"
        android:title="@string/theme"
        android:actionViewClass="android.widget.CheckBox"
        android:showAsAction="always"
        tools:ignore="AppCompatResource" />
```
这就是个`CheckBox`,然后可以拿出来进行一顿操作
```
        toolbar.inflateMenu(R.menu.main)
        val toggleTheme = toolbar.menu.findItem(R.id.menu_theme)
        val actionView = toggleTheme.actionView

        (actionView as CheckBox?)?.apply {
            setButtonDrawable(R.drawable.asl_theme)//这里设置的是图标图片和动画了，
            isChecked = ColorUtils.isDarkTheme(this@HomeActivity)
            jumpDrawablesToCurrentState()
            setOnCheckedChangeListener { _, isChecked ->}
```
`setButtonDrawable(R.drawable.asl_theme)`这里设置的是图标图片和动画了，在`drawable`文件夹下,
需要的话自己复制吧，这里不贴了，动画和矢量那一堆坐标得靠美工来了。
```xml

<animated-selector xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/night"
        android:state_checked="true"
        android:drawable="@drawable/ic_theme_night" />
    <item
        android:id="@+id/day"
        android:drawable="@drawable/ic_theme_day" />
    <transition
        android:fromId="@id/night"
        android:toId="@id/day"
        android:drawable="@drawable/avd_night_to_day" />
    <transition
        android:fromId="@id/day"
        android:toId="@id/night"
        android:drawable="@drawable/avd_day_to_night" />
</animated-selector>
```

5. MD风格下，UI是有高度的，高度就可以是` android:elevation="@dimen/z_app_bar"`
6. 转场动画，俩个view设置相同的动画名称,比如`android:transitionName="@string/transition_search_back" `，然后转场的时候，`ActivityOptions.makeSceneTransitionAnimation()`将view和对应的动画名添加进去
7. 在各个模块都有其对应的数据仓库xxxRepository，里面全是kotlin 协程的suspend方法发动网络请求。然后在viewmodle里调回，viewmodle是有自己的scope的直接调`viewModelScope`即可启动协程。这个协程域会在viewmodle被clear的时候取消。
