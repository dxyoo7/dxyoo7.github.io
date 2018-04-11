---
title: Android Architecture Components研究
date: 2018-04-11 16:08:29
tags: Java
---

## 简介
Google IO 2017发布Android Architecture Components，自己先研究了一下，蛮有意思的，特来记录一下。本文内容主要是参考官方文档以及自己的理解，如有错误之处，恳请指出。

Android Architecture Components
Android Architecture Components是Google发布的一套新的架构组件，使App的架构更加健壮，后面简称AAC。

AAC主要提供了Lifecycle，ViewModel，LiveData，Room等功能，下面依次说明：


**Lifecycle**
生命周期管理，把原先Android生命周期的中的代码抽取出来，如将原先需要在onStart()等生命周期中执行的代码分离到Activity或者Fragment之外。

**LiveData**
一个数据持有类，持有数据并且这个数据可以被观察被监听，和其他Observer不同的是，它是和Lifecycle是绑定的，在生命周期内使用有效，减少内存泄露和引用问题。

**ViewModel**
用于实现架构中的ViewModel，同时是与Lifecycle绑定的，使用者无需担心生命周期。可以在多个Fragment之间共享数据，比如旋转屏幕后Activity会重新create，这时候使用ViewModel还是之前的数据，不需要再次请求网络数据。

**Room**
谷歌推出的一个Sqlite ORM库，不过使用起来还不错，使用注解，极大简化数据库的操作，有点类似Retrofit的风格。

