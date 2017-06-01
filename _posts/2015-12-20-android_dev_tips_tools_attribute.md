---
layout: post
title: Android Dev Tips — Tools 属性
excerpt: 加速，加快你的布局设计.
comments: true
category: Android
tags: [Android]

---

## Tools 属性

tools 属性在 Android 开发中专门用来记录或者测试 xml 中一些信息,它主要有三个用处：
- 被 Lint 工具用来分析代码
- 在 Android Studio 布局编辑器中预览和测试一些布局属性
- 被 Android Studio Gradle 插件用来做资源收缩处理

要使用 `tools` 属性,必须在XML中加入命名空间: `xmlns:tools="http://schemas.android.com/tools"` ,然后我们在使用时以 `tools` 作为前缀:

{% highlight xml %}
<FrameLayout
       xmlns:android="http://schemas.android.com/apk/res/android"
       xmlns:tools="http://schemas.android.com/tools"
       android:layout_width="match_parent"
       android:layout_height="match_parent" >
    ....

{% endhighlight %}

下面详细的介绍 tools 相关属性.

### 一.与 Lint 相关的 tools 属性

#### 1.tools:ignore

<div align="center">
<img src="/attachments/images/demo_tools_ignore.gif"  width="443"  height="284" />
 </div>


**功能:**
用来设置 Android Studio 在编译时忽略 Lint 的某些警告，任何一个 XML 元素都可以使用它

**使用:**

{% highlight xml %}
<string name="show_all_apps" tools:ignore="MissingTranslation">All</string>
{% endhighlight %}

#### 2.tools:targetApi

<div align="center">
<img src="/attachments/images/demo_tools_targetapi.gif"  width="586"  height="215" />
 </div>

**功能:**
用来设置 Lint 忽略 XML 中有关 API 版本的问题，假设你使用了一个属性在 api21 才会有，但是你当前的minSdk版本为14，这个时候 Android Studio 就会提示警告.

**使用:**

{% highlight xml %}
<GridLayout tools:targetApi="ICE_CREAM_SANDWICH" >
{% endhighlight %}


#### 3.tools:locale

**功能:**
用来设定资源文件使用的语言.

**使用:**

{% highlight xml %}
<resources xmlns:tools="http://schemas.android.com/tools" tools:locale="es">
{% endhighlight %}

这样 Android Studio 就会知道资源文件使用的默认语言是西班牙语而不是英语.

### 二.在布局编辑器中预览和测试一些布局属性

我们在画布局的时候经常会写一些测试属性来测试布局，控件是否画的符合设计要求，例如使用 `android:text="测试测试"` 来测试 `TextView` 控件的位置和大小，但是问题也随之而来:
 - 如果不及时擦除测试属性，产品上线后会出现意想不到的bug.
 - 如果每次画完布局就擦除测试属性，下次再修改布局时可能需要重新再写一遍测试属性.

Android 有一个特殊的属性能够解决这个问题: `tools` 属性,`tools` 是被用来在XML文件中做些测试,然后当应用编译打包后便会自动擦除，完全不会影响运行时的效果.

#### 1. tools:context

**功能:**
这个属性可以用来预览你在 `Manifest` 文件中指定 `Activity` 的主题，因为 `Activity` 的主题在 `Manifest` 文件中指定而不是在布局文件中。所以默认情况下布局编辑器无法知道你实际设置的主题.

**使用:**
`context` 属性经常用在一个XML布局文件的根布局中，然后使用 `.` 前缀来制定某个 `Activity` 类，

{% highlight xml %}
<android.support.v7.widget.GridLayout
   xmlns:android="http://schemas.android.com/apk/res/android"
   xmlns:tools="http://schemas.android.com/tools"
   tools:context=".MainActivity" ... >
{% endhighlight %}


#### 2. tools:layout

**功能:**
这个属性在 `<fragment>` 标签中使用,用来预览设计时的布局

**使用:**

{% highlight xml %}
<fragment
  android:name="com.example.master.ItemListFragment"
  tools:layout="@android:layout/list_content" />
  {% endhighlight %}



#### 3. tools:listitem / listheader / listfooter

