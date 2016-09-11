---
layout: post
title:  "Android Animations"
date:   2016-09-12 00:22:33 +0800
categories: android animation
tags: [ android, animation ]
---
<p>In this post I'll create an app which demonstrates how animations are implemented.
First we will create the layout.  Our layout has one image view and four buttons representing each of the following tween animation, alpha, rotate, scale, translate.</p>

Why use animations?
Animations can make your app look and feel better.

Why not use animations?
Adding too much animation can slow down your app.

Two types of View animation:
Tween - Perform one or more transformations to a single view.

Fade in/out
Zoom in/out
Rotating
Card Flipping

<b>activity_main.xml</b>

{% highlight xml linenos %}
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:paddingBottom="@dimen/activity_vertical_margin"
    android:paddingLeft="@dimen/activity_horizontal_margin"
    android:paddingRight="@dimen/activity_horizontal_margin"
    android:paddingTop="@dimen/activity_vertical_margin"
    tools:context="com.emmanuelcorrales.animationsdemo.MainActivity">

    <ImageView
        android:id="@+id/imageView"
        android:layout_width="150dp"
        android:layout_height="150dp"
        android:layout_centerInParent="true"
        android:src="@mipmap/ic_launcher" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        android:orientation="horizontal">

        <Button
            android:id="@+id/rotate"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onClick"
            android:text="@string/rotate" />

        <Button
            android:id="@+id/translate"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onClick"
            android:text="@string/translate" />

        <Button
            android:id="@+id/scale"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onClick"
            android:text="@string/scale" />

        <Button
            android:id="@+id/alpha"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:onClick="onClick"
            android:text="@string/alpha" />
    </LinearLayout>

</RelativeLayout>
{% endhighlight %}

<p>In our res folder we create a new directory called anim which will store all our animations as xml files.</p>

<h4>Fade in/out Animation</h4>

<p>To implement the fade in/out animation. We will create a file called alpha.xml inside the anim folder.</p>

<b>res/anim/aplha.xml</b>

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<alpha xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:fromAlpha="1.0"
    android:toAlpha="0.0" />
{% endhighlight %}

<p>This animation makes the view fade out after 1 second. The attribute android:fromAlpha represents the initial value of the view's transparency while android:toAlpha tag represents its final value after the animation. The values should be between 0.0 to 1.0. The lower the value the more transparent the view will be and the higher the value the more opaque the view will become. The android:duration represents the interval of how long this animation will occur.</p>

<h4>Rotate Animation</h4>

<p>To implement the rotate animation. We will create a file called rotate.xml inside the anim folder.</p>

res/anim/rotate.xml

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<rotate xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1500"
    android:fromDegrees="0"
    android:pivotX="50%"
    android:pivotY="50%"
    android:toDegrees="360" />
{% endhighlight %}

<p>This animation rotates the view clockwise for 1.5 seconds. The attribute android:fromDegrees represents the starting angle of the view while the attribute android:toDegrees represents the ending angle of the view. The attribute android:pivotX represents the x-coordinate of the rotation while the attribute android:pivotY represents the y-coordinate of the rotation.</p>

<h4>Scale Animation</h4>

<p>To implement the scale animation. We will create a file called scale.xml inside the anim folder.</p>

<b>res/anim/scale.xml</b>

{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<scale xmlns:android="http://schemas.android.com/apk/res/android"
    android:duration="1000"
    android:fromXScale="1.0"
    android:fromYScale="1.0"
    android:pivotX="50%"
    android:pivotY="50%"
    android:toXScale="2.0"
    android:toYScale="2.0" />
{% endhighlight %}


<p>This animation scales the view to twice its original size in 1 second. The attributes android:fromXScale and android:fromYScale represents the starting x and y size offsets of the view. The value of the attributes android:fromXScale and android:fromYScale from the xml above is 1.0 which means that the view will retain its size at the start of the animation. The attributes android:toXScale and android:toYScale represents the ending x and y size offsets of the view. The value of the attributes android:toXScale and android:toYScale from the xml above is 2.0 which means that the view will be twice its size by the end of the animation. The attributes android:pivotX and android:pivotY represents the x and y coordinate to remain fixed when the object is scaled.</p>

<h4>Translate Animation</h4>

<p>To implement the scale animation. We will create a file called scale.xml inside the anim folder.</p>

<b>res/anim/translate.xml</b>
