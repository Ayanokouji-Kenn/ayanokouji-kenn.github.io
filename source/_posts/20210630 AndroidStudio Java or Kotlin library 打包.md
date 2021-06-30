---
title: AndroiStudio Java or Kotlin library 打包
date: 2021-06-30 13:40:27
tags: AndroiStudio
---
# 在AS中建新的java kotlin library 打包后jar无法运行
在module的`build.gradle`里加上
```
jar {
    manifest {
    // 冒号后面的是你主函数所在类名，我这Kt是因为这是个kotlin file
        attributes 'Main-Class': 'com.optimais.hidenavigationbar.MyClassKt' 
    }

    // 下面这行也要加，不然打的jar包莫的kotlin环境
    from { configurations.compileClasspath.collect { it.isDirectory() ? it : zipTree(it) } }
}

```

