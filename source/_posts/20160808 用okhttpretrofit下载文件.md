---
title: 用okhtt/retrofit下载文件
date: 2016-08-08 10:33:27
tags: Utils
---
项目中需要在开屏页下载东西，在github上发现一个好用的框架
大家可以去`https://github.com/lingochamp/FileDownloader/blob/master/README-zh.md`查看
关于retrofit的用法就不多说了，**这个框架依赖于okhttp 3.4.1**
1.  首先在项目中引用
`compile 'com.liulishuo.filedownloader:library:0.3.4'`
2.  然后在Application的onCreate()中初始化
` FileDownloader.init(applicationContext); 
`
3. 在工具类中丢进这2个方法
~~~java
/**
     * 单任务下载
     */
    public static void downloadFile(String path, String url) {
        FileDownloader.getImpl().create(url)
                .setPath(path)
                .setListener(new FileDownloadListener() {
                    @Override
                    protected void pending(BaseDownloadTask task, int soFarBytes, int totalBytes) {
                    }

                    @Override
                    protected void connected(BaseDownloadTask task, String etag, boolean isContinue, int soFarBytes, int totalBytes) {
                    }

                    @Override
                    protected void progress(BaseDownloadTask task, int soFarBytes, int totalBytes) {
                    }

                    @Override
                    protected void blockComplete(BaseDownloadTask task) {
                    }

                    @Override
                    protected void retry(final BaseDownloadTask task, final Throwable ex, final int retryingTimes, final int soFarBytes) {
                    }

                    @Override
                    protected void completed(BaseDownloadTask task) {
                    }

                    @Override
                    protected void paused(BaseDownloadTask task, int soFarBytes, int totalBytes) {
                    }

                    @Override
                    protected void error(BaseDownloadTask task, Throwable e) {
                    }

                    @Override
                    protected void warn(BaseDownloadTask task) {
                    }
                }).start();
    }

    /**
     * 多任务下载
     * 参数1：url的集合   参数2：下载的路径
     */

    public static void downloadFiles(List<RingListJson.RingBean> urls,File dir) {
        final FileDownloadListener downloadListener = new FileDownloadListener() {
            @Override
            protected void pending(BaseDownloadTask task, int soFarBytes, int totalBytes) {
            }

            @Override
            protected void connected(BaseDownloadTask task, String etag, boolean isContinue, int soFarBytes, int totalBytes) {
            
            }

            @Override
            protected void progress(BaseDownloadTask task, int soFarBytes, int totalBytes) {
            }

            @Override
            protected void blockComplete(BaseDownloadTask task) {
            }

            @Override
            protected void retry(final BaseDownloadTask task, final Throwable ex, final int retryingTimes, final int soFarBytes) {
            }

            @Override
            protected void completed(BaseDownloadTask task) {
            
            }

            @Override
            protected void paused(BaseDownloadTask task, int soFarBytes, int totalBytes) {
            }

            @Override
            protected void error(BaseDownloadTask task, Throwable e) {
             }

            @Override
            protected void warn(BaseDownloadTask task) {
            }
        };

        final FileDownloadQueueSet queueSet = new FileDownloadQueueSet(downloadListener);

        final List<BaseDownloadTask> tasks = new ArrayList<>();
        for (int i = 0; i < urls.size(); i++) {
            if (FileDownloader.getImpl().create((urls.get(i).url)).isReusedOldFile()) {
                continue;
            }

            tasks.add(FileDownloader.getImpl()
                    .create(urls.get(i).url)
                    .setTag(i + 1)
                    .setPath(dir.getAbsolutePath()+ File.separator+urls.get(i).name.concat(".aac")));

        }
    /*由于是队列任务, 这里是我们假设了现在不需要每个任务都回调`FileDownloadListener#progress`, 
      我们只关系每个任务是否完成, 
      所以这里这样设置可以很有效的减少ipc.*/
        queueSet.disableCallbackProgressTimes(); 

        // 所有任务在下载失败的时候都自动重试一次
        queueSet.setAutoRetryTimes(1);

        queueSet.downloadTogether(tasks);
        queueSet.start();

    }
~~~
4. 在需要下载的地方调用者2个方法即可
```java
Retrofit retrofit = new Retrofit.Builder().baseUrl("http:/xxx.xxx.com").addConverterFactory(GsonConverterFactory.create()).build();
        RingListResult ringListResult = retrofit.create(RingListResult.class);
        Call<RingListJson> call = ringListResult.getRings();
        
            final File dir = D8Application.applicationContext.getFilesDir();
            final File rings = new File(dir + "/rings");
            if (!rings.exists()) {
                rings.mkdirs();
            }
        call.enqueue(new Callback<RingListJson>() {
            @Override
            public void onResponse(Call<RingListJson> call, Response<RingListJson> response) {
                RingListJson ringListJson = response.body();
                List<RingListJson.RingBean> ringList = ringListJson.ring; //在服务器拿到url的集合
                Api.downloadFiles(ringList,rings);    //传进方法里  Api是自己的工具类
            }

            @Override
            public void onFailure(Call<RingListJson> call, Throwable t) {

            }
        });
```