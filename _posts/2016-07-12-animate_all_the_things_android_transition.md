---
layout: post
title: 动起来！动起来！- Android Transitions 转场动画
comments: true
category: Android
tags: [Android]
---

------
> 原文地址：
本文主要介绍 Github 开源 Android 的动画库 [Transitions-Everywhere][1] .文中大部分内容译自开源作者的博客：[Animate all the things. Transitions in Android][2].

作为 Andorid 开发者我们都知道，Google 在最近的设计理念中引入了 Material Design.可以说开启了 Android 动画交互的新篇章。在  Material Deisgn 中有一个概念: [Material Motion][3]：

>“Motion provides meaning. Objects are presented to the user without breaking the continuity of experience even as they transform and reorganize. Motion in the world of material design is used to describe spatial relationships, functionality, and intention with beauty and fluidity.”
- Material Design guidelines.

概念提的那是非常有内涵，也吸引了大批设计者和开发者追捧。但是回到现实，在 Andorid 写起动画来并不是那么容易。当然我们可以只关注业务逻辑，不去优化交互动画，任何交互的变化都可以使用 setVisibility(View.VISIBLE) 带过。但是有一点：好的交互动画能让人产生愉悦感，让UI妹子对你刮目相看 ~.~ .

空说不干假把式。当写起 Andorid 转场动画时，你就会发现写起来比你想象的难多了。不知道你是否用过 Android 5.0 的 Transitions API ? 没错，它就是 Google 最新提供的 API ，能让你写出更加酷而美的转场动画，遗憾的是我们只能在 Android 5.0 以上版本使用，由于 Android 版本碎片化的问题，导致这个 API 看似有点鸡肋.但是设想一下,如果这种 API 能兼容低版本，而非常容易使用，会不会很期待？


## Android 历史上的转场动画框架

 - Android 4.0,引入了新的属性 android:animateLayoutChanges=[true/false]  ，所有派生自 ViewGroup 的控件都具有此属性，只要在XML中添加上这个属性，就能实现添加/删除其中控件时，带有默认动画，如果要自定义动画，就需要使用 LayoutTransaction 了。实践证明，实际上这套机制使用起来并不是那么灵活。

 - Android 4.4 引入了 Scenes 和 Transitions（场景和变换），Scene 保存了布局的状态，包括所有的控件和控件的属性。布局可以是一个简单的视图控件或者复杂的视图树和子布局。保存了这个布局状态到 Scene 后，我们就可以从另一个场景变化到该场景。从一个场景到另一个场景的变换中会有动画效果，这些动画信息就保存在 Transition 对象中。要运行动画，我们要使用 TransitionManager 实例来应用 Transition。

长江后浪推前浪，一浪更比一浪强，Android 4.4 引入的这套特性或许已经有那么点意思了，是骡子是马，拉出来溜溜就知道了：

实现效果：点击某个按钮，从按钮的下面出现一个文本。

**xml** ：


{% highlight xml %}
<LinearLayout
xmlns:android="http://schemas.android.com/apk/res/android"
             android:id="@+id/transitions_container"
             android:layout_width="match_parent"
             android:layout_height="match_parent"
             android:gravity="center"
             android:orientation="vertical">

    <Button
       android:id="@+id/button"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:text="DO MAGIC"/>

    <TextView
       android:id="@+id/text"
       android:layout_width="wrap_content"
       android:layout_height="wrap_content"
       android:layout_marginTop="16dp"
       android:text="Transitions are awesome!"
       android:visibility="gone"/>

</LinearLayout>
{% endhighlight %}


**Java** :

{% highlight java %}
final ViewGroup transitionsContainer = (ViewGroup) view.findViewById(R.id.transitions_container);
final TextView text = (TextView) transitionsContainer.findViewById(R.id.text);
final Button button = (Button) transitionsContainer.findViewById(R.id.button);

button.setOnClickListener(new View.OnClickListener() {

    boolean visible;

    @Override
    public void onClick(View v) {
        TransitionManager.beginDelayedTransition(transitionsContainer);
        visible = !visible;
        text.setVisibility(visible ? View.VISIBLE : View.GONE);          
    }

});
{% endhighlight %}




 最终效果：

