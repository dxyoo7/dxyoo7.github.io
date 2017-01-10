---
title: Java(Android) 约定规范
date: 2016-11-01 17:21:31
tags: Android
---

# 1.项目指南

## 1.1项目结构

新项目应该遵循[Android Gradle插件用户指南](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Project-Structure)中定义的Android Gradle项目结构。

## 1.2文件命名

### 1.2.1类命名

类命名按 **驼峰命名法 UpperCamelCase** 规则书写.

对于继承自Android组件的类，应该以组件名结尾；比如：SignInActivity, SignInFragment, ImageUploaderService, ChangePasswordDialog.

### 资源文件

资源文件按 **小写_下划线** 规则书写

#### 1.2.2.1 Drawable 文件

drawables 命名约定：


| Asset Type   | Prefix            |		Example               |
|--------------| ------------------|-----------------------------|
| Action bar   | `ab_`             | `ab_stacked.9.png`          |
| Button       | `btn_`	            | `btn_send_pressed.9.png`    |
| Dialog       | `dialog_`         | `dialog_top.9.png`          |
| Divider      | `divider_`        | `divider_horizontal.9.png`  |
| Icon         | `ic_`	            | `ic_star.png`               |
| Menu         | `menu_	`           | `menu_submenu_bg.9.png`     |
| Notification | `notification_`	| `notification_bg.9.png`     |
| Tabs         | `tab_`            | `tab_pressed.9.png`         |

