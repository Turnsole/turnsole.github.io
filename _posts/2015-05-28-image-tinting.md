---
layout: post
title:  "Image tinting on Android"
date:   2015-05-28 12:19:30
categories: android 
---

##Basics 

Drawable tinting is a useful development technique on Android, it lets you reuse drawables and create nice-looking overlays on background images. While there are some Lollipop specific features that deserve their own post, this post is focused on something that's been available since api level 4: Duff-Porter blending. 

Duff-Porter modes are twelve rules[1] that determine the RGBA values of a pixel based on two input pixels. This produces a composite image; we can use it to change the color of a bitmap. For example, here's a kitty: 

![Original Cat](/images/demonstration_cat.png)

You can create an image filter that uses Duff-Porter rules like so: 

{% highlight java %}
private void tintImage(int color, PorterDuff.Mode mode, ImageView imageView) {
	PorterDuffColorFilter porterDuffColorFilter = new PorterDuffColorFilter(color, mode);
	imageView.setColorFilter(porterDuffColorFilter);
}
{% endhighlight %}

There are twelve rules, each with a PorterDuff.Mode, but many of them aren't distinct when you're blending a solid color with an image. To tint opaque pixels with another color, use PorterDuff.Mode.SRC_IN. 

![Cyan Cat](/images/demonstration_cat_cyan.png)

##DrawableCompat 

Using the v4 support library, you can take advantage of Lollipop's tinting for builds where the minimum SDK is earlier. Unfortunately, it does no actual tinting unless the device is running Lollipop or higher. 

{% highlight java %}
ImageView icon = (ImageView) findViewById(R.id.icon);

// This has no effect on devices running a version < Lollipop. 
DrawableCompat.setTint(icon.getDrawable(), color);
DrawableCompat.setTintMode(icon.getDrawable(), PorterDuff.Mode.SRC_IN);
{% endhighlight %}

##Themes 
Now let's take this a step further. 

Imagine that you have some solid colored icons that get used throughout your app, but their color depends on what context they're in. You can make all your icons white (best for straightforward blending), create a theme for each context in the app, and tint the icons with a theme color.

attrs.xml:
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <attr name="colorIcons" format="reference|color" />
</resources>
{% endhighlight %}

Activity theme:
{% highlight xml %}
<item name="colorIcons">@color/red</item>
{% endhighlight %}

Get the appropriate color from the theme for a given Activity: 
{% highlight java %}
TypedValue typedValue = new TypedValue();
getTheme().resolveAttribute(R.attr.colorIcons, typedValue, true);
int color = typedValue.data;

tintImage(color, PorterDuff.Mode.SRC_IN, icon);
{% endhighlight %}

Now each time you tweak your theme, all these icons will be displayed in the appropriate colour, and you haven't had to touch them or create duplicates for each section of the app. 

That custom attribute you just created is accessible from XML as well. In fact, if the image in question is in a static layout, you can define the tint there rather than adding a filter in Java. (Works at least back to api level 10.)

{% highlight xml %}
    <ImageView
        android:src="@drawable/ic_action_search"
        android:id="@+id/icon"
        android:contentDescription="@string/link_search"
        android:layout_height="wrap_content"
        android:layout_width="wrap_content"
        android:tint="?colorIcons"/>
{% endhighlight %}

[1] Porter, Thomas; Tom Duff (1984). "Compositing Digital Images". Computer Graphics 18 (3): 253–259