<div align="center">
 <img src="https://cdn-images-1.medium.com/max/800/1*4hcHR-RVHO09ZulwxYb03g.gif"  />
</div>

 还不错，实现这个动画只需要一行代码，有趣的是不仅 TextView 有动画，连 button 的位置也跟着变了。这套转场动画框架能够处理由 TextView 动画的引起的布局变化，并应用到其他受影响的 View 上，这样我们就不用自己去处理了。还有一点是，当老的动画在执行时，你可以再开始一个新的动画，这时老的动画将会停止在当前位置，然后新的动画从当前位置继续执行。当然所有的这些都是这套动画框架帮你做的。

 接下来让我们看看有哪些 Transition  ：

 - ChangeBounds. 改变 View 的位置和大小。
 - Fade.  继承自 Visibility 类，可以用来做最常用的淡入和淡出动画-，上个例子中 TextView的出现和消失用的就是这个。
 - TransitionSet.用来驱动其他的 Transition .类似于 AnimationSet,能够让一组 Transition 有序，或者同时执行。
 - AutoTransition. TransitionSet 同时包含了 Fade out ，ChangeBounds 和 Fade in 效果，只不过是有序的执行，首先 View 会在退场时，执行淡出，并伴随大小和位置的变化，然后在进场是执行淡入。 如果不指定 beginDelayedTransition 的第二个参数，默认的转场效果就是 AutoTransition 。

## 移植

 所以，既然这个框架这么好用，我们为什么不用呢。但是你懂得， 作为开发者，我们都想一劳永逸，同样的代码能够在不同的 Android 版本准确无误的执行并达到自己的想要的效果，这是我们最开心的事。但是这个框架只能在 Andorid 4.4 以上使用。

天下无难事，只怕有心人。好消息是我们能够在不同版本使用 Transitions API，下面有2个十分相似的开源库，能够做到 API 低版本兼容：

