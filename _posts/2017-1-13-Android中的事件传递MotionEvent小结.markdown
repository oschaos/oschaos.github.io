---
layout: post
title: "Android中事件传递MotionEvent小结"
subtitle: "年前准备"
date: 2017-1-13
author: "chaos"
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
    - Android
---

 > 该文只是随笔小结，有时间会完善


## Android中的Touch事件

Android中的touch事件都封装在MotionEvent中
- ACTION_DOWN
- ACTION_MOVE
- ACTION_UP		
- ACTION_CANCEL	
...

处理这些事件，通常用到的主要时以下三种方法

* boolean dispatchTouchEvent() 事件分发
* boolean onInterceptTouchEvent() 事件拦截
* boolean onTouchEvent() 事件消费

<hr/>

## Touch 事件测试@test


<p><b>首先建立三个嵌套的view，有外向内一次为bottomview（粉色）、midddleviet（蓝色）、topView（黄色） </b></p> 

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/activity_main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.github.chaos.chaosblog.MainActivity">


    <com.github.chaos.chaosblog.view.ViewBottom
        android:id="@+id/view_bottom"
        android:layout_width="300dp"
        android:layout_height="300dp"
        android:layout_centerInParent="true"
        android:background="@color/colorAccent"
        android:gravity="center">

        <com.github.chaos.chaosblog.view.ViewMiddle
            android:id="@+id/view_middle"
            android:layout_width="200dp"
            android:layout_height="200dp"
            android:background="@color/colorPrimaryDark"
            android:gravity="center">

            <com.github.chaos.chaosblog.view.ViewTop
                android:id="@+id/view_top"
                android:layout_width="100dp"
                android:layout_height="100dp"
                android:background="#ff0" />


        </com.github.chaos.chaosblog.view.ViewMiddle>


    </com.github.chaos.chaosblog.view.ViewBottom>

</RelativeLayout>

```

<p>这三个自定义的view都是继承自LinearLayout，重写里面的 dispatchTouchEvent() onInterceptTouchEvent() onTouchEvent()三个方法，在里面打个log</p>
```java

	@Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        Clog.e("Touch","ViewMiddle-->dispatchTouchEvent");
        return super.dispatchTouchEvent(ev);
    }

    @Override
    public boolean onInterceptTouchEvent(MotionEvent ev) {
        Clog.e("Touch","ViewMiddle-->onInterceptTouchEvent");
        return super.onInterceptTouchEvent(ev);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        Clog.e("Touch","ViewMiddle-->onTouchEvent");
        return super.onTouchEvent(event);

    }
