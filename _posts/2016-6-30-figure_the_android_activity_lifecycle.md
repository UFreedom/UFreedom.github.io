---
layout: post
title: Figure The Android - Activity Lifecycle
comments: true
category: Android
tags: [Android]
---

> 有句话说“眼睛是心灵的窗户”，观察一个人，再没有比观察他的眼睛更好的了。作为一个 APP ，用户如何能够全面的感知它呢，那就是交互界面了。在 Android 3.0 之前，大家说的 APP 界面在技术层面上一般都是指 Activity，可以说一个 Activity 撑起一片天，N 个 Activity 可以撑起整片天。Android 3.0 之后，官方增加了 Fragment 组件，用以分担 Activity 的任务。

说起 Activity ，大家都很熟悉，很可能 Android 初学者了解的第一个专业术语就是 Activity。Activity 直译过来就是 “活动”，可以理解为一系列的交互过程，比如使用购物APP浏览商品，添加商品到购物车，确认结算等。说到 Activity ，最值得搞清楚的就是其生命周期，有句话说“身其位，谋其职”，不同阶段有不同的回调方法，而我们在不同的回调方法内做最适合的工作才是最正确的，下图是一张完整的生命周期图：


<div align="center">
<img src="/attachments/images/figure_the_android/android_lifecycle/lifecycle.jpg" />
 </div>



根据 Android 官方文档，我们可以从下面几个角度去理解 Activity 生命周期。

## 用户感知的三个状态

 - 前台 (Foreground)

   前台状态就是说用户能用眼睛感知它的存在，一个 Activity 在前台就说明能它在承担与用户进行交互的责任。

 - 可见 (Visible)

   可见状态比前台状态更加广泛，它包含前台状态。也就是说处于可见状态的 Activity ，还有一种情况是仅仅是可见，即使用户能用眼睛感知它的存在，但不能和它进行交互，可见但不在前台与用户交互的 Activity 就如博物馆内隔着玻璃陈列的物品一样，你能看看而已，无法用手触摸。这种情况的 Activity 我们会经常见到，比如 一个 Activity 中间弹出一个对话框，即使用户看到了下面的 Activity，用户的所有焦点也都在对话框上。

 - 后台(Background)

   一个 Activity进入了后台状态，就代表这个Activity已经退居幕后了，将不再承担和用户交互的责任，用户无法再用眼睛感知它的存在，或者说它在或者不在，只有我程序知道，反正用户是感觉不到了。  


上面三个状态，从一个比较主观的角度来感知一个Activity：能不能眼睛感知到，能不能与其进行交互。根据 前台(Froeground) 和 可见 (Visible) 状态，我们将 Activity 的生命周期进行分割，每个回调方法相当于一个阶段开始的节点，结束的节点。

<div align="center">
<img src="/attachments/images/figure_the_android/android_lifecycle/foreground_visible.jpg" />
 </div>


## 本质状态 ，Essentially State

 用户感知角度是一个比较主观的角度，在程序中 Activity 有四种本质状态:

  - （活跃) Active

     Activity 处于可见状态，并在前台能与用户进行交互，当前 Activity 栈的在 Activity 栈的栈顶。

  - （暂停）Paused

     Activity 处于可见状态，但是失去焦点，用户无法和其进行交互，例如一个Activity 的中间弹出一个对话框。

  - (停止) Stopped

    Activity 完全的被其他的 Activity 所掩盖，对用户不见。

  - (销毁) Being Destroy

    一旦 Activity 的状态是 stooped。系统可能会要求结束这个 Activity 或者直接杀死它进程 。


##  Activity 生命周期中的回调方法：

- onCreate()

  整个生命周期中的第一个被回调的方法， 只有在 Activity 第一次被创建时会被调用，只要Activity 没被销毁，
  Activity的可见与不可见都与此方法再无关联。在这里可以做初始化任务:  创建 View ， 绑定数据到列表视图 .etc

- onRestart()

  如果 Activity 回调此方法，必然是由于 Activity 之前不可见，但没有销毁，然后现在又开始重新可见。
  比如某个时刻，用户在当前界面时按Home键返回桌面，然后又重新返回当前界面。