icon的命名约定(取自[Android iconography guidelines](http://developer.android.com/design/style/iconography.html)):

| Asset Type                      | Prefix             | Example                      |
| --------------------------------| ----------------   | ---------------------------- |
| Icons                           | `ic_`              | `ic_star.png`                |
| Launcher icons                  | `ic_launcher`      | `ic_launcher_calendar.png`   |
| Menu icons and Action Bar icons | `ic_menu`          | `ic_menu_archive.png`        |
| Status bar icons                | `ic_stat_notify`   | `ic_stat_notify_msg.png`     |
| Tab icons                       | `ic_tab`           | `ic_tab_recent.png`          |
| Dialog icons                    | `ic_dialog`        | `ic_dialog_info.png`         |

状态选择器(selector states)命名约定:

| State	       | Suffix          | Example                     |
|--------------|-----------------|-----------------------------|
| Normal       | `_normal`       | `btn_order_normal.9.png`    |
| Pressed      | `_pressed`      | `btn_order_pressed.9.png`   |
| Focused      | `_focused`      | `btn_order_focused.9.png`   |
| Disabled     | `_disabled`     | `btn_order_disabled.9.png`  |
| Selected     | `_selected`     | `btn_order_selected.9.png`  |


#### 1.2.2.2 layout 文件

布局文件应该与它们打算用于的Android组件的名称匹配，但将顶层组件名称移动到开头。 例如，如果我们为`SignInActivity`创建一个布局，布局文件的名称应该是`activity_sign_in.xml`

| Component        | Class Name             | Layout Name                   |
| ---------------- | ---------------------- | ----------------------------- |
| Activity         | `UserProfileActivity`  | `activity_user_profile.xml`   |
| Fragment         | `SignUpFragment`       | `fragment_sign_up.xml`        |
| Dialog           | `ChangePasswordDialog` | `dialog_change_password.xml`  |
| AdapterView item | ---                    | `item_person.xml`             |
| Partial layout   | ---                    | `partial_stats_bar.xml`       |

一个稍微不同的情况是，当我们创建一个布局将被一个`Adapter`渲染，例如填充一个 `ListView`。 在这种情况下，布局的名称应以`item_`开头。

请注意，有些情况下，这些规则将无法应用。 例如，当创建旨在作为其他布局的一部分的布局文件时。 在这种情况下，您应该使用前缀`partial_`。

#### 1.2.2.3 Menu 文件

与布局文件类似，Menu 文件应与组件的名称匹配。 例如，如果我们定义将要在`UserActivity`中使用的菜单文件，那么文件的名称应该是`activity_user.xml`

一个好的做法是不要将“menu”一词作为名称的一部分，因为这些文件已经位于`menu`目录中。

#### 1.2.2.4 Values 文件

values文件夹中的资源文件应为 __复数__ ，例如 `strings.xml`，`styles.xml`，`colors.xml`，`dimensions.xml`，`attrs.xml`

# 2 代码指南

## 2.1 Java 语法规则

### 2.1.1 不要忽略异常

你永远不要这样做:

```java
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) { }
}
```
也许你会认为：你的代码永远不会碰到这种出错的情况，或者处理异常并不重要，可类似上述忽略异常的代码将会在代码中埋下一颗地雷，说不定哪天它就会炸到某个人了。你必须在代码中以某种规矩来处理所有的异常。根据情况的不同，处理的方式也会不一样。

_无论何时，空的catch语句都会让人感到不寒而栗。虽然很多情况下确实是一切正常，但至少你不得不去忧虑它。在Java中你无法逃离这种恐惧感。_ -[James Gosling](http://www.artima.com/intv/solid4.html)

可接受的替代方案包括（按照推荐顺序）：

* 向方法的调用者抛出异常。

```java
void setServerPort(String value) throws NumberFormatException {
    serverPort = Integer.parseInt(value);
}
```

* 根据抽象级别抛出新的异常。

```java
void setServerPort(String value) throws ConfigurationException {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        throw new ConfigurationException("Port " + value + " is not valid.");
    }
}
```

* 默默地处理错误并在catch {}语句块中替换为合适的值。

```java
/** Set port. If value is not a valid number, 80 is substituted. */

void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        serverPort = 80;  // default port for server
    }
}
```

* 捕获异常并抛出一个新的RuntimeException。这种做法比较危险：只有确信发生该错误时最合适的做法就是崩溃，才会这么做。

```java
/** Set port. If value is not a valid number, die. */

void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        throw new RuntimeException("port " + value " is invalid, ", e);
    }
}
```
> 请记住，最初的异常是传递给构造方法的RuntimeException。如果代码必须在Java 1.3版本下编译，需要忽略该异常

* 最后一招：如果确信忽略异常比较合适，那就忽略吧，但必须把理想的原因注释出来：

```java
/** If value is not a valid number, original port number is used. */
void setServerPort(String value) {
    try {
        serverPort = Integer.parseInt(value);
    } catch (NumberFormatException e) {
        // Method is documented to just ignore invalid user input.
        // serverPort will just be unchanged.
    }
}
```

### 2.1.2 不要捕获顶级的Exception

有时在捕获Exception时偷懒也是很吸引人的，类似如下的处理方式：

```java
try {
    someComplicatedIOFunction();        // may throw IOException
    someComplicatedParsingFunction();   // may throw ParsingException
    someComplicatedSecurityFunction();  // may throw SecurityException
    // phew, made it all the way
} catch (Exception e) {                 // I'll just catch all exceptions
    handleError();                      // with one generic handler!
}
```

绝大部分情况下，捕获顶级的Exception或Throwable都是不合适的，Throwable更不合适，因为它还包含了Error异常。这种捕获非常危险。这意味着本来不必考虑的Exception（包括类似ClassCastException的RuntimeException）被卷入到应用程序级的错误处理中来。这会让代码运行的错误变得模糊不清。这意味着，假如别人在你调用的代码中加入了新的异常，编译器将无法帮助你识别出各种不同的错误类型。绝大部分情况下，无论如何你都不应该用同一种方式来处理各种不同类型的异常。

本规则也有极少数例外情况：期望捕获所有类型错误的特定的测试代码和顶层代码（为了阻止这些错误在用户界面上显示出来，或者保持批量工作的运行）。这种情况下可以捕获顶级的Exception（或Throwable）并进行相应的错误处理。在开始之前，你应该非常仔细地考虑一下，并在注释中解释清楚为什么这么做是安全的。

比捕获顶级Exception更好的方案：

* 分开捕获每一种异常，在一条try语句后面跟随多个catch 语句块。这样可能会有点别扭，但总比捕获所有Exception要好些。请小心别在catch语句块中重复执行大量的代码。

* 重新组织一下代码，使用多个try块，使错误处理的粒度更细一些。把IO从解析内容的代码中分离出来，根据各自的情况进行单独的错误处理。

* 再次抛出异常。很多时候在你这个级别根本就没必要捕获这个异常，只要让方法抛出该异常即可。

请记住：异常是你的朋友！当编译器指出你没有捕获某个异常时，请不要皱眉头。而应该微笑：编译器帮助你找到了代码中的运行时（runtime）问题。

### 2.1.3 不要使用 finalizers

Finalizer提供了一个机会，可以让对象被垃圾回收器回收时执行一些代码。

优点：便于执行清理工作，特别是针对外部资源。

缺点：调用finalizer的时机并不确定，甚至根本就不会调用。

结论：我们不要使用finalizers。大多数情况下，可以用优秀的异常处理代码来执行那些要放入finalizer的工作。如果确实是需要使用finalizer，那就定义一个close()方法（或类似的方法），并且在文档中准确地记录下需要调用该方法的时机。相关例程可以参见InputStream。这种情况下还是适合使用finalizer的，但不需要在finalizer中输出日志信息，因为日志不能因为这个而被撑爆。

### 2.1.4 使用完全限定Import
当需要使用foo包中的Bar类时，存在两种可能的import方式：

*   `import foo.*;`

     有点：可能会减少import语句。

*   `import foo.Bar;`

     优点：实际用到的类一清二楚。代码的可读性更好，便于维护。

结论：用后一种写法来`import`所有的Android代码。不过导入java标准库(`java.util.*`、`java.io.*`等) 和单元测试代码 (`junit.framework.*`)时可以例外。

## 2.2 Java类库规范
使用Android Java类库和工具存在一些惯例。有时这些惯例会作出重大变化，可之前的代码也许会用到过时的模板或类库。如果用到这部分过时的代码，沿用已有的风格就是了（参阅Consistency)。创建新的组件时就不要再使用过时的类库了。

## 2.3 Java编程规范
### 2.3.1 使用Javadoc标准注释
每个文件的开头都应该有一句版权说明。然后下面应该是package包语句和import语句，每个语句块之间用空行分隔。然后是类或接口的定义。在Javadoc注释中，应描述类或接口的用途。

```java
/*
 * Copyright (C) 2015 The Android Open Source Project
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.android.internal.foo;

import android.os.Blah;
import android.view.Yada;

import java.sql.ResultSet;
import java.sql.SQLException;

/**
 * Does X and Y and provides an abstraction for Z.
 */

public class Foo {
    ...
}
```
每个类和自建的public方法必须包含Javadoc注释，注释至少要包含描述该类或方法用途的语句。并且该语句应该用第三人称的动词形式来开头。

例如：

```java
/** Returns the correctly rounded positive square root of a double value. */
static double sqrt(double a) {
    ...
}
```
或者

```java
/**
 * Constructs a new String by converting the specified array of
 * bytes using the platform's default character encoding.
 */
public String(byte[] bytes) {
    ...
}
```

如果所有的Javadoc都会写成“sets Foo”，对于那些无关紧要的类似setFoo()的get和set语句是不必撰写Javadoc的。如果方法执行了比较复杂的操作（比如执行强制约束或者产生很重要的副作用），那就必须进行注释。如果“Foo”属性的意义不容易理解，也应该进行注释。

无论是public的还是其它类型的，所有自建的方法都将受益于Javadoc。public的方法是API的组成部分，因此更需要Javadoc。

Android目前还没有规定自己的Javadoc注释撰写规范，但是应该遵守[Sun Javadoc约定](http://www.oracle.com/technetwork/java/javase/documentation/index-137868.html)。

### 2.3.2 编写简短的方法
为了把规模控制在合理范围内，方法应该保持简短和重点突出。不过，有时较长的方法也是合适的，所以对方法的代码长度并没有硬性的限制。如果方法代码超过了40行，就该考虑是否可以在不损害程序结构的前提下进行分拆。

### 2.3.4 在标准的位置定义字段
字段应该定义在文件开头，或者紧挨着使用这些字段的方法之前。


### 2.2.1 属性的定义你和命名

属性应该定义在 __文件的顶部__ 并且他们应该遵循以下规则：

* Private （私有）, non-static（非静态）属性名以  __m__ 开头.
* Private （私有）, static（静态） 属性以 __s__ 开头.
* 其它属性以小写字母开头.
* final static (常量) 全部大写 ALL\_CAPS\_WITH\_UNDERSCORES.

Example:

```java
public class MyClass {
    public static final int SOME_CONSTANT = 42;
    public int publicField;
    private static MyClass sSingleton;
    int mPackagePrivate;
    private int mPrivate;
    protected int mProtected;
}
```

### 2.2.3 将缩略词作为单词

| Good           | Bad            |
| -------------- | -------------- |
| `XmlHttpRequest` | `XMLHTTPRequest` |
| `getCustomerId`  | `getCustomerID`  |
| `String url`     | `String URL`     |
| `long id`        | `long ID`        |

### 2.2.4 使用空格缩进

使用 __4 个空格__ 缩进下面的代码块:

```java
if (x == 1) {
    x++;
}
```

使用 __8 个空格__ 行格式对齐缩进:

```java
Instrument i =
        someLongExpression(that, wouldNotFit, on, one, line);
```

### 2.2.5 使用标准括号风格

大括号与它们之前的代码在同一行上.

```java
class MyClass {
    int func() {
        if (something) {
            // ...
        } else if (somethingElse) {
            // ...
        } else {
            // ...
        }
    }
}
```

需要在语句周围括号，除非条件和body块适合一行。

如果条件和body块同一行，同时这行的长度小于最大行长，那么括号不不需要，比如：

```java
if (condition) body();
```

 __不好的做法__:

```java
if (condition)
    body();  // bad!
```

### 2.2.6 注解

#### 2.2.6.1 注解实践

根据 安卓代码风格指南，Java里面有些预定义注释的标准实践。

* `@Deparected` 无论何时一个元素被标注废弃，必须使用 `@Deparected`. If you use the @Deprecated annotation, you must also have a @deprecated Javadoc tag and it should name an alternate implementation. In addition, remember that a @Deprecated method is still supposed to work. If you see old code that has a @deprecated Javadoc tag, please add the @Deprecated annotation.

* `@Override`: @Override 注解 __必须使用__ , 当一个方法重载或实现它父类方法的时候. 例如, 如果使用一个 @inheritdocs Javadoc 标签, 从一个类（不是一个接口）派生，你还必须注释该方法@Overrides 父类的方法。

* `@SuppressWarnings`: The @SuppressWarnings annotation should only be used under circumstances where it is impossible to eliminate a warning. If a warning passes this "impossible to eliminate" test, the @SuppressWarnings annotation must be used, so as to ensure that all warnings reflect actual problems in the code.

More information about annotation guidelines can be found [here](http://source.android.com/source/code-style.html#use-standard-java-annotations).

#### 2.2.6.2 Annotations style

__Classes, Methods and Constructors__

When annotations are applied to a class, method, or constructor, they are listed after the documentation block and should appear as __one annotation per line__ .

```java
/* This is the documentation block about the class */
@AnnotationA
@AnnotationB
public class MyAnnotatedClass { }
```

__Fields__

Annotations applying to fields should be listed __on the same line__, unless the line reaches the maximum line length.

```java
@Nullable @Mock DataManager mDataManager;
```

### 2.2.7 限制变量的作用范围

局部变量的作用范围应该是限制为最小的（Effective Java第29条）。使用局部变量，可以增加代码的可读性和可维护性，并且降低发生错误的可能性。每个变量都应该在最小范围的代码块中进行声明，该代码块的大小只要能够包含所有对该变量的使用即可。

应该在第一次用到局部变量的地方对其进行声明。几乎所有局部变量声明都应该进行初始化。如果还缺少足够的信息来正确地初始化变量，那就应该推迟声明，直至可以初始化为止 - ([Android code style guidelines](https://source.android.com/source/code-style.html#limit-variable-scope))

### 2.2.8 对Import语句排序

如果你使用IDE，比如Android Studio，你不用担心因为你的IDE已经遵守这些规则了。如果不是，请看下面。

import语句的次序应该如下：

1. Android imports
2. Imports from third parties (com, junit, net, org)
3. java and javax
4. Same project imports

为了精确匹配IDE的配置，import顺序应该是：

* 在每组内部按字母排序，大写字母排在小写字母的前面。
* 每个大组之间应该空一行（android、com、junit、net、org、java、javax）。

更多 [点这](https://source.android.com/source/code-style.html#limit-variable-scope)

### 2.2.9 Logging 使用

使用`Log`类提供的方法来打印输出错误或者其它的信息，帮助我们定位问题。

* `Log.v(String tag, String msg)` (verbose)
* `Log.d(String tag, String msg)` (debug)
* `Log.i(String tag, String msg)` (information)
* `Log.w(String tag, String msg)` (warning)
* `Log.e(String tag, String msg)` (error)

一个通用的规则，我们使用 `static final` 类名定义tag。例如:

```java
public class MyClass {
    private static final String TAG = MyClass.class.getSimpleName();

    public myMethod() {
        Log.e(TAG, "My error message");
    }
}
```

VERBOSE 和 DEBUG 日志 __必须__ 在release 版本中关掉。也推荐关掉 INFORMATION, WARNING and ERROR 级别的日志，但是你可能想要保持打开为了在release 版本中定位问题。如果你决定打开他们，你必须保证不要再日子里泄漏一些敏感的信息，比如email地址，用户id等。

只会在debug版本中显示的日志：

```java
if (BuildConfig.DEBUG) Log.d(TAG, "The value of x is " + x);
```

### 2.2.10 类成员的排序

这不是唯的答案，但是使用一个 __逻辑性__ 和 __一致性__ 的排序将改善你代码的可读性和可学性。推荐使用一下排序

1. Constants(常量)
2. Fields(成员)
3. Constructors(构造)
4. Override methods and callbacks (public or private)(重写方法和回调)
5. Public methods (公共方法)
6. Private methods (私有方法)
7. Inner classes or interfaces (内部类和接口)

Example:

```java
public class MainActivity extends Activity {

	 private String mTitle;
    private TextView mTextViewTitle;

    public void setTitle(String title) {
    	mTitle = title;
    }

    @Override
    public void onCreate() {
        ...
    }

    private void setUpView() {
        ...
    }

    static class AnInnerClass {

    }

}
```
如果你的类继承一个 __Android组件__ 例如一个Activity 或Fragment，一个好的做法是 __匹配组件的生命周期__ 排序重写方法。比如你继承Activity实现了`onCreate()`, `onDestroy()`, `onPause()` 和 `onResume()`, 那么正确的做法是：


```java
public class MainActivity extends Activity {

	//Order matches Activity lifecycle
    @Override
    public void onCreate() {}

    @Override
    public void onResume() {}

    @Override
    public void onPause() {}

    @Override
    public void onDestroy() {}

}
```

### 2.2.11 方法中参数的排序
当你编写Android程序时，一个常见定义方法是写一个 `Context` 参数，如果你正在写一个这样的方法，必须将 `Context` 放在第一个参数。

相反 __callback__ 接口应该永远放在最后一个参数。

例如:

```java
// Context always goes first
public User loadUser(Context context, int userId);

// Callbacks always go last
public void loadUserAsync(Context context, int userId, UserCallback callback);
```

### 2.2.13 字符常量、命名、取值
在Andoid SDK中有许多使用了类似于健值对(key-value pair)的元素，像 `SharedPreferences`, `Bundle`, 和 `Intent`，这样及时一个很小的app也要定义许多字符常量。

当使用这些组件，你 __必须__ 定义很多 `static final` 的 key，你应该使用下面这些前缀表示他们：

| Element            | Field Name Prefix |
| -----------------  | ----------------- |
| SharedPreferences  | `PREF_`             |
| Bundle             | `BUNDLE_`           |
| Fragment Arguments | `ARGUMENT_`         |
| Intent Extra       | `EXTRA_`            |
| Intent Action      | `ACTION_`           |

注意Fragment 的参数-`Fragment.getArguments()`-其实也是一个Bundle。然而，更准确的使用Bundle，我们应该定义不同的前缀区分它们，像上面那样。

例如:

```java
// Note the value of the field is the same as the name to avoid duplication issues
static final String PREF_EMAIL = "PREF_EMAIL";
static final String BUNDLE_AGE = "BUNDLE_AGE";
static final String ARGUMENT_USER_ID = "ARGUMENT_USER_ID";

// Intent-related items use full package name as value
static final String EXTRA_SURNAME = "com.myapp.extras.EXTRA_SURNAME";
static final String ACTION_OPEN_USER = "com.myapp.action.ACTION_OPEN_USER";
```

### 2.2.14 Fragments Activities 中的参数

当数据被传递到 `Activity` 或 `Fragment` 通过一个 `Intent` 或 `Bundle`，传不同的值 __必须__ 按照下面描述的规则执行。

当一个 `Activity` 或 `Fragment` 携带参数时，应该提供一个 `publi static` 方法有利于创建相关的 `Intent` 或 `Fragment`。

在Activity中创建Intent 携带参数：

```java
public static Intent getStartIntent(Context context, User user) {
	Intent intent = new Intent(context, ThisActivity.class);
	intent.putParcelableExtra(EXTRA_USER, user);
	return intent;
}
```
对于Fragment应该命名为 `newInstance` 并且处理创建Framgnet携带的参数：

```java
public static UserFragment newInstance(User user) {
	UserFragment fragment = new UserFragment;
	Bundle args = new Bundle();
	args.putParcelable(ARGUMENT_USER, user);
	fragment.setArguments(args)
	return fragment;
}
```

__注意 1__: 这些方法应该在类的顶部创建。

__注意 2__: 如果我们提过上面描述的方法，额外的参数应该是 `private`，因为它不应该暴露给外部类。

### 2.2.15 限制代码行的长度

每行代码的长度应该不超过100个字符。

有关本规则的讨论有很多，最后的结论还是最多不超过100个字符。

#### 2.2.15.1 换行策略

There isn't an exact formula that explains how to line-wrap and quite often different solutions are valid. However there are a few rules that can be applied to common cases.

__Break at operators__

When the line is broken at an operator, the break comes __before__ the operator. For example:

```java
int longName = anotherVeryLongVariable + anEvenLongerOne - thisRidiculousLongOne
        + theFinalOne;
```

__Assignment Operator Exception__

An exception to the `break at operators` rule is the assignment operator `=`, where the line break should happen __after__ the operator.

```java
int longName =
        anotherVeryLongVariable + anEvenLongerOne - thisRidiculousLongOne + theFinalOne;
```

__Method chain case__

When multiple methods are chained in the same line - for example when using Builders - every call to a method should go in its own line, breaking the line before the `.`

```java
Picasso.with(context).load("http://ribot.co.uk/images/sexyjoe.jpg").into(imageView);
```

```java
Picasso.with(context)
        .load("http://ribot.co.uk/images/sexyjoe.jpg")
        .into(imageView);
```

__Long parameters case__

When a method has many parameters or its parameters are very long, we should break the line after every comma `,`

```java
loadPicture(context, "http://ribot.co.uk/images/sexyjoe.jpg", mImageViewProfilePicture, clickListener, "Title of the picture");
```

```java
loadPicture(context,
        "http://ribot.co.uk/images/sexyjoe.jpg",
        mImageViewProfilePicture,
        clickListener,
        "Title of the picture");
```

### 2.2.16 RxJava chains styling

Rx chains of operators require line-wrapping. Every operator must go in a new line and the line should be broken before the `.`

```java
public Observable<Location> syncLocations() {
    return mDatabaseHelper.getAllLocations()
            .concatMap(new Func1<Location, Observable<? extends Location>>() {
                @Override
                 public Observable<? extends Location> call(Location location) {
                     return mRetrofitService.getLocation(location.id);
                 }
            })
            .retry(new Func2<Integer, Throwable, Boolean>() {
                 @Override
                 public Boolean call(Integer numRetries, Throwable throwable) {
                     return throwable instanceof RetrofitError;
                 }
            });
}
```

## 2.3 XML style rules

### 2.3.1 Use self closing tags

When an XML element doesn't have any contents, you __must__ use self closing tags.

This is good:

```xml
<TextView
	android:id="@+id/text_view_profile"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content" />
```

This is __bad__ :

```xml
<!-- Don\'t do this! -->
<TextView
    android:id="@+id/text_view_profile"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" >
</TextView>
```


### 2.3.2 Resources naming

Resource IDs and names are written in __lowercase_underscore__.

#### 2.3.2.1 ID naming

IDs should be prefixed with the name of the element in lowercase underscore. For example:


| Element            | Prefix            |
| -----------------  | ----------------- |
| `TextView`           | `text_`             |
| `ImageView`          | `image_`            |
| `Button`             | `button_`           |
| `Menu`               | `menu_`             |

Image view example:

```xml
<ImageView
    android:id="@+id/image_profile"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content" />
```

Menu example:

```xml
<menu>
	<item
        android:id="@+id/menu_done"
        android:title="Done" />
</menu>
```

#### 2.3.2.2 Strings

String names start with a prefix that identifies the section they belong to. For example `registration_email_hint` or `registration_name_hint`. If a string __doesn't belong__ to any section, then you should follow the rules below:


| Prefix             | Description                           |
| -----------------  | --------------------------------------|
| `error_`             | An error message                      |
| `msg_`               | A regular information message         |
| `title_`             | A title, i.e. a dialog title          |
| `action_`            | An action such as "Save" or "Create"  |



#### 2.3.2.3 Styles and Themes

Unless the rest of resources, style names are written in __UpperCamelCase__.

### 2.3.3 Attributes ordering

As a general rule you should try to group similar attributes together. A good way of ordering the most common attributes is:

1. View Id
2. Style
3. Layout width and layout height
4. Other layout attributes, sorted alphabetically
5. Remaining attributes, sorted alphabetically

## 2.4 Tests style rules

### 2.4.1 Unit tests

Test classes should match the name of the class the tests are targeting, followed by `Test`. For example, if we create a test class that contains tests for the `DatabaseHelper`, we should name it `DatabaseHelperTest`.

Test methods are annotated with `@Test` and should generally start with the name of the method that is being tested, followed by a precondition and/or expected behaviour.

* Template: `@Test void methodNamePreconditionExpectedBehaviour()`
* Example: `@Test void signInWithEmptyEmailFails()`

Precondition and/or expected behaviour may not always be required if the test is clear enough without them.

Sometimes a class may contain a large amount of methods, that at the same time require several tests for each method. In this case, it's recommendable to split up the test class into multiple ones. For example, if the `DataManager` contains a lot of methods we may want to divide it into `DataManagerSignInTest`, `DataManagerLoadUsersTest`, etc. Generally you will be able to see what tests belong together because they have common [test fixtures](https://en.wikipedia.org/wiki/Test_fixture).

### 2.4.2 Espresso tests

Every Espresso test class usually targets an Activity, therefore the name should match the name of the targeted Activity followed by `Test`, e.g. `SignInActivityTest`

When using the Espresso API it is a common practice to place chained methods in new lines.

```java
onView(withId(R.id.view))
        .perform(scrollTo())
        .check(matches(isDisplayed()))
```

# License

```
Copyright 2015 Ribot Ltd.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```





