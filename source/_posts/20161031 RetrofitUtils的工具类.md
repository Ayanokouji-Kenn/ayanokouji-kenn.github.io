---
title: RetrofitUtils的工具类
date: 2016-10-31 10:33:27
tags: Utils
---

- 直接上代码 ，一个RetrofitFactory，一个ApiFactory

  ```java
  package com.aidebar.retrofitutils.Utils.RetrofitUtils;

  import okhttp3.OkHttpClient;
  import retrofit2.Retrofit;
  import retrofit2.adapter.rxjava.RxJavaCallAdapterFactory;
  import retrofit2.converter.gson.GsonConverterFactory;

  /**
   * @author xzj
   * @date 2016/8/25 09:37.
   * 用于获取配置好的retrofit对象
   * 需要先调用setBaseUrl，如果项目中BaseUrl不变，可以写死
   */
  public class RetrofitFactory {
      private static Retrofit retrofit;
      private static String baseUrl;

      public static void setBaseUrl(String url) {
          baseUrl = url;
      }

      /**
       * 获取配置好的retrofit对象来生产Manager对象
       */
      public static Retrofit getRetrofit() {
          if (retrofit == null) {
              if (baseUrl == null || baseUrl.length() <= 0)
                  throw new IllegalStateException("请在调用getFactory之前先调用setBaseUrl");

              Retrofit.Builder builder = new Retrofit.Builder();
              builder.baseUrl(baseUrl)
                      .addCallAdapterFactory(RxJavaCallAdapterFactory.create()) // 参考RxJava
                      .addConverterFactory(GsonConverterFactory.create()); // 参考与GSON的结合

              // 参考自定义Log输出
              OkHttpClient client = new OkHttpClient().newBuilder()
  //                    .addInterceptor(new Interceptor() {     //这个拦截器是操作请求头的
  //                        @Override
  //                        public Response intercept(Chain chain) throws IOException {
  //                            Request request = chain.request().newBuilder()
  //                                    .addHeader("version", "123411") //这里就是添加一个请求头
  //                                    .build();
  //
  ////                        Buffer buffer = new Buffer();       不依赖下面的Interceptor，用这三行也能打印出请求体
  ////                        request.body().writeTo(buffer);
  ////                        Log.d(getClass().getSimpleName(), "intercept: " + buffer.readUtf8());
  //
  //                            return chain.proceed(request);
  //                        } 
  //                    })
  //                    .addInterceptor(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY))       //这个拦截器是用来打印日志的，不稳定
                      .build();
              builder.client(client);
              retrofit = builder.build();
          }
          return retrofit;
      }
  }
  ```

  ~~~java
  package com.aidebar.retrofitutils.Utils.RetrofitUtils;

  import java.util.HashMap;

  /**
   * @author xzj
   * @date 2016/8/25 09:38.
   * 通过定义好的api接口以及Retrofit来生成具体的实例.
   */
  public class ApiFactory {
      private static ApiFactory factory;
      private static HashMap<String, Object> serviceMap = new HashMap<>();

      public static ApiFactory getFactory() {
          if (factory == null) {
              synchronized (ApiFactory.class) {
                  if (factory == null)
                      factory = new ApiFactory();
              }
          }
          return factory;
      }

      public <T> T create(Class<T> clz) {
          Object service = serviceMap.get(clz.getName());
          if (service == null) {
              service = RetrofitFactory.getRetrofit().create(clz);
              serviceMap.put(clz.getName(), service);
          }
          return (T) service;
      }
  }
  ~~~

  ***

  还有2个是用rxjava进行配合的可以选用

```java
package com.aidebar.retrofitutils.Utils.RetrofitUtils;
import android.content.Context;
import android.widget.Toast;

import com.aidebar.retrofitutils.R;
import com.aidebar.retrofitutils.Utils.RetrofitUtils.JsonBean.BaseJsonBean;

import java.net.ConnectException;
import java.net.SocketTimeoutException;

import retrofit2.adapter.rxjava.HttpException;
import rx.Subscriber;

/**
 * @author xzj
 * @date 2016/8/25 11:07.
 */
public abstract class ResponseSubscriber<T> extends Subscriber<T> {
    private Context mContext;
    
    public ResponseSubscriber(Context context) {
        mContext = context;
    }

    @Override
    public void onCompleted() {

    }

    @Override
    public void onError(Throwable e) {
        if (!error(e)) {
            if (e instanceof ConnectException) {
                //网络异常
                Toast.makeText(mContext, R.string.network_error,Toast.LENGTH_SHORT).show();
            } else if (e instanceof HttpException) {
                //服务器异常
                Toast.makeText(mContext, R.string.network_servier_error,Toast.LENGTH_SHORT).show();
            } else if (e instanceof SocketTimeoutException) {
                //网络超时
                Toast.makeText(mContext, R.string.network_timeout,Toast.LENGTH_SHORT).show();
            } else {
                Toast.makeText(mContext,e.getMessage(),Toast.LENGTH_SHORT).show();
            }
        }

    }

    @Override
    public void onNext(T t) {
        BaseJsonBean data;
        if (t instanceof BaseJsonBean) {
            data = (BaseJsonBean) t;
            if (data.success) {             //服务端返回的是true
                success(t);
            } else {                        //服务端返回false，就是操作异常
                if (!operationError(t, data.errorCode, data.msg)) {    //可以复写此方法，返回true，就用户自己处理，返回false，走下面的代码
                    Toast.makeText(mContext,data.msg,Toast.LENGTH_SHORT).show();
                }
            }
        } else {
            success(t);
        }
    }

    /**
     * 请求成功同时业务成功的情况下会调用此函数
     */
    public abstract void success(T t);

    /**
     * 请求成功但业务失败的情况下会调用此函数.
     * @return 空实现，默认返回false，执行父类方法。 用户可以复写此方法，返回true来自己处理
     */
    public boolean operationError(T t, int errorCode, String message) {
        return  false;
    }

    /**
     * 请求失败的情况下会调用此函数
     * @return 空实现，默认返回false，执行父类方法。 用户可以复写此方法，返回true来自己处理
     */
    public boolean error(Throwable e) {
        return false;
    }
}

package com.aidebar.retrofitutils.Utils.RetrofitUtils;

import rx.Observable;
import rx.android.schedulers.AndroidSchedulers;
import rx.schedulers.Schedulers;

/**
 * 用于对网络请求的Observable做转换.
 * 配合{@link com.trello.rxlifecycle.ActivityLifecycleProvider#bindToLifecycle()}一起使用
 * 可以将原始Observable绑定至Activity/Fragment生命周期, 同时声明在IO线程运行, 在main线程接收.
 * 像这样用 
 * manager.getAds().compose(new ResponseTransformer<>(this.<BaseJsonBean> bindToLifeCycle()));
 */
public class ResponseTransformer<T> implements Observable.Transformer<T, T> {

    private Observable.Transformer<T, T> transformer;

    public ResponseTransformer() {}

    public ResponseTransformer(Observable.Transformer<T, T> t) {
        transformer = t;
        
    }

    @Override
    public Observable<T> call(Observable<T> source) {
        if (transformer != null)
            return transformer.call(source).subscribeOn(Schedulers.io())
                    .observeOn(AndroidSchedulers.mainThread());
        else
            return source.subscribeOn(Schedulers.io())
                    .observeOn(AndroidSchedulers.mainThread());
    }
}
```