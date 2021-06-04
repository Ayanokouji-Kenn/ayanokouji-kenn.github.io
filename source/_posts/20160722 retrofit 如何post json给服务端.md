---
title: retrofit如何post json给服务端
date: 2016-09-02 10:33:27
tags: problems
---
- 需求：
开发新项目时，拿到接口文档，需要请求消息体是json类型的
---
可能你这么写过post：
```java
interface NService {
        @FormUrlEncoded
        @POST("alarmclock/add.json")
        Call<ResponseBody> getResult(@FieldMap Map<String, Object> params);
    }
```
   ```java
Retrofit retrofit = new Retrofit.Builder().baseUrl(URL).client(client).build();
        NService nService = retrofit.create(NService.class);
        Map<String, Object> params = new HashMap<>();
        params.put("id", "123");
        params.put("name", "ksi");

        Call<ResponseBody> call = nService.getResult(params);
        call.enqueue(new Callback<ResponseBody>() {
            @Override
            public void onResponse(Call<ResponseBody> call, retrofit2.Response<ResponseBody> response) {

            }

            @Override
            public void onFailure(Call<ResponseBody> call, Throwable t) {

            }
        });
```
这是表单提交，你提交上去的其实是`id=123&name=ksi`这么个东西。
如果要提交的是json那么自然要改变请求体了

好，有的同学可能会搜索以下问题：**怎么查看/更改/添加请求头、请求体、响应体**？
我的版本是：retrofit2.1.0，2.0以前的做法可能不一样。

首先，在你的build.gradle下面依赖这玩意
`compile 'com.squareup.okhttp3:logging-interceptor:3.4.1'`
然后配置client，添加拦截器，第一个拦截器是用于添加请求头的，第二个就是打印日志了
```java
OkHttpClient client = new OkHttpClient().newBuilder()
                .addInterceptor(new Interceptor() {
                    @Override
                    public Response intercept(Chain chain) throws IOException {
                        Request request = chain.request().newBuilder()
                                .addHeader("creater_oid", "123411") //这里就是添加一个请求头
                                .build();

//                        Buffer buffer = new Buffer();       不依赖logging，用这三行也能打印出请求体
//                        request.body().writeTo(buffer);
//                        Log.d(getClass().getSimpleName(), "intercept: " + buffer.readUtf8());

                        return chain.proceed(request);
                    } //下面是关键代码
                }).addInterceptor(new HttpLoggingInterceptor().setLevel(HttpLoggingInterceptor.Level.BODY))
                .build();
```

好，我们来干正经事了，json格式的请求，参数注解用@Body
```java
interface ApiService {
        @POST("add.json")
        Call<ResponseBody> add(@Body RequestBody body);
    }
```
```java
Retrofit retrofit = new Retrofit.Builder().baseUrl(URL).client(client).build();
        ApiService apiService = retrofit.create(ApiService.class);

//new JSONObject里的getMap()方法就是返回一个map，里面包含了你要传给服务器的各个键值对，然后根据接口文档的请求格式，直接拼接上相应的东西就行了
//比如{"data":{这里面是参数}}，那就在外面拼上大括号和"data"好了
        RequestBody requestBody = RequestBody.create(MediaType.parse("Content-Type, application/json"),
                                   "{\"data\":"+new JSONObject(getMap()).toString()+"}");
        Call<ResponseBody> call = apiService.add(requestBody);
        call.enqueue(new Callback<ResponseBody>() {
            @Override
            public void onResponse(Call<ResponseBody> call, retrofit2.Response<ResponseBody> response) {
                Log.d(getClass().getSimpleName(), "onResponse: ----" + response.body().toString());
            }

            @Override
            public void onFailure(Call<ResponseBody> call, Throwable t) {
                Log.d(getClass().getSimpleName(), "onFailure: ------" + t.toString());
            }
        });
```
OK，大功告成，来看看打印结果吧


![QQ截图20160722012756.png](http://upload-images.jianshu.io/upload_images/2524531-612f8177ff844aa4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
看到第三行了么，那就是自定义添加的请求头，第四行就是json格式的请求体了
<---200 OK下面是响应体。