---
title: Android分包MultiDex原理详解
date: 2016-03-11 17:37:54
tags: Android
---

## 介绍

Android app 迭代到一定的程度(引入第三方库，代码量太多）时，就很容易踩到系统的一个坑：

```Java
Unable to execute dex: method ID not in [0, 0xffff]: 65536
Conversion to Dalvik format failed: Unable to execute dex: method ID not in [0, 0xffff]: 65536
```

能踩到这个坑的基本上你的app已经很大了，或许你已经通过[stackoverflow][1]或[google][2]找到了解决方案，
但是我们coder必须是知其然知其所以然，下面我们就到清这个坑的前世今生：


## MultiDex

当Android系统安装一个应用的时候，有一步是对Dex进行优化，这个过程有一个专门的工具来处理，叫DexOpt。DexOpt的执行过程是在第一次加载Dex文件的时候执行的。这个过程会生成一个ODEX文件，即Optimised Dex。执行ODex的效率会比直接执行Dex文件的效率要高很多。

但是在早期的Android系统中，DexOpt有一个问题，DexOpt会把每一个类的方法id检索起来，存在一个链表结构里面。但是这个链表的长度是用一个short类型来保存的，导致了方法id的数目不能够超过65536个。当一个项目足够大的时候，显然这个方法数的上限是不够的。尽管在新版本的Android系统中，DexOpt修复了这个问题，但是我们仍然需要对低版本的Android系统做兼容。

为了解决方法数超限的问题，需要将该dex文件拆成两个或多个，为此谷歌官方推出了multidex兼容包，配合AndroidStudio实现了一个APK包含多个dex的功能。


## MultiDex的简要原理
我们以APK中有两个dex文件为例，第二个dex文件为classes2.dex。

兼容包在Applicaion实例化之后，会检查系统版本是否支持 multidex，classes2.dex是否需要安装。
如果需要安装则会从APK中解压出classes2.dex并将其拷贝到应用的沙盒目录下。
通过反射将classes2.dex注入到当前的classloader中。
下面引入一下官方的文档。

https://developer.android.com/tools/building/multidex.html#about


## 关于65K方法限制

我们知道Android中的可执行伟剑都存储在dex文件中，其中包含已编译的代码来运行你的应用程序。Dalvik虚拟机对可执行dex文件的规格是有方法限制的，即一个单一的dex文件的方法总数最多为65536。

其中包括：

> * 引用的Android Framework方法
> * library的方法
> * 我们自己书写代码的方法。

为了突破这个方法数的限制，我们就提出了一个方案——生成多个dex文件。这个多个dex文件的方案，我们又称为multidex方案配置。

Multidex支持Android 5.0之前的版本Android5.0版本的平台之前，Android使用的是Dalvik Runtime执行的程序代码。默认情况下，限制应用到一个单一的classes.dex。

Dalvik字节码文件每APK。为了绕过这个限制，你可以使用multidex支持库，成为你的应用程序的主要部分和DEX文件进行管理，获得额外的dex文件，它们包含的代码。

Multidex支持Android 5.0及更高版本Android 5.0和更高的Runtime 如art，本身就支持从应用的APK文件加载多个DEX文件。art支持预编译的应用程序在安装时扫描类（..）。Dex文件编译成一个单一的Android设备上执行.oat文件。


## 避免65K限制

当你确定使用multidex的分包策略的时候，请你先确定自己的代码中都是优秀的。你还需要做以下几步：

> * 去掉一些未使用的import和library
> * 使用ProGuard去掉一些未使用的代码


## 用Gradle配置使用Multidex

Android 的 Gradle插件在 Android Build Tool 21.1开始就支持使用multidex了。

设置你的应用程序开发项目中使用multidex配置，要求你做出一些修改您的应用程序开发项目：

> * 修改Gradle的配置，支持multidex
> * 修改你的manifest。让其支持multidexapplication类

修改Gradle的build如下：

``` Groovy
android {
	compileSdkVersion 21
	buildToolsVersion "21.1.0"
	defaultConfig {         
	    minSdkVersion 14         
	    targetSdkVersion 21         
	    // Enabling multidex support.         
	    multiDexEnabled true     
	}     
    
    dependencies {
        compile 'com.android.support:multidex:1.0.0' 
    }
}
```

**Tips: 你可以在Gradle配置文件中的 multiDexEnabled 在 defaultConfig、buildType、productFlavor选项设置。**

在manifest文件中，添加MultidexApplication Class的引用，如下所示：

``` Xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.android.multidex.myapplication">
    <application
        ...
        android:name="android.support.multidex.MultiDexApplication">
        ...
    </application>
</manifest>
```

当然，如果你重写了 Application，就对自定义Application的继承方式做一个修改。

## Multidex的方式的局限性

虽然我们开起来multidex是一个极好的东西，但是multidex还是存在自己的局限性，我们在开发测试之前要清楚局限性是什么：

如果二DEX文件太大，安装分割dex文件是一个复杂的过程，可能会导致应用程序无响应（ANR）的错误。在这种情况下，你应该尽量的减小dex
文件的大小和删除无用的逻辑，而不是完全依赖于multidex。
在Android 4.0设备（API Level 14）之前，由于Dalvik linearalloc bug（问题22586），multidex很可能是无法运行的。
如果希望运行在Level 14之前的Android系统版本，请先确保完整的测试和使用。
应用程序使用了multiedex配置的，会造成使用比较大的内存。当然，可能还会引起dalvik虚拟机的崩溃(issue 78035)。
对于应用程序比较复杂的，存在较多的library的项目。multidex可能会造成不同依赖项目间的dex文件函数相互调用，找不到方法。


## 优化multidex开发和构建

一个multidex的配置，对系统apk的构建、签名、打包复杂性大大的增加。这就意味着，你每一次的构建过程都是相当耗时的。

为了加快我们的开发速度，加快构建的过程，我们可以在Gradle productFlavors新建出来一个 development flavor 和 production flavor 来满足我们不同构建需求。

下面是一个列子演示我们如何设置这些flavors在Gradle build文件中:

``` Groovy
android {
    productFlavors {
        // Define separate dev and prod product flavors.
        dev {
            // dev utilizes minSDKVersion = 21 to allow the Android gradle plugin
            // to pre-dex each module and produce an APK that can be tested on
            // Android Lollipop without time consuming dex merging processes.
            minSdkVersion 21
        }
        prod {
            // The actual minSdkVersion for the application.
            minSdkVersion 14
        }
    }
          ...
    buildTypes {
        release {
            runProguard true
            proguardFiles getDefaultProguardFile(‘proguard-android.txt‘),
                                                 ‘proguard-rules.pro‘
        }
    }
}
dependencies {
  compile ‘com.android.support:multidex:1.0.0‘
}
```

在你完成了上处的配置修改之后，你配置productFlavor 和 buildType来使用 ，devDebug 变种app。使用这些变种app，可以设置proguard disable、multidex enable方便我们测试。

这些配置需要针对Android Gradle插件做如下操作：

在分包前，编译应用程序中的每一个module包括依赖项目，这个步骤称为 pre-dexing。

> * include每一个dex文件
> * 最重要的是，对于主dex文件，不会做切分。以保证计算速度。
> * 这样设置既能够保证我们的最终报是一个使用了multidex模式的，而又不影响我们平时开发的测试效率。