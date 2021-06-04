---
title: Retrofit处理https请求
date: 2017-06-05 10:33:27
tags: 
- refrofit
- https
- android
categories: Android
---

### 问题描述 
一些通过CA认证的，https是可以直接访问的，但一些自签名证书，用retrofit直接访问则会走到onFailure里,错误信息是无法通过证书验证。

`onFailure: java.security.cert.CertPathValidatorException: Trust anchor for certification path not found.`
比如下面这样。

```java
        String baseUrl = "https://kyfw.12306.cn/";
        OkHttpClient.Builder clientBuilder = new OkHttpClient.Builder();
        Retrofit retrofit = new Retrofit.Builder().baseUrl(baseUrl).client(clientBuilder.build()).build();
        HttpsInterf httpsInterf = retrofit.create(HttpsInterf.class);
        Call<ResponseBody> responseBodyCall = httpsInterf.get();
        responseBodyCall.enqueue(new Callback<ResponseBody>() {
            @Override
            public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
                Log.d("ZFDT", "onResponse: " + response.body().toString());
            }

            @Override
            public void onFailure(Call<ResponseBody> call, Throwable t) {
                Log.d("ZFDT", "onFailure: " + t.getLocalizedMessage());
            }
        });
```

### 解决办法：

1. 首先把证书放到assets下面。如果你是chrome浏览器的话，请按CTRL+SHIFT+I打开开发者工具，点击Security->View certificate->详细信息->复制到文件

![image.png](http://upload-images.jianshu.io/upload_images/2524531-b099c96bbb5e6b52.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 增加下面的方法

```
/**
  * 通过okhttpClient来设置证书
  * @param clientBuilder OKhttpClient.builder
  * @param certificates 读取证书的InputStream
  */
    public void setCertificates(OkHttpClient.Builder clientBuilder, InputStream... certificates) {
        try {
            CertificateFactory certificateFactory = CertificateFactory.getInstance("X.509");
            KeyStore keyStore = KeyStore.getInstance(KeyStore.getDefaultType());
            keyStore.load(null);
            int index = 0;
            for (InputStream certificate : certificates) {
                String certificateAlias = Integer.toString(index++);
                keyStore.setCertificateEntry(certificateAlias, certificateFactory
                        .generateCertificate(certificate));
                try {
                    if (certificate != null)
                        certificate.close();
                } catch (IOException e) {
                }
            }
            TrustManagerFactory trustManagerFactory = TrustManagerFactory.getInstance(
                    TrustManagerFactory.getDefaultAlgorithm());
            trustManagerFactory.init(keyStore);
            TrustManager[] trustManagers = trustManagerFactory.getTrustManagers();
            if (trustManagers.length != 1 || !(trustManagers[0] instanceof X509TrustManager)) {
                throw new IllegalStateException("Unexpected default trust managers:"
                        + Arrays.toString(trustManagers));
            }
            X509TrustManager trustManager = (X509TrustManager) trustManagers[0];
            SSLContext sslContext = SSLContext.getInstance("TLS");
            sslContext.init(null, trustManagerFactory.getTrustManagers(), new SecureRandom());
            SSLSocketFactory sslSocketFactory = sslContext.getSocketFactory();
            clientBuilder.sslSocketFactory(sslSocketFactory, trustManager);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

3. 在建立retrofit实例前，调用上面的方法即可。
```java
		String baseUrl = "https://kyfw.12306.cn/";
        OkHttpClient.Builder clientBuilder = new OkHttpClient.Builder();
        try {
            setCertificates(clientBuilder,getAssets().open("https12306.cer"));
        } catch (IOException e) {
            e.printStackTrace();
        }

        Retrofit retrofit = new Retrofit.Builder().baseUrl(baseUrl).client(clientBuilder.build()).build();
        HttpsInterf httpsInterf = retrofit.create(HttpsInterf.class);
        Call<ResponseBody> responseBodyCall = httpsInterf.get();
        responseBodyCall.enqueue(new Callback<ResponseBody>() {
            @Override
            public void onResponse(Call<ResponseBody> call, Response<ResponseBody> response) {
                Log.d("ZFDT", "onResponse: " + response.body().toString());
            }

            @Override
            public void onFailure(Call<ResponseBody> call, Throwable t) {
                Log.d("ZFDT", "onFailure: " + t.getLocalizedMessage());
            }
        });
```



