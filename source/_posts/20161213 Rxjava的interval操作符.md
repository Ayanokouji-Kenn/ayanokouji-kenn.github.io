---
title: Rxjava的interval操作符
date: 2016-12-13 10:33:27
tags: Utils

---

- 有个需求重复发送，当然用timer、handler都行啦，不过要优雅还是得用rxjava啊

小demo，一个progressDialog进度条递增，涨到100，进度条消失，如下所示

```java
private ProgressDialog pd;
    private void download() {
        if (pd == null) {
            pd = new ProgressDialog(this);
            pd.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
            pd.setTitle("download new version");
            pd.setProgress(0);
            pd.setCanceledOnTouchOutside(false);

        }
        pd.show();
      //每100ms发送一次，进度递增为1，max不设置的话默认是100
        Observable.interval(100, TimeUnit.MILLISECONDS)
                  .subscribe(new Observer<Long>() {
                    @Override
                    public void onCompleted() {
                            pd.setProgress(0);
                            pd.dismiss();
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(Long aLong) {
                        pd.setProgress(pd.getProgress() + 1);
                    }
                });
    }
```

爽不爽！ 爽的根本停不下来！没错停不下来了。。。

所以要用到take操作符，这里需求是让进度条到达max就停止发送，所以用的是takeUtil

```java
private ProgressDialog pd;
    private void download() {
        if (pd == null) {
            pd = new ProgressDialog(this);
            pd.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
            pd.setTitle("download new version");
            pd.setProgress(0);
            pd.setCanceledOnTouchOutside(false);

        }
        pd.show();
        Observable.interval(100, TimeUnit.MILLISECONDS)
                .takeUntil(new Func1<Long, Boolean>() {
                    @Override
                    public Boolean call(Long aLong) {
                      //当进度条与max相同就停止发送
                        return pd.getProgress() == pd.getMax();
                    }
                })
                .subscribe(new Observer<Long>() {
                    @Override
                    public void onCompleted() {
                      //结束的时候将进度条清0，并且消失
                        pd.setProgress(0);
                        pd.dismiss();
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(Long aLong) {
                        pd.setProgress(pd.getProgress() + 1);
                    }
                });
    }
```

