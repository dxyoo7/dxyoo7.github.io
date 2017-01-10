---
title: Android中 MotionEvent MeasureSpec 位操作及含义
date: 2017-01-10 11:37:54
tags: Android
---


在Android中为了节省内存，有很多变量使用位存储，例如MotionEvent.getAction()，下面我们来梳理下它们的含义和使用方法。


## MotionEvent.getAction();

`MotionEvent.getAction()`得到是一个32位整型，现在可以告诉你有意义的是低16位。

1、低8位表示的是`action`动作, 例如0x00 代表 `ACTION_DOWN`。

2、高8位代表的是`point_index` 即 哪一个手指点击，例如0x0100 代表 第二个手指 `ACTION_DOWN`事件。

然后是一些常量：

```Java
public static final int ACTION_MASK = 0xff; 
public static final int ACTION_POINTER_INDEX_MASK = 0xff00; 
public static final int ACTION_POINTER_INDEX_SHIFT = 8; 
```
看源码可知：

```Java
public final int getAction() {
        return nativeGetAction(mNativePtr);
}

/**
 * 获得Action动作 
 */
public final int getActionMasked() {
        //取低8位
        return nativeGetAction(mNativePtr) & ACTION_MASK;
}

/**
 * 获得触控点索引
 */
public final int getActionIndex() {
        //取高8位，右移8位
        return (nativeGetAction(mNativePtr) & ACTION_POINTER_INDEX_MASK) >> ACTION_POINTER_INDEX_SHIFT;
}
```

## MeasureSpec 布局宽高存储

通常在自定义控件的时候我们都要重写 `onMeasure(int widthMeasureSpec, int heightMeasureSpec)` 那这个MeasueSpec是个啥意思呢。

`measureSpec`其实是父布局给子布局提供的宽高的容量，让子布局正确的计算出自己的宽高，`measureSpec`里面就携带了宽或高的参数，还有测量模式。

`measureSpec`是一个32位整型，高2位代表的是 `MeasureMode`

```Java
 private static final int MODE_SHIFT = 30;
 
  /**
  * Measure specification mode: The parent has not imposed any constraint
  * on the child. It can be whatever size it wants.
  */
 public static final int UNSPECIFIED = 0 << MODE_SHIFT;

  /**
   * Measure specification mode: The parent has determined an exact size
   * for the child. The child is going to be given those bounds regardless
   * of how big it wants to be.
   */
 public static final int EXACTLY     = 1 << MODE_SHIFT;

 /**
  * Measure specification mode: The child can be as large as it wants up
  * to the specified size.
  */
 public static final int AT_MOST     = 2 << MODE_SHIFT;
```

低30位代表宽或高的大小,看源码可知。

```Java
public static int getMode(int measureSpec) {
    //根据提供的测量值(格式)提取模式(上述三个模式之一)
    return (measureSpec & MODE_MASK);
}

public static int getSize(int measureSpec) {
    //根据提供的测量值(格式)提取大小值(这个大小也就是我们通常所说的大小)
    return (measureSpec & ~MODE_MASK);
}

public static int makeMeasureSpec(int size, int mode) {
    if (sUseBrokenMakeMeasureSpec) {
        return size + mode;
    } else {
        //根据提供的大小值和模式创建一个测量值(格式)
        return (size & ~MODE_MASK) | (mode & MODE_MASK);
    }
}
```


总结：
我们看到系统搞这么复杂的操作，往往我们使用几个变量就解决了，可是为啥系统要大费周章的使用位来存储呢？其实这正是系统的高明之处，我们知道计算机底层存储及通讯都是二机制位，而上层直接使用bit操作，可以减少内存的分配，试想下一个点击事件使用一个字节包含了所有的手势及触摸索引 比 你使用多个int存储节省多少内存呀。