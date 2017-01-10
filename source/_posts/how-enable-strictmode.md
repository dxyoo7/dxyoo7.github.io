---
title: 启用 Android_StrictMode
date: 2016-11-01 17:21:31
tags: Android
---
##StrictMode 严格模式
StrictMode用来基于线程或VM设置一些策略, 一旦检测到策略违例, 控制台将输出一些警告，包含一个trace信息展示你的应用在何处出现问题.

通常用来检测主线程中的磁盘读写或网络访问等耗时操作.

在Application或是Activity的onCreate中开启StrictMode:

```java
if (Constant.DEBUG) {
	// 针对线程的相关策略
   StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
             .detectDiskReads()
             .detectDiskWrites()
             .detectNetwork()   // or .detectAll() for all detectable problems
             .penaltyLog()
             .build());

            // 针对VM的相关策略
    StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
              .detectLeakedSqlLiteObjects()
              .detectLeakedClosableObjects()
              .penaltyLog()
              .penaltyDeath()
              .build());
        }
```

