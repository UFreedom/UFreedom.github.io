---
layout: post
title: 你的 Typeface 优化了吗？
comments: true
category: Android
tags: [Android]
---

------

前些日子在优化公司应用的 RV 列表时，使用 TraceView 工具分析哪些因素在影响 RV 滑动的流畅度;

在分析 TraceView 记录时，我发现了下面的问题：

![](http://upload-images.jianshu.io/upload_images/1721932-9858cd5dfea1357d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

> 在 TraceView 视图中，左侧显示的方法的调用关系，右边显示的是分析数据，你可以根据自己的需求对分析的数据进行筛选。


在上面的截图中，第二列是 Incl CPU Time （某函数占用 CPU 的时间）。很明显能够看出 onCreateViewHolder 占用 CPU时间为 42.71，而 ```Typeface.createFromAsset()``` 方法神奇的霸占了其 97 %  时间，那么多的 findViewById() 也仅仅是 3%。

我擦，我立即去浏览了下代码，原来在 Item 中某个 TextView 使用了一个自定义的字体，设置字体的过程发生在每个 Item 的 ```onCreateViewHolder``` 阶段,设置字体时使用 ```Typeface.createFromAsset()``` 创建字体资源，然后使用 ```textView.setTypeface()``` 设置字体 ，CPU 占用时间高的原因是每次调用 Typeface.createFromAsset() 时，都会去解析字体，然后创建相对应的字体资源实例。让我产生疑问的是代码中引用的字体资源都是同一个，类比 Drawable 资源，难道 Android 对相同的字体资源没有优化？

带着我的疑问去 Google 了一下，发现了下面的讨论：
[Typeface.createFromAsset leaks asset stream](https://code.google.com/p/android/issues/detail?id=9904)

提问者通过分析底层 ``` Typeface.c```  源码，发现每次调用 ```Typeface.createFormAsset``` 都会在内存中加载一个新的实例，关键是分配的这些内存都不会被回收掉，这就造成了内存泄露问题，按照提问者给出的测试方法，我写了一个 Demo 来验证这个问题。

1.首先是 xml 布局文件，布局中有四个纵向并列的 TextView ：

{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.ufreedom.typefacedemo.MainActivity">

    <TextView
        android:id="@+id/text1"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />

    <TextView
        android:id="@+id/text2"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />

    <TextView
        android:id="@+id/text3"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />

    <TextView
        android:id="@+id/text4"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Hello World!" />

</LinearLayout>

{% endhighlight %}



2. 在  Java 代码中我为每个 TextView 都创建了一个字体实例，字体使用的是 Google 开源的 Roboto 字体，四个 TextView 使用的都是同一个字体资源。

{% highlight java %}

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        Typeface typeface =  Typeface.createFromAsset(getAssets(),"Roboto-Italic.ttf");
        Typeface typeface2 =  Typeface.createFromAsset(getAssets(),"Roboto-Italic.ttf");
        Typeface typeface3 =  Typeface.createFromAsset(getAssets(),"Roboto-Italic.ttf");
        Typeface typeface4 =  Typeface.createFromAsset(getAssets(),"Roboto-Italic.ttf");

        ((TextView)findViewById(R.id.text1)).setTypeface(typeface);
        ((TextView)findViewById(R.id.text2)).setTypeface(typeface2);
        ((TextView)findViewById(R.id.text3)).setTypeface(typeface3);
        ((TextView)findViewById(R.id.text4)).setTypeface(typeface4);

    }
}

{% endhighlight %}


然后运行 Demo 后效果：

<div align="center">
 <img src="http://upload-images.jianshu.io/upload_images/1721932-50d6274357d54267.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240"  />
</div>


可以看到上面四个 TextView 的字体使用的都是 Google  Roboto 字体，然后我用 ``` adb shell dumpsys meminfo com.ufreedom.typefacedemo ``` 查看内存分配情况：

<div align="center">
 <img src="http://upload-images.jianshu.io/upload_images/1721932-010029aabac63062.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240"  />
</div>


从上图可以看出，在 Asset Allocations 那个指标项，内存中共分配了 4 个 Roboto 字体实例，也就是说每次调用 ```Typeface.createFromAsset```  都会加载一个新的实例到内存中。显然这个是没有必要的，更何况每个分配的字体资源都不会回收掉。

针对这个问题，网上也有优化方案，那就是将首次解析的字体资源缓存起来，以后再用到相同的字体资源时，直接取缓存的即可：

{% highlight java %}

public class TypefaceHelper {

    private static final String TAG = "TypefaceHelper";
    private static final SimpleArrayMap<String, Typeface> TYPEFACE_CACHE = new SimpleArrayMap<String, Typeface>();

    public static Typeface get(Context context, String name) {
        synchronized (TYPEFACE_CACHE) {
            if (!TYPEFACE_CACHE.containsKey(name)) {

                try {
                    Typeface t = Typeface.createFromAsset(context.getAssets(), name);
                    TYPEFACE_CACHE.put(name, t);
                } catch (Exception e) {
                    Log.e(TAG, "Could not get typeface '" + name
                            + "' because " + e.getMessage());
                    return null;
                }
            }
            return TYPEFACE_CACHE.get(name);
        }
    }
}

{% endhighlight %}


下面我就对 Demo 进行优化：

{% highlight java %}

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        ((TextView)findViewById(R.id.text1)).setTypeface(TypefaceHelper.get(this,"Roboto-Italic.ttf"));
        ((TextView)findViewById(R.id.text2)).setTypeface(TypefaceHelper.get(this,"Roboto-Italic.ttf"));
        ((TextView)findViewById(R.id.text3)).setTypeface(TypefaceHelper.get(this,"Roboto-Italic.ttf"));
        ((TextView)findViewById(R.id.text4)).setTypeface(TypefaceHelper.get(this,"Roboto-Italic.ttf"));

    }
}
{% endhighlight %}

经过优化后，运行效果和之前一样，四个 TextView 的字体依然使用的是 Google  Roboto 字体，然后通过 ``` adb shell dumpsys meminfo com.ufreedom.typefacedemo ``` 查看内存分配情况：

<div align="center">
 <img src="http://upload-images.jianshu.io/upload_images/1721932-a565f229aeb39af4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240"  />
</div>

可以清晰的看到内存中 Roboto 字体资源只分配了一个实例。

如果你的 APP 也在使用自定义字体，那就可以使用``` adb shell dumpsys meminfo <package_name|pid>  ``` 查看内存分配情况，如果也有上述的问题，就可以使用缓存机制进行优化。