<div align="center">
     <img src="/attachments/images/demo_tools_list.gif"  width="300"  height="300" />
</div>


**功能**
这些属性可以用在 `AdapterView` 的子类如 `<ListView>` : `<GridView>` ,`<ExpandableListView>`等，在设计时预览其item的布局， `headers` 和 `footers`:
- `listitem` : 一个列表的item，编辑器自动填充一些模拟数据显示item.
- `listheader` : 一个列表的头部.
- `listfooter` : 一个列表的尾部.

**使用:**

{% highlight xml %}
<ListView
    android:id="@android:id/list"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:listitem="@android:layout/simple_list_item_2"
    tools:listheader="@android:layout/list_view_header"
    tools:listfooter="@android:layout/list_view_footer"/>

{% endhighlight %}


#### 4. tools:showIn


<div align="center">
     <img src="/attachments/images/demo_tools_showin.gif"  width="300"  height="300" />
</div>



**功能:**
我们知道，使用 `<include>` 标签引用一个布局，一个布局可以被其他多个布局使用 `<include>` 标签同时引用.假设布局 D 被布局 A，B，C 引用，我们在设计布局 D 时想知道 D 在某个布局中如何显示，这个时候就可以使用 `tools"showIn` 指定其中一个布局.如果指定的布局并被有使用 `<include>`标签引用 D 布局，这个时候 AndroidStuidio就会提示 `Rendering Problems : The surrounding layout(xxx) did not actually include this layout,romove toools:showIn=... from root tag `.

**使用:**

{% highlight xml %}
<TextView
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:text="@string/hello_world"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    tools:showIn="@layout/activity_main" />
{% endhighlight %}

#### 5. tools:menu
**功能:**
这个属性可以让 Android Stuidio 在预览窗口显示 `ActionBar` 菜单, Android Studio 会尝试查找 `onCreateOptionsMenu()` 中使用哪个菜单(需要使用 `tools:context` 属性指定关联的 `Activity`). `tools:menu` 可以指定多个菜单资源,多个菜单资源名用逗号隔开.

**使用:**
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:menu="menu1,menu2" />
{% endhighlight %}


#### 5. tools:actionBarNavMode
**功能:**
这个属性用来指定预览窗口 `ActionBar` 的显示模式，总共有三个模式：`standard`,`list`,`tabs`

**使用:**
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:menu="menu1,menu2" />
{% endhighlight %}

### 三.使用 Tools 收缩资源
 Android Studio Gradle 编译系统支持 “资源收缩”，“资源收缩” 是什么个意思呢？就是它可以在打包编译时根据一些配置自动移除掉没有被使用的资源，这样能够有效减少资源污染.更多的详情请参考 [Android Dev Tips - 资源收缩 (Resource Shrinking)](http://ufreedom.me).这里仅仅介绍三个相关的 tools 属性: `tools:shrinkMode`,`tools:keep`,`tools:discard`

#### 1. tools:shrinkMode

**功能:**
当使用资源收缩来移除没有使用的资源时，这里有两个方案可选：

- 安全，保守方案 - `shrinkMode="safe"` ：将所有被直接引用的资源，已经可能会通过 `getIdentifier` 引用的资源列入一个白名单，这种能最大的保证不会错杀，但是也会将一些实际上真正无用资源加入白名单。

- 更加苛刻方案 - `shrinkMode="strict"` ： 这种情况是仅仅考虑被明确引用的资源，这里又有两个属性可以使用: `tools:keep`和`tools:discard`,这俩属性可以明确的指定要留下或移除的资源

**使用:**
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:shrinkMode="safe"
    tools:discard="@layout/unused2" />
{% endhighlight %}

#### 2. tools:keep
**功能:**
当`shrinkMode="strict` 模式为 `strict` 时，使用此属性指定哪些资源可以被保留，最常见的情况是他们被运行时代码 `Resources#getIdentifier`直接引用
**使用:**

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*" />
{% endhighlight %}

#### 3. tools:keep

**功能:**
同样是在`shrinkMode="strict` 模式为 `strict` 情况下，不过与 `tools:keep` 相反，这个属性可以指定哪些属性被移除

**使用:**

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:discard="@layout/unused2" />
{% endhighlight %}