[TransitionsBackport](http://github.com/guerwan/TransitionsBackport)

[TransitionSupportLibrary](http://github.com/pardom/TransitionSupportLibrary)

不过他们已经不再维护了，而且有些特性已经落后于官方。所以开源库作者 [andkulikov](https://github.com/andkulikov) 在他们的基础上，创建了自己的库，增加许多新的特性并且能兼容老的 Andorid 版本，并且从 Lollipop 到 Marshmallow 的所有新的 API 变化都被合并到这个库 。

所以，下面就是开源库 [Transitions-Everywhere][4] 登场了，Transitions-Everywhere 向后移植到 Android 4.O ,并且兼容 Android 2.2 +.

## 使用 Transitions-Everywhere ##

 首先是在 Gradle 中引入：


 {% highlight gradle %}

    dependencies {
         compile "com.andkulikov:transitionseverywhere:1.6.5"
    }
{% endhighlight %}



 将所有类包名为  android.transition.\* 的替代为 com.transitionseverywhere.*

 <div align="center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*obonx4eJ2OdYTWtkCzxqMQ.jpeg"  />
 </div>


 接下来就可以磨刀霍霍向牛羊了，我们可以设置 Transition 动画时长，差值器，延迟执行的时间：

 {% highlight java %}
    transition.setDuration(300);
    transition.setInterpolator(new FastOutSlowInInterpolator());
    transition.setStartDelay(200);
 {% endhighlight %}


下面让我们看下有哪些使用的 Transition .

### Slide （滑行）
 类似于 Fade transition 淡入淡出动画，继承与 Visibility 类，它能让新的 View 在出场或者退场时从一边滑动到另一边，比如：Slide(Gravity.RIGHT)：

 {% highlight java %}

View view = inflater.inflate(R.layout.fragment_slide, container, false);
final ViewGroup transitionsContainer = (ViewGroup) view.findViewById(R.id.transitions_container);
final TextView text = (TextView)transitionsContainer.findViewById(R.id.text);

transitionsContainer.findViewById(R.id.button)
               .setOnClickListener(new VisibleToggleClickListener() {   

                     @Override   
                     protected void changeVisibility(boolean visible) {           

                            TransitionManager.beginDelayedTransition(transition
                            sContainer, new Slide(Gravity.RIGHT));   
                            text.setVisibility(visible ? View.VISIBLE : View.GONE);
                   }});


{% endhighlight %}


<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*daO7zweWeE6DIQDC2x66cw.gif"  />
</div>


### Explode and Propagation (粒子扩散)
 使用 Explode 可以做粒子扩散的效果，粒子扩散的中心点可以通过 setEpicenterCallback 方法设定。具体扩散的效果可以通过 TransitionPropagation 设定，TransitionPropagation 会计算每个动画的开始延迟时间。比如默认情况下  Explode 使用的 CircularPropagation，这个是一个圆形扩散效果，每个元素执行扩散动画的延迟时间是其距中心的距离决定的。 我们使用 setPropagation 方法就可以设置 TransitionPropagation.

 下面这个例子，使用 RecyclerView 和 GridLayoutManager 构建一个网格布局，当我们点击某个特殊位置时，就会移除掉所有的元素：

 {% highlight java %}

  public void onClick(View clickedView) {
    // save rect of view in screen coordinates
    final Rect viewRect = new Rect();
    clickedView.getGlobalVisibleRect(viewRect);

    // create Explode transition with epicenter
    Transition explode = new Explode()
        .setEpicenterCallback(new Transition.EpicenterCallback() {
            @Override
            public Rect onGetEpicenter(Transition transition) {
                return viewRect;
            }
        });
    explode.setDuration(1000);
    TransitionManager.beginDelayedTransition(recyclerView, explode);

    // remove all views from Recycler View
    recyclerView.setAdapter(null);
}
{% endhighlight %}

<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*FEt3l_4BZd3MC1OKLgAFbA.gif"  />
</div>


### ChangeImageTransform

 ChangeImageTransform 可以对一个图片的矩阵信息进行变换，当我们改变 ImageView 的 scaleType 属性时，就非常有用。很多时候我们可以结合  ChangeBounds 来改变位置,大小以及 scaleType。

 {% highlight java %}
 TransitionManager.beginDelayedTransition(transitionsContainer, new TransitionSet()
    .addTransition(new ChangeBounds())
    .addTransition(new ChangeImageTransform()));

ViewGroup.LayoutParams params = imageView.getLayoutParams();
params.height = expanded ? ViewGroup.LayoutParams.MATCH_PARENT :
    ViewGroup.LayoutParams.WRAP_CONTENT;
imageView.setLayoutParams(params);

imageView.setScaleType(expanded ? ImageView.ScaleType.CENTER_CROP :
    ImageView.ScaleType.FIT_CENTER);

{% endhighlight %}

<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*Hw0YOgANc-Z5CS3DGlrgiw.gif"  />
</div>



### Path (路径) 动画


> “Real-world forces, like gravity, inspire an element’s movement along an arc rather than in a straight line.”
- Material Design guidelines.

使用 setPathMotion 方法，可以在任意两点之间的位置变换做路径动画,比如使用 ChangeBounds 改变 View的位置：

{% highlight java %}

TransitionManager.beginDelayedTransition(transitionsContainer,
    new ChangeBounds().setPathMotion(new ArcMotion()).setDuration(500));

FrameLayout.LayoutParams params = (FrameLayout.LayoutParams) button.getLayoutParams();
params.gravity = isReturnAnimation ? (Gravity.LEFT | Gravity.TOP) :
    (Gravity.BOTTOM | Gravity.RIGHT);
button.setLayoutParams(params);

{% endhighlight %}


<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*9RHBNUVdHJXbmCJCx6K-qQ.gif"  />
</div>



### TransitionName

当我们需要移除父容器内所有的 view，然后再增加一些新的 view。这些元素可能非常相似，我们怎么能够让 Transition 框架分清哪些元素是被移除的，哪些元素是需要移动到新的位置呢？ 这个简单，我们只需要调用 TransitionManager.setTransitionName(View v, String transitionName)  方法就好了，第一参数传入想要标记的 view，在第二个参数传入一个唯一的标识符。这样就能可以保证每个 View 的 Transitions 的唯一性。

例如我们想创建一个标题 list。每次点击按钮的时候，这些新创建的 view 在出现时做相对运动动画：


{% highlight java %}

createViews(inflater, layout, titles);
shuffleButton.setOnClickListener(new View.OnClickListener() {

    @Override
    public void onClick(View v) {
        TransitionManager.beginDelayedTransition(layout, new ChangeBounds());
        Collections.shuffle(titles);
        createViews(inflater, layout, titles);
    }

});

// In createViews we should provide transition name for every view.

private static void createViews(LayoutInflater inflater, ViewGroup layout, List<String> titles) {
    layout.removeAllViews();
    for (String title : titles) {
        TextView textView = (TextView) inflater.inflate(R.layout.text_view, layout, false);
        textView.setText(title);
        TransitionManager.setTransitionName(textView, title);
        layout.addView(textView);
    }
}

{% endhighlight %}


<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*ezmDeXOkoZhAb7AaF3O7iA.gif"  />
</div>

![]()

### Scale


这个并不是官方的 API，而是开源库作者自己新加的特性。我们可以在 view 的可见性变换时做缩放动画，只需要添加 new Scale() 就可以了。

<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*0OFXGCH5paD7ljAMjwbq8Q.gif"  />
</div>




当然这个动画可以和其他的动画一起执行，例如 Fade:

{% highlight java %}

TransitionSet set = new TransitionSet()
    .addTransition(new Scale(0.7f))
    .addTransition(new Fade())
    .setInterpolator(visible ? new LinearOutSlowInInterpolator() :
        new FastOutLinearInInterpolator());

TransitionManager.beginDelayedTransition(transitionsContainer, set);
text2.setVisibility(visible ? View.VISIBLE : View.INVISIBLE);
{% endhighlight %}


<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*nVcdrcIEVnGgXuMMJbceBQ.gif"  />
</div>


### Recolor

这个酷，它能给 View 的背景，TextView 的字体颜色加上颜色渐变动画：

{% highlight java %}

TransitionManager.beginDelayedTransition(transitionsContainer, new Recolor());

button.setTextColor(getResources().getColor(!isColorsInverted ? R.color.second_accent :R.color.accent));
button.setBackgroundDrawable(
    new ColorDrawable(getResources().getColor(!mColorsInverted ? R.color.accent :
        R.color.second_accent)));

{% endhighlight %}


<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*ugwT8xl3jEuLRf2ITPudYg.gif"  />
</div>



### Rotate

彪悍的人生不需解释，你懂得：

{% highlight java %}

TransitionManager.beginDelayedTransition(transitionsContainer, new Rotate());
icon.setRotation(isRotated ? 135 : 0);
{% endhighlight %}


<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*SAMn0rQ9vk5hd3kTKuvzWA.gif"  />
</div>



### ChangeText  

可以给 TextView 的文本内容变换加上淡入淡出动画：

{% highlight java %}

TransitionManager.beginDelayedTransition(transitionsContainer,
    new ChangeText().setChangeBehavior(ChangeText.CHANGE_BEHAVIOR_OUT_IN));
 textView.setText(isSecondText ? TEXT_2 : TEXT_1);
 {% endhighlight %}


 <div align="center">
  <img src="https://cdn-images-1.medium.com/max/1600/1*RwU04vBzskxwBr7cwwBQvg.gif"  />
 </div>



## Targets

配置 Transitions 也非常容易，你可以给一些特殊目标的 View 指定 Transitions，仅仅只有它们才能有动画.

增加动画目标：

 - addTarget(View target) .  view
 - addTarget(int targetViewId).  通过view 的id
 - addTarget(String targetName)  .与 TransitionManager
.setTransitionName 方法设定的标识符相对应。

- addTarget(Class targetType) . 类的类型 ，比如android.
widget.TextView.class。


移除动画目标：

- removeTarget(View target)
- removeTarget(int targetId)
- removeTarget(String targetName)
- removeTarget(Class target)

排除不想做动画的view：

- excludeTarget(View target, boolean exclude)
- excludeTarget(int targetId, boolean exclude)
- excludeTarget(Class type, boolean exclude)
- excludeTarget(Class type, boolean exclude)

排除某个 ViewGroup 的所有子 View：

- excludeChildren(View target, boolean exclude)
- excludeChildren(int targetId, boolean exclude)
- excludeChildren(Class type, boolean exclude)


## 使用 xml 创建 Translation

使用 xml 也可以创建 Translation，需要将 Translation 资源放在 res/anim 目录，例如：

{% highlight xml %}

<?xml version="1.0" encoding="utf-8"?>
<transitionSet xmlns:app="http://schemas.android.com/apk/res-auto"
              app:transitionOrdering="together"
              app:duration="400">
    <changeBounds/>
    <changeImageTransform/>
    <fade
       app:fadingMode="fade_in"
       app:startDelay="200">
        <targets>
            <target app:targetId="@id/transition_title"/>
        </targets>
    </fade>
</transitionSet>

// And inflating:
TransitionInflater.from(getContext()).inflateTransition(R.anim.my_the_best_transition);
{% endhighlight %}




## Activity 和 Fragment 动画

这个戳到痛点了，Activity 和 Fragment 动画没法搞，所以需要自己手动撸了。

## 自定义 Transitions

Transitions API 内提供了一些常用的 Transitions，但是业务场景千变万化，某些情况还是需要我们自定义一些 Transitions.

自定 Transitions ，我们需要实现三个方法：captureStartValues，captureEndValues 和 createAnimator.前面两个方法用来捕捉 view 在转场前后的状态。

下面一个例子，我们创建一个平滑滑动的水平进度条：

ProgressBar：

{% highlight java %}


private class ProgressTransition extends Transition {

    /**
     * Property is like a helper that contain setter and getter in one place
     */
    private static final Property<ProgressBar, Integer> PROGRESS_PROPERTY =
        new IntProperty<ProgressBar>() {

        @Override
        public void setValue(ProgressBar progressBar, int value) {
            progressBar.setProgress(value);
        }

        @Override
        public Integer get(ProgressBar progressBar) {
            return progressBar.getProgress();
        }
    };

    /**
      * Internal name of property. Like a intent bundles
      */
    private static final String PROPNAME_PROGRESS = "ProgressTransition:progress";

    @Override
    public void captureStartValues(TransitionValues transitionValues) {
        captureValues(transitionValues);
    }

    @Override
    public void captureEndValues(TransitionValues transitionValues) {
        captureValues(transitionValues);
    }

    private void captureValues(TransitionValues transitionValues) {
        if (transitionValues.view instanceof ProgressBar) {
            // save current progress in the values map
            ProgressBar progressBar = ((ProgressBar) transitionValues.view);
            transitionValues.values.put(PROPNAME_PROGRESS, progressBar.getProgress());
        }
    }

    @Override
    public Animator createAnimator(ViewGroup sceneRoot, TransitionValues startValues,
            TransitionValues endValues) {
        if (startValues != null && endValues != null && endValues.view instanceof ProgressBar) {
            ProgressBar progressBar = (ProgressBar) endValues.view;
            int start = (Integer) startValues.values.get(PROPNAME_PROGRESS);
            int end = (Integer) endValues.values.get(PROPNAME_PROGRESS);
            if (start != end) {
                // first of all we need to apply the start value, because right now
                // the view is have end value
                progressBar.setProgress(start);
                // create animator with our progressBar, property and end value
                return ObjectAnimator.ofInt(progressBar, PROGRESS_PROPERTY, end);
            }
         }
         return null;
    }

{% endhighlight %}


下面使用新创建的 ProgressTransition:


{% highlight java %}

private void setProgress(int value) {
    TransitionManager.beginDelayedTransition(mTransitionsContainer, new ProgressTransition());
    value = Math.max(0, Math.min(100, value));
    mProgressBar.setProgress(value);
}
{% endhighlight %}


效果：

<div align="center">
 <img src="https://cdn-images-1.medium.com/max/1600/1*GOiOF1beDE7A-IQOUgTj5g.gif"  />
</div>



## 结束语：

所有的例子都在开源库中：[github.com/andkulikov/transitions-everywhere][5]

Talk is cheap,Reading the code.

  [1]: https://github.com/andkulikov/Transitions-Everywhere
  [2]: https://medium.com/@andkulikov/animate-all-the-things-transitions-in-android-914af5477d50#.vvwbwmb9j
  [3]: https://www.google.com/design/spec/motion/material-motion.html
  [4]: https://github.com/andkulikov/Transitions-Everywhere
  [5]: github.com/andkulikov/transitions-everywhere