AAC的架构是这样的：
![AAC](https://raw.githubusercontent.com/LiushuiXiaoxia/AndroidArchitectureComponents/master/doc/final-architecture.png)




* Activity/Fragment

UI层，通常是Activity/Fragment等，监听ViewModel，当VIewModel数据更新时刷新UI，监听用户事件反馈到ViewModel，主流的数据驱动界面。

* ViewModel

持有或保存数据，向Repository中获取数据，响应UI层的事件，执行响应的操作，响应数据变化并通知到UI层。

* Repository

App的完全的数据模型，ViewModel交互的对象，提供简单的数据修改和获取的接口，配合好网络层数据的更新与本地持久化数据的更新，同步等

* Data Source

包含本地的数据库等，网络api等，这些基本上和现有的一些MVVM，以及Clean架构的组合比较相似

**Gradle 集成**

根目录gradle文件中添加Google Maven Repository

```
allprojects {
    repositories {
        jcenter()
        maven { url 'https://maven.google.com' }
    }
}
```

在模块中添加对应的依赖

如使用Lifecycle,LiveData、ViewModel，添加如下依赖。

```
compile "android.arch.lifecycle:runtime:1.0.0-alpha1"
compile "android.arch.lifecycle:extensions:1.0.0-alpha1"
annotationProcessor "android.arch.lifecycle:compiler:1.0.0-alpha1"
```

如使用Room功能，添加如下依赖。

```
compile "android.arch.persistence.room:runtime:1.0.0-alpha1"
annotationProcessor "android.arch.persistence.room:compiler:1.0.0-alpha1"

// For testing Room migrations, add:
testCompile "android.arch.persistence.room:testing:1.0.0-alpha1"

// For Room RxJava support, add:
compile "android.arch.persistence.room:rxjava2:1.0.0-alpha1"
```

### LifeCycles

Android开发中，经常需要管理生命周期。举个栗子，我们需要获取用户的地址位置，当这个Activity在显示的时候，我们开启定位功能，然后实时获取到定位信息，当页面被销毁的时候，需要关闭定位功能。

下面是简单的示例代码。

```
class MyLocationListener {
    public MyLocationListener(Context context, Callback callback) {
        // ...
    }

    void start() {
        // connect to system location service
    }

    void stop() {
        // disconnect from system location service
    }
}

class MyActivity extends AppCompatActivity {

    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, (location) -> {
            // update UI
        });
    }

    public void onStart() {
        super.onStart();
        myLocationListener.start();
    }

    public void onStop() {
        super.onStop();
        myLocationListener.stop();
    }
}
```

上面只是一个简单的场景，我们来个复杂一点的场景。当定位功能需要满足一些条件下才开启，那么会变得复杂多了。可能在执行Activity的stop方法时，定位的start方法才刚刚开始执行，比如如下代码，这样生命周期管理就变得很麻烦了。

```
class MyActivity extends AppCompatActivity {
    private MyLocationListener myLocationListener;

    public void onCreate(...) {
        myLocationListener = new MyLocationListener(this, location -> {
            // update UI
        });
    }

    public void onStart() {
        super.onStart();
        Util.checkUserStatus(result -> {
            // what if this callback is invoked AFTER activity is stopped?
            if (result) {
                myLocationListener.start();
            }
        });
    }

    public void onStop() {
        super.onStop();
        myLocationListener.stop();
    }
}
```

AAC中提供了Lifecycle，用来帮助我们解决这样的问题。LifeCycle使用2个枚举类来解决生命周期管理问题。一个是事件，一个是状态。

事件：

生命周期事件由系统来分发，这些事件对于与Activity和Fragment的生命周期函数。

状态：

Lifecycle的状态，用于追踪中Lifecycle对象，如下图所示。

![](https://raw.githubusercontent.com/LiushuiXiaoxia/AndroidArchitectureComponents/master/doc/lifecycle-states.png)


上面的定位功能代码，使用LifeCycle实现以后是这样的，实现一个LifecycleObserver接口，然后用注解标注状态，最后在LifecycleOwner中添加监听。

```
public class MyObserver implements LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    public void onResume() {
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    public void onPause() {
    }
}

aLifecycleOwner.getLifecycle().addObserver(new MyObserver());
```

上面代码中用到了`LifecycleOwner`是`LifecycleOwner`接口对象，`LifecycleOwner`是一个只有一个方法的接口`getLifecycle()`，需要由子类来实现。

在Lib中已经有实现好的子类，我们可以直接拿来使用。比如`LifecycleActivity`和`LifecycleFragment`，我们只需要继承此类就行了。

当然实际开发的时候我们会自己定义`BaseActivity`，Java是单继承模式，那么需要自己实现`LifecycleRegistryOwner`接口。

如下所示即可，代码很近简单

```
public class BaseFragment extends Fragment implements LifecycleRegistryOwner {

    LifecycleRegistry lifecycleRegistry = new LifecycleRegistry(this);

    @Override
    public LifecycleRegistry getLifecycle() {
        return lifecycleRegistry;
    }
}
```

### LiveData
`LiveData` 是一个 `Data Holder` 类，可以持有数据，同时这个数据可以被监听的，当数据改变的时候，可以触发回调。但是又不像普通的Observable，LiveData绑定了App的组件，`LiveData`可以指定在`LifeCycle`的某个状态被触发。比如`LiveData`可以指定在`LifeCycle`的 `STARTED` 或 `RESUMED`状体被触发。

```
public class LocationLiveData extends LiveData<Location> {

    private LocationManager locationManager;

    private SimpleLocationListener listener = new SimpleLocationListener() {
        @Override
        public void onLocationChanged(Location location) {
            setValue(location);
        }
    };

    public LocationLiveData(Context context) {
        locationManager = (LocationManager) context.getSystemService(Context.LOCATION_SERVICE);
    }

    @Override
    protected void onActive() {
        locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, listener);
    }

    @Override
    protected void onInactive() {
        locationManager.removeUpdates(listener);
    }
}
```

`onActive()`
这个方法在`LiveData`在被激活的状态下执行，我们可以开始执行一些操作。

`onActive()`
这个方法在`LiveData`在的失去活性状态下执行，我们可以结束执行一些操作。

`setValue()`
执行这个方法的时候，`LiveData`可以触发它的回调。

`LocationLiveData`可以这样使用。

```
public class MyFragment extends LifecycleFragment {

    public void onActivityCreated (Bundle savedInstanceState) {
        LiveData<Location> myLocationListener = ...;
        Util.checkUserStatus(result -> {
            if (result) {
                myLocationListener.addObserver(this, location -> {
                    // update UI
                });
            }
        });
    }
}
```

注意，上面的`addObserver`方法，必须传`LifecycleOwner`对象，也就是说添加的对象必须是可以被`LifeCycle`管理的。

如果`LifeCycle`没有触发对对应的状态（`STARTED` or `RESUMED`），它的值被改变了，那么`Observe`就不会被执行，

如果`LifeCycle`被销毁了，那么`Observe`将自动被删除。

实际上`LiveData`就提供一种新的供数据共享方式。可以用它在多个`Activity`、`Fragment`等其他有生命周期管理的类中实现数据共享。

还是上面的定位例子。

```
public class LocationLiveData extends LiveData<Location> {

    private static LocationLiveData sInstance;

    private LocationManager locationManager;

    @MainThread
    public static LocationLiveData get(Context context) {
        if (sInstance == null) {
            sInstance = new LocationLiveData(context.getApplicationContext());
        }
        return sInstance;
    }

    private SimpleLocationListener listener = new SimpleLocationListener() {
        @Override
        public void onLocationChanged(Location location) {
            setValue(location);
        }
    };

    private LocationLiveData(Context context) {
        locationManager = (LocationManager) context.getSystemService(Context.LOCATION_SERVICE);
    }

    @Override
    protected void onActive() {
        locationManager.requestLocationUpdates(LocationManager.GPS_PROVIDER, 0, 0, listener);
    }

    @Override
    protected void onInactive() {
        locationManager.removeUpdates(listener);
    }
}
```

然后在Fragment中调用。

```
public class MyFragment extends LifecycleFragment {

    public void onActivityCreated (Bundle savedInstanceState) {

        Util.checkUserStatus(result -> {
            if (result) {
                LocationLiveData.get(getActivity()).observe(this, location -> {
                   // update UI
                });
            }
        });
  }
}
```
从上面的示例，可以得到使用LiveData优点：

> * 没有内存泄露的风险，全部绑定到对应的生命周期，当LifeCycle被销毁的时候，它们也自动被移除

> * 降低Crash，当Activity被销毁的时候，LiveData的Observer自动被删除，然后UI就不会再接受到通知

> * 实时数据，因为LiveData是持有真正的数据的，所以当生命周期又重新开始的时候，又可以重新拿到数据

> * 正常配置改变，当Activity或者Fragment重新创建的时候，可以从LiveData中获取上一次有用的数据

不再需要手动的管理生命周期

Transformations
有时候需要对一个LiveData做Observer，但是这个LiveData是依赖另外一个LiveData，有点类似于RxJava中的操作符，我们可以这样做。

Transformations.map()
用于事件流的传递，用于触发下游数据。
```
LiveData<User> userLiveData = ...;
LiveData<String> userName = Transformations.map(userLiveData, user -> {
    user.name + " " + user.lastName
});
```
Transformations.switchMap()
这个和map类似，只不过这个是用来触发上游数据。
```
private LiveData<User> getUser(String id) {
  // ...
}

LiveData<String> userId = ...;
LiveData<User> user = Transformations.switchMap(userId, id -> getUser(id) );
```

### ViewModel
ViewModel是用来存储UI层的数据，以及管理对应的数据，当数据修改的时候，可以马上刷新UI。

Android系统提供控件，比如Activity和Fragment，这些控件都是具有生命周期方法，这些生命周期方法被系统调用。

当这些控件被销毁或者被重建的时候，如果数据保存在这些对象中，那么数据就会丢失。比如在一个界面，保存了一些用户信息，当界面重新创建的时候，就需要重新去获取数据。当然了也可以使用控件自动再带的方法，在onSaveInstanceState方法中保存数据，在onCreate中重新获得数据，但这仅仅在数据量比较小的情况下。如果数据量很大，这种方法就不能适用了。

另外一个问题就是，经常需要在Activity中加载数据，这些数据可能是异步的，因为获取数据需要花费很长的时间。那么Activity就需要管理这些数据调用，否则很有可能会产生内存泄露问题。最后需要做很多额外的操作，来保证程序的正常运行。

同时Activity不仅仅只是用来加载数据的，还要加载其他资源，做其他的操作，最后Activity类变大，就是我们常讲的上帝类。也有不少架构是把一些操作放到单独的类中，比如MVP就是这样，创建相同类似于生命周期的函数做代理，这样可以减少Activity的代码量，但是这样就会变得很复杂，同时也难以测试。

AAC中提供ViewModel可以很方便的用来管理数据。我们可以利用它来管理UI组件与数据的绑定关系。ViewModel提供自动绑定的形式，当数据源有更新的时候，可以自动立即的更新UI。

下面是一个简单的代码示例。

```
public class MyViewModel extends ViewModel {

    private MutableLiveData<List<User>> users;
    public LiveData<List<User>> getUsers() {
        if (users == null) {
            users = new MutableLiveData<List<Users>>();
            loadUsers();
        }
        return users;
    }

    private void loadUsers() {
        // do async operation to fetch users
    }
```

```
public class MyActivity extends AppCompatActivity {

    public void onCreate(Bundle savedInstanceState) {
        MyViewModel model = ViewModelProviders.of(this).get(MyViewModel.class);
        model.getUsers().observe(this, users -> {
            // update UI
        });
    }
}
```

当我们获取ViewModel实例的时候，ViewModel是通过ViewModelProvider保存在LifeCycle中，ViewModel会一直保存在LifeCycle中，直到Activity或者Fragment销毁了，触发LifeCycle被销毁，那么ViewModel也会被销毁的。下面是ViewModel的生命周期图。

![](https://raw.githubusercontent.com/LiushuiXiaoxia/AndroidArchitectureComponents/master/doc/viewmodel-lifecycle.png)


### Room

（未完待续）

