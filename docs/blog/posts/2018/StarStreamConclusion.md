---
title: 星空直播平台安卓端开发的一些总结
date: 2018-02-04 13:31:33
# tags: Android
# categories: 开发
---

项目地址：[https://github.com/xingkongus/stream-watch-android](https://github.com/xingkongus/stream-watch-android)

## 项目的开始

[星空直播](http://live.xingkong.us)平台的后端整个算是[翰新](https://github.com/Hansin1997)搭建起来的。他给我后端的API接口，我试着把他封装成SDK，并且做一个安卓端的Demo出来。于是项目就开始了......

<!-- more -->

## 一些知识的总结

### Fragment全局获取上下文联系Context

在封装SDK的过程中，我用SharedPreferences来存储从服务器获取的Cookie，里面保存和获取保存的Cookie的方法为：

```
    public static void saveCookiePreference(Context context, String value) {
        SharedPreferences preference = context.getSharedPreferences(ISLOGINED, Context.MODE_PRIVATE);
        SharedPreferences.Editor editor = preference.edit();
        editor.putString(COOKIE, value);
        editor.apply();
    }

    public static String getCookiePreference(Context context) {
        SharedPreferences preference = context.getSharedPreferences(ISLOGINED, Context.MODE_PRIVATE);
        return preference.getString(COOKIE, "");
    }
```

很明显我需要用到Context，所以我在实例化操作API的对象时需要传入一个参数：
```
    public Client(Context context) {
        this.context = context;
        ......
    }
```

但是我在Fragment里面需要实例化Cilent类时，不管是利用Fragment类的getActivity()还是getContext()方法，SharedPreferences的操作总是会报错，大致的意思就是没有办法获取Context。而AS也会有getActivity()或getContext()返回值可能为空的警告。我不知道为什么，但是就干脆自己写一个获取全局变量的方法。

首先写一个自定义的Application类MyApplication，写法如下:

```
public class MyApplication extends Application {

    /**
     * 一个MyApplication的静态对象
     * 用于Fragment获取上下文练习Context
     */
    private static MyApplication mInstance;

    @Override
    public void onCreate() {
        super.onCreate();
        mInstance = this;   //在MyApplication 启动时获取本对象

        ......

    }
    /**
     * 获取context
     *
     * @return mInstance
     */
    public static Context getInstance() {
        return mInstance;
    }
}
```

以此便可以获取当前的Application实例，然后写BaseFragment：

```
public class BaseFragment extends Fragment {

    /**
     * 一个Activity对象作为Context
     */
    private Activity activity;

    /**
     * 子Fragment获取Context的方法
     * @return activity
     */
    public Context getInstantContext() {
        if (activity == null) {
            return MyApplication.getInstance();
        }
        return activity;
    }

    /**
     * 用Fragment的onAttach(Context context);方法来绑定Context
     * @param context
     */
    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        activity = getActivity();
    }
}
```

随后的Fragment继承BaseFragment，实例化Client时调用getInstantContext()方法即可。

### ActivityLifecycleCallbacks接口的使用

现在很多安卓开发的过程中都会用到Toolbar这一个强大的控件，但是一般的App都不止由一个Activity组成，如果在写每个Activity时都要实例化一个Toolbar显然不高效。

我原本的实现方式是封装一个BaseActivity，在里面实现Toobar然后将子Activity的布局用LayoutInflater投射到BaseActivity布局的容器里。

然而，我在开发的过程中引入了ButterKnife，一个很好用的依赖注入框架。但是问题也出现了：继承BaseActivity的子Activity中的布局，可能是由LayoutInflater投射到容器的原因，ButterKnife无法获取布局中的控件值。

在试了很多方法都无法解决之后，我突然想起在有道云笔记收藏的一篇文章：[我一行代码都不写实现Toolbar!你却还在封装BaseActivity?](https://www.jianshu.com/p/75a5c24174b2);文章不提倡封装BaseActivity，因为会有管理困难的问题。在我此次开发也会有这样的问题：并非所有的Activity都需要Toobar，但是其可能需要一些其他的对象或着操作。

而文章的解决办法用到了Application类内部的一个接口ActivityLifecycleCallbacks和方法registerActivityLifecycleCallbacks()来解决。作用就是就是在你调用这个方法传入这个接口实现类后,**系统会在每个 Activity 执行完对应的生命周期后都调用这个实现类中对应的方法，注意是每个**

#### 具体的用法

先按照自己的需求单独写一个toolbar.xml

```
<?xml version="1.0" encoding="utf-8"?>
<android.support.v7.widget.Toolbar xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/us.xingkong.testing"
    android:id="@+id/toolbar"
    android:layout_width="match_parent"
    android:layout_height="?attr/actionBarSize"
    app:layout_scrollFlags="scroll||enterAlways"
    app:popupTheme="@style/ThemeOverlay.AppCompat.Light">

</android.support.v7.widget.Toolbar>
```

在需要的地方include：

```
<include layout="@layout/toolbar" />
```

那么回到自定义的MyApplication：

```
public class MyApplication extends Application {

   ......

    @Override
    public void onCreate() {
        super.onCreate();
        ......

        registerActivityLifecycleCallbacks(new ActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
                if (activity.findViewById(R.id.toolbar)!=null){
                    ((BaseActivity)activity).setSupportActionBar((Toolbar)activity.findViewById(R.id.toolbar));
                }
            }

            ......
        });
    }

    ......
}
```

在Activity中：

```
public abstract class BaseActivity extends AppCompatActivity {

    ......

    @Override
    protected void onCreate(final Bundle savedInstanceState) {
        setContentView(getLayout());
        super.onCreate(savedInstanceState);
        ......
    }

    ......
}
```

>**注意：由于 ActivityLifecycleCallbacks 中所有方法的调用时机都是在 Activity 对应生命周期的 Super 方法中进行的,所以在 Activity 的 onCreate 方法中使用 setContentView 必须在 super.onCreate(savedInstanceState); 之前,不然在 onActivityCreated 方法中 findViewById 会发现找不到。（文章原话，这个坑我就踩了）**

由此便可以不用BaseActivity来实现Toolbar。

## 关于项目的一些思考

* 总的来说我对此项目有点不满意的是由于才疏浅薄，我想不到一个更好的封装SDK封装方式，以至于需要在每个Activity创建的时候都实例一个Client对象。显然这还是比较低级的。暂时的想法可能会试着将其封装为用 Client.方法名 的静态方法的方式来调用。

* 没有使用结构，比如MVP、MVVM等。接下来还是重点学习一下MVP...

* Java(HttpUrlConnection)是否已经有将Cookie存于本地的方法，这个有待于我去好好研究，毕竟如果本身已经有更高效的方法，就没有必要再用SharedPreferences。

* 依然不能明白Fragment中的getActivity()或getContext()返回值为何为空...

***当然开发的过程中遇到了很多坑，等想到了再继续补充吧***