- onStart()

  当前 Activity 正在启动，即将进入前台对用户可见。

- onResume()

  Activity 已经在前台对用户可见。即将可以与用户进行交互，这个时候当前 Activity 已经在 activity 栈的栈顶。

- onPause()

  当前 Activity 即将不可见，因为别的 Activity 即将可见。这里做一些不耗时的暂停任务，比如：停止动画及其他会消耗 CPU 的任务，
  提交未保存的持久化数据。一定要注意的是这个方法不能试行耗时任务，因为下一个 Activity 必须在当前Activity的 onPause()方法执行完毕后才会启动。

- onStop()

  Activity 不再对用户可见，因为其他 Activity 已经完全占据了前台，此时的 Activity 已经被切入后台。
  如果此Activity 被再次切换到前台并对用户可见，onRestart() 将会被回调。

- onDestroy()

  当此方法被回调时，也就意味着 Activity 生命的终结，这个方法在 Activity 被销毁之前调用。可以在这里做一些比较耗时的资源释放任务。



## 异常情况下的回调

上面的几个回调方法，都是在正常情况下的回调，比如一般的界面间跳转，应用间跳转，应用于桌面间切换。另外有两种异常情况会引起其他额外的回调方法：

- 系统配置发生变化

  系统配置变化有多种因素：屏幕方向，系统语言，系统时区，输入设备，系统字体等。如果系统的配置发生变化，应用界面就可能需要做些更新，以应对系统配置的变化。

  默认情况下，屏幕方向，系统语言等变化都会导致当前展示的 Activity 被销毁，如果 Activity 可见，Activity 将会回调 onPause(),onStop(),onDestory()方法，当 onDestroy()被回调后，也就意味着当前的 Activity 实例已经被销毁，紧接着系统又会重新创建一个 Activity 实例，进入前台和用户交互。

  在此期间，除了 onPause,onStop,onDestroy 会被回调外，还有一个重要的方法将会被回调：onSaveInstanceSate(Bundle)，这个方法用于保存当前的 Activity 一些状态，值得注意的是这个方法只有在 Activity 异常销毁时才会被调用。 对于 onSaveInstanceSate 方法，它的回调时间并没有一个固定时间，但是能够确定的是它必然会在 onStop() 之前回调，至于它和 onPause() 的顺序，并没有一个分明的顺序。当 Activity 实例被重新创建后， 紧接着 onCreate 方法的是 onRestoreInstanceState() 方法，可以在这个方法里面恢复现场数据。

  有一点需要注意的是，onCreate(Bundle saveInstanceState) 方法中的 saveInstanceState 参数代表 onSaveInstanceState() 中保存的数据，但是onCreate 方法中的这个参数并不是一直都有值，只有当 Activity 被异常销毁然后重建， saveInstanceState 才是非空值。

- 系统资源不足

  这种情况就是比较极端的情况，一旦系统内存吃紧，系统就会销毁优先级较低的 Activity，被销毁的 Activity 的数据保存和数据恢复与 第一种情况一致。一般 Activity 的优先级会被分为下面三情况，从高到低排列：


    - 前台 activity

       Activity 对用户可见， 并与用户进行交互，它的进程优先级最高，是系统最后一个才会考虑的进程，

    - 可见，但不是前台，不能与用户进行交互 Activity

       Activity对用户可见，但不在前台与用户进行交互，例如 activity 中间弹出一个对话框，这种情况的优先级会低一些 ， 一旦 Activity 只有前台的能存活，这种情况的 Activity也会被杀死。

     - 后台 Activity

       Activity 对用户完全不可见，已经被暂停的 Activity.这种情况的优先级最低。

当系统资源不足时，就会按照上面三个情况考虑杀死 Activity 以释放资源，另外还有一种既不持有 Activity 也没有其他组件（Service，Broadcast）的空进程，空进程没有任何优先级，一旦资源不足，就会必然被杀死。所以在 Activity 之外的任何后台任务都需要在 Service 或者 Broadcast 中执行，这样才能不会轻易的被系统杀死。


<div align="center">
<img src="/attachments/images/figure_the_android/android_lifecycle/exception.jpg" />
 </div>
