---
title: 【译】Android 主题层级
date: 2016-07-08 16:23:49
tags:
---
# 【译】Android 主题层级
[https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown](https://www.zybuluo.com/mdeditor?url=https://www.zybuluo.com/static/editor/md-help.markdown)

Theme.AppCompat, Theme.Base.AppCompat, Base.V7.Theme.AppCompat, Base.v11.Theme.AppCompat, Base.v21.Theme.AppCompat, ThemeOverlay, Platform.AppCompat, DeviceDefault, Material, Holo, Classic…

假如你使用Android themes 和一些 support 库时，有时你可能会问自己：

> * 这些 Base.V{?}, Theme.Base.AppCompat Platform.AppCompat 是什么呀?
> * 这些主题是怎么组织的?
> * 我该使用哪一个?

在这篇文章中我准备回答这些问题并且尝试理清那些没人知道如何组合在一起。

## AppCompat v7

鉴于不同的安卓平台定义了不同的主题、样式和属性，最初安卓主题的层级非常繁杂，而且很不直观。直到 v7 支持库带来了全新的主题**架构**，使得所有安卓平台自 API v7 起能够获得一致的材质外观 (Matertial apperance)。*Base.V...* 和 *Platform.AppCompat* 正是在这个时候被加入了进来。 

> 在Github上有一份 README 文件，该文件解释了主题的层级关系，我建议看一下。

[https://github.com/android/platform_frameworks_support/blob/master/v7/appcompat/THEMES.txt](https://github.com/android/platform_frameworks_support/blob/master/v7/appcompat/THEMES.txt)

在 *AppCompat* 主题是四个层次的结构，每个层次继承自更低一层：

**Level1 → Level2 → Level3 → Level4**

除此之外，每个版本的安卓 API 都有一个对应的 *values-v{api}* 文件夹存放各自需要定义或覆写的样式和属性：

**values, values-v11, values-v14, values-v21, values-v22, values-v23**


## Level 4(最底层)

最底层包含了 *Platform.AppCompat* 主题。该主题总是继承自当前版本中的默认主题，例如：

**values**

*Platform.AppCompat -> android:Theme*


**values-v11**

*Platform.AppCompat -> android:Theme.Holo*


**values-v21**

*Platform.AppCompat -> android:Theme.Material*

## Level 3

大部分工作都在 **Base.V7.Theme.AppCompat, Base.V11.Theme.AppCompat, Base.V21.Theme.AppCompat**, 等的定义中完成，这层主题继承自 Platform.AppCompat

**values**

*Base.V7.Theme.AppCompat → Platform.AppCompat → android:Theme*

**values-v11**

*Base.V11.Theme.AppCompat → Platform.AppCompat → android:Theme.Holo*

**values-v21**

*Base.V21.ThemeAppCompat → Base.V7.ThemeAppCompat → Platform.AppCompat → android:Theme.Material*

> \* 还包括 Base.V7.Theme.AppCompat.Light, Base.V7.Theme.AppCompat.Dialog, 等变体。

绝大部分属性和工作都已在 Base.V{api}.Theme.AppCompat 主题中定义及完成。ActionBar, DropwDown, ActionMode, Panel, List, Spinner, Toolbar 等控件的所有主题都定义在这里，你可以通过此链接查看详细内容。
[https://github.com/android/platform_frameworks_support/blob/master/v7/appcompat/res/values/themes_base.xml](https://github.com/android/platform_frameworks_support/blob/master/v7/appcompat/res/values/themes_base.xml)

## Level 2

根据安卓的官方解释，我们在这一层拿到的主题只是第三层主题的别名：

> There are the themes which are pointers to the correct third level theme.They can also be used to set attributes for that specific platform (and platforms up until the next declaration).
这些主题指向第三层中相应的主题。它们也可以用来配置那些特定平台的属性。

这层主题指向第三层主题，他们可以被设置到指定的平台的 attributes 标签中(and platforms up until the next declaration)

**values**

Base.Theme.AppCompat* → Base.V7.Theme.AppCompat

**values-v21** 

Base.Theme.AppCompat → Base.V21.Theme.AppCompat


> 和类似于 Base.Theme.AppCompat.Light, Base.Theme.AppCompat.Dialog, 等…


## Level 1(顶层)

*Theme.AppCompat, Theme.AppCompat.Light, Theme.AppCompat.NoActionBar* **等主题在这里被定义。开发者应该使用这些主题，而非那些更底层的。**

**values**

Theme.AppCompat → Base.Theme.AppCompat

这些主题只在 values 文件夹中被定义，并根据安卓应用运行的 API 环境，继承自下层中定义的相应主题。例如：


**运行在 v7 (Android2.2)**

*Theme.AppCompat → Base.Theme.AppCompat → Base.V7.Theme.AppCompat → Platform.AppCompat → android:Theme*


**运行在 v11 (Android3.0)**

*Theme.AppCompat → Base.Theme.AppCompat → Base.V7.Theme.AppCompat → Platform.AppCompat → Platform.V11.AppCompat → android:Theme.Holo*

**运行在 v21 (Android 5.0)**

*Theme.AppCompat → Base.Theme.AppCompat → Base.V21.Theme.AppCompat → Base.V7.Theme.AppCompat → Platform.AppCompat → android:Theme.Material*


这就是你如何能够在所有Android api中得到一个统一 Material 风格，正如你所见的通过主题层级寻找是复杂的。

## Theme Diagram(simplified)

![hello-world](https://cdn-images-1.medium.com/max/800/1*iCallLnKKdoj_82nvNxfLw.png)

ThemeOverlays
在所有可用的主题中，我们可以发现一个名字带有 ThemeOverlay 的系列：

ThemeOverlay
ThemeOverlay.Light
ThemeOverlay.ActionBar.Light
ThemeOverlay.ActionBar.Dark

这些主题又是做什么的呢？答案是 仅用于为特定的用途定义必要的属性。 例如 ThemeOverlay 主题只定义了 textColor，textAppearance，窗口的颜色属性和一些类似 colorControlButton 的属性；通常用作于 Toolbar 主题的 ThemeOverlay.ActionBar.Light，仅将 colorControlButton 的值定义为 ?attr:textColorSecondary。