```


## 查看touch事件的传递和执行情况
<!-- 1 默认情况-->
### 1、默认情况，三个函数都走super(),运行情况
<pre>
E/Touch: ViewBottom-->dispatchTouchEvent
E/Touch: ViewBottom-->onInterceptTouchEvent
E/Touch: ViewMiddle-->dispatchTouchEvent
E/Touch: ViewMiddle-->onInterceptTouchEvent
E/Touch: ViewTop-->dispatchTouchEvent
E/Touch: ViewTop-->onInterceptTouchEvent
E/Touch: ViewTop-->onTouchEvent
E/Touch: ViewMiddle-->onTouchEvent
E/Touch: ViewBottom-->onTouchEvent
E/Touch: Activity--ACTION_DOWN
E/Touch: Activity--ACTION_UP
</pre>
在默认情况下，可以得出以下结论
<li>touch事件是由外层向内层传递，touch事件是由内向外执行的</li>
<li>touch事件假如不消费的话，最终会被acivity消费掉</li>
<hr>


<!-- 2 viewmiddle 不分发 dispatchTouchEvent() 返回false-->
### 2、ViewMiddle 不分发 dispatchTouchEvent() 返回false
<pre>
01-13 21:19:57.163 19817-19817/com.github.chaos.chaosblog E/Touch: ViewBottom-->dispatchTouchEvent
01-13 21:19:57.163 19817-19817/com.github.chaos.chaosblog E/Touch: ViewBottom-->onInterceptTouchEvent
01-13 21:19:57.163 19817-19817/com.github.chaos.chaosblog E/Touch: ViewMiddle-->dispatchTouchEvent
01-13 21:19:57.163 19817-19817/com.github.chaos.chaosblog E/Touch: ViewBottom-->onTouchEvent
01-13 21:19:57.163 19817-19817/com.github.chaos.chaosblog E/Touch: Activity--ACTION_DOWN
01-13 21:19:57.190 19817-19817/com.github.chaos.chaosblog E/Touch: Activity--ACTION_MOVE
01-13 21:19:57.200 19817-19817/com.github.chaos.chaosblog E/Touch: Activity--ACTION_UP
</pre>
在不分发情况下，可以得出以下结论
<li>touch事件在哪层开始不分发，则当前层view也包含在内不会进行 touch 事件传递</li>
<li>当事件开始不分发了，则它的parent则开始执行onTouchEvent，执行传递过程中都没有消费掉，最终由activity消费掉</li>
<hr>

<!-- 3 viewmiddle 拦截 onInterceptTouchEvent()返回true-->
### 3、viewmiddle 拦截 onInterceptTouchEvent()返回true
<pre>
01-13 21:27:35.748 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->dispatchTouchEvent
01-13 21:27:35.749 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->onInterceptTouchEvent
01-13 21:27:35.749 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->dispatchTouchEvent
01-13 21:27:35.749 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->onInterceptTouchEvent
01-13 21:27:35.749 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->onTouchEvent
01-13 21:27:35.749 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->onTouchEvent
01-13 21:27:35.749 29838-29838/com.github.chaos.chaosblog E/Touch: Activity--ACTION_DOWN
01-13 21:27:35.769 29838-29838/com.github.chaos.chaosblog E/Touch: Activity--ACTION_MOVE
01-13 21:27:35.789 29838-29838/com.github.chaos.chaosblog E/Touch: Activity--ACTION_UP

</pre>
在拦截情况下，可以得出以下结论
<li>touch事件在哪层开始拦截，则在哪层开始执行onTouchEvent()也就是开始消费判断</li>
<li>touch事件在执行过程中，假若没有view对其进行消费，最终会被activity消费掉</li>
<hr>

<!-- 4 viewmiddle 事件消费 onTouchEvent()返回true -->
### 4、viewmiddle 事件消费 onTouchEvent()返回true
<pre>
01-13 21:35:41.496 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->dispatchTouchEvent
01-13 21:35:41.496 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->onInterceptTouchEvent
01-13 21:35:41.496 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->dispatchTouchEvent
01-13 21:35:41.496 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->onInterceptTouchEvent
01-13 21:35:41.496 29838-29838/com.github.chaos.chaosblog E/Touch: ViewTop-->dispatchTouchEvent
01-13 21:35:41.496 29838-29838/com.github.chaos.chaosblog E/Touch: ViewTop-->onInterceptTouchEvent
01-13 21:35:41.496 29838-29838/com.github.chaos.chaosblog E/Touch: ViewTop-->onTouchEvent
01-13 21:35:41.496 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->onTouchEvent
01-13 21:35:41.496 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->ACTION_DOWN
01-13 21:35:41.565 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->dispatchTouchEvent
01-13 21:35:41.565 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->onInterceptTouchEvent
01-13 21:35:41.565 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->dispatchTouchEvent
01-13 21:35:41.565 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->onTouchEvent
01-13 21:35:41.565 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->ACTION_MOVE
01-13 21:35:41.580 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->dispatchTouchEvent
01-13 21:35:41.580 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->onInterceptTouchEvent
01-13 21:35:41.580 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->dispatchTouchEvent
01-13 21:35:41.580 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->onTouchEvent
01-13 21:35:41.580 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->ACTION_MOVE
01-13 21:35:41.581 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->dispatchTouchEvent
01-13 21:35:41.581 29838-29838/com.github.chaos.chaosblog E/Touch: ViewBottom-->onInterceptTouchEvent
01-13 21:35:41.581 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->dispatchTouchEvent
01-13 21:35:41.581 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->onTouchEvent
01-13 21:35:41.581 29838-29838/com.github.chaos.chaosblog E/Touch: ViewMiddle-->ACTION_UP

</pre>
在事件消费的情况下，可以得出以下结论
<li>一旦在某层开始消费该事件，则接下来一系列 action 在当层被拦截了也就不会在下发</li>
<li>只有在action_down被消费了之后的action 才能执行，一个touch事件要被消费那么ACTION_DOWN必须消费</li>
<hr>
<br>

## 总结
<li>touch事件是由外向内即父容器向子容器传递，而它的执行onTouchEvent则由内向外(child view向parent view)</li>
<li>parentView通过onInterceptionTouchEvent将事件拦截在该层不向后传递 </li>
<li>dispatchTouchEvent可以组织事件在某一层下发，而onInterceptTouchEvent则将事件拦截在某一层</li>
<li>事件在执行的过程中，只有ACTION_DOWN消费了，后面的ACTION_EVENT才能被消费</li>







