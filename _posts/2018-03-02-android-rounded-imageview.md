---
layout: post
title:  "Rounded image view in Android"
date:   2018-03-02 11:00:00 +0200
categories: android
---

The common use-case is like this: you need to get an image from some server, resize and cache it (you’re using Picasso for this, aren’t you?), make corners rounded (ok, this is not _the most_ popular use-case, but it’s what this article is about) and load it into view. So, how can we make rounded corners?

As usual, we have several ways to achieve this in Android.

## Option #1. Overlay
Some desperate person can offer you to place your image view inside `FrameLayout`, add another image above it with a transparent body and solid corners and… you’re done. Do I need to tell you that this is very bad practice, and you should never do that?

## Option #2. clipPath
Rather a simple method. You just need to create a custom view with an overridden `onDraw` method of `ImageView` and use method `clipPath` on canvas instance there. The problem is that path clipping doesn’t support anti-aliasing, so your image will have ugly sharp edges.

##Option #3. BitmapShader
A way better option is to use a `Paint` with `BitmapShader`. It allows filling the rounded rectangle with a texture, not only with a simple color. Luckily we have RoundedBitmapDrawable within support library that uses this technique, we just need to combine image loading with Picasso and transforming it into one view, which leads us to option #4:

## Option #4. RoundedBitmapDrawable
So, let’s start. The code is written in Kotlin, because… well, why would you write in Java when you have Kotlin?

As we will use our view with Picasso only, we don’t even need to inherit it from `ImageView`, simple `View` would be enough:

```kotlin
class RoundedImageView(
  context: Context, 
  attributeSet: AttributeSet?
) : View(context, attributeSet), Target {
    constructor(context: Context) : this(context, null)
}
```

In order to allow Picasso to load an image into our view, it should implement `Target` interface, which includes 3 methods: `onPrepareLoad`, `onBitmapFailed` and `onBitmapLoaded`. All the magic is done in `onBitmapLoaded` method:

```kotlin
override fun onBitmapLoaded(bitmap: Bitmap?, from: Picasso.LoadedFrom?) {
    val roundedDrawable = RoundedBitmapDrawableFactory.create(resources, bitmap)
    roundedDrawable.cornerRadius = dip(DEFAULT_RADIUS).toFloat()
    drawable = roundedDrawable
}
```

Finally, we need to draw our `Drawable` in `onDraw` method (sounds very reasonable, isn’t it?):

```kotlin
override fun onDraw(canvas: Canvas?) {
    drawable?.setBounds(0, 0, width, height)
    drawable?.draw(canvas)
}
```

So the complete code together with helper method for loading image by URL string is here:

```kotlin
class RoundedImageView(context: Context, attributeSet: AttributeSet?) : View(context, attributeSet), Target {
    constructor(context: Context) : this(context, null)

    private var drawable: Drawable? = null
        set(value) {
            field = value
            postInvalidate()
        }

    fun loadImage(url: String?) {
        if (url == null) {
            drawable = null
        } else {
            Picasso.with(context)
                    .load(url)
                    .placeholder(R.drawable.image_stub)
                    .error(R.drawable.image_stub)
                    .into(this)
        }
    }

    override fun onDraw(canvas: Canvas?) {
        drawable?.setBounds(0, 0, width, height)
        drawable?.draw(canvas)
    }

    override fun onPrepareLoad(placeHolderDrawable: Drawable?) {
        drawable = placeHolderDrawable
    }

    override fun onBitmapFailed(errorDrawable: Drawable?) {
        drawable = errorDrawable
    }

    override fun onBitmapLoaded(bitmap: Bitmap?, from: Picasso.LoadedFrom?) {
        val roundedDrawable = RoundedBitmapDrawableFactory.create(resources, bitmap)
        roundedDrawable.cornerRadius = dip(DEFAULT_RADIUS).toFloat()
        drawable = roundedDrawable
    }

    companion object {
        private const val DEFAULT_RADIUS = 4
    }
}
```
