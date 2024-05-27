---
title: Android 里的 dpi dp px
date: 2023-11-08 16:18:43
mathjax: true
tags:
- Android
- dpi
- dp
- px
thumbnail: "https://darkflamemasterdev.github.io/img/language_headimg.png"
---

## 概念

Dpi (Dots Per Inch)

每英寸点数，原本是从打印机过来的概念，对于显示屏幕，用 ppi

很多刚接触 `Android` 开发的小伙伴，对于这个 `dp` 一脸懵逼，百度了很久，也是一些计算`dpi` 的方法，根本没有讲到点子上！

我们需要先了解一下这个 `DisplayMetrics` 类

里面有几个常用的常量

```kotlin
Log.d("lucas", "density = ${resources.displayMetrics.density}")
Log.d("lucas", "densityDpi = ${resources.displayMetrics.densityDpi}")
Log.d("lucas", "heightPixels = ${resources.displayMetrics.heightPixels}")
Log.d("lucas", "widthPixels = ${resources.displayMetrics.widthPixels}")
Log.d("lucas", "xdpi = ${resources.displayMetrics.xdpi}")
Log.d("lucas", "ydpi = ${resources.displayMetrics.ydpi}")
```

```json
// 显示的逻辑密度，也就是一个比例。
// 其中 1 DIP等于一个屏幕密度为160 DPI的屏幕上的一个物理像素，提供系统显示的基线。
// 因此，在 160dpi 屏幕上，此密度值将为 1；在 120 dpi 的屏幕上，它将是 0.75。
// 此值并不完全遵循实际屏幕大小（由 xdpi 和 ydpi给出），而是用于根据显示 dpi 的粗略变化分步缩放整个 UI 的大小。
// 例如，即使 240x320 屏幕的宽度为 1.8 英寸、1.3 英寸等，其密度也将为 1。
// 但是，如果屏幕分辨率增加到 320x480，但屏幕尺寸仍为 1.5“x2”，则密度将增加（可能增加到 1.5）。
density = 2.75

// 就是屏幕密度以每英寸点数表示
densityDpi = 440

// 可用显示大小的绝对高度（以像素为单位）
heightPixels = 2250

// 可用显示大小的绝对宽度（以像素为单位）
widthPixels = 1080

// X 维度中每英寸屏幕的确切物理像素数。
xdpi = 386.366

// Y 维度中每英寸屏幕的确切物理像素数。
ydpi = 386.701
```

{% notel blue DIP??? %}

DIP（Density-Independent Pixel 密度无关像素）：

这个就是我们平时用的 dp

dp 要和 px 转化，要用到 `Android` 里面的 `density`

公式是：$1dp=density×1px$

density 并不会完全按照 dpi 进行非常紧密的比例变化，而是会有几个等级，

{% endnotel %}

| 密度限定符 | density | 说明                                                                                                                                                                                                                                                                                                                                                                                      |
| ---------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ldpi`     | 0.75    | 适用于低密度 (ldpi) 屏幕 (~ 120dpi) 的资源。                                                                                                                                                                                                                                                                                                                                              |
| `mdpi`     | 1.0     | 适用于中密度 (mdpi) 屏幕 (~ 160dpi) 的资源（这是基准密度）。                                                                                                                                                                                                                                                                                                                              |
| `hdpi`     | 1.5     | 适用于高密度 (hdpi) 屏幕 (~ 240dpi) 的资源。                                                                                                                                                                                                                                                                                                                                              |
| `xhdpi`    | 2.0     | 适用于加高 (xhdpi) 密度屏幕 (~ 320dpi) 的资源。                                                                                                                                                                                                                                                                                                                                           |
| `xxhdpi`   | 3.0     | 适用于超超高密度 (xxhdpi) 屏幕 (~ 480dpi) 的资源。                                                                                                                                                                                                                                                                                                                                        |
| `xxxhdpi`  | 4.0     | 适用于超超超高密度 (xxxhdpi) 屏幕 (~ 640dpi) 的资源。                                                                                                                                                                                                                                                                                                                                     |
| `nodpi`    | 1.0     | 适用于所有密度的资源。这些是与密度无关的资源。无论当前屏幕的密度是多少，系统都不会缩放以此限定符标记的资源。                                                                                                                                                                                                                                                                              |
| `tvdpi`    |         | 适用于密度介于 mdpi 和 hdpi 之间的屏幕（约 213dpi）的资源。这不属于“主要”密度组。它主要用于电视，而大多数应用都不需要它。对于大多数应用而言，提供 mdpi 和 hdpi 资源便已足够，系统将视情况对其进行缩放。如果您发现有必要提供 tvdpi 资源，应按一个系数来确定其大小，即 1.33*mdpi。例如，如果某张图片在 mdpi 屏幕上的大小为 100px x 100px，那么它在 tvdpi 屏幕上的大小应该为 133px x 133px。 |



网上通常是下面的转换方式

```kotlin
fun Context.dpToPx(dp: Float): Float {
  val density: Float = this.resources.displayMetrics.density
  return (dp * density)
}

fun Context.pxToDp(px: Float): Float {
  val density: Float = this.resources.displayMetrics.density
  return (px / density)
}
```

又或者这种

```kotlin
fun Float.toPx(): Float {
  return TypedValue.applyDimension(
    TypedValue.COMPLEX_UNIT_DIP,
    this,
    Resources.getSystem().displayMetrics
  )
}

@RequiresApi(34)
fun Float.toDp(): Float {
  return TypedValue.deriveDimension(
    TypedValue.COMPLEX_UNIT_DIP,
    this,
    Resources.getSystem().displayMetrics
  )
}
```

第一种需要 Context，第二种 Api 有兼容性问题，因为 `deriveDimension` 是在 [api34](https://developer.android.com/guide/topics/manifest/uses-sdk-element?hl=zh-cn#ApiLevels) 才添加进来的

所以我们只要用 `Resources.getSystem().displayMetrics.density` 就没有了以上的问题

```kotlin
fun Float.toPx(): Float {
  val density: Float = Resources.getSystem().displayMetrics.density
  return (this * density)
}

fun Float.toDp(): Float {
  val density: Float = Resources.getSystem().displayMetrics.density
  return (this / density)
}
```



adb 命令中 wm density 这里的数值 指的是 dpi 并不是刚才提到的那个比例

```shell
adb shell wm density 160
```

{% notel blue 提问：分辨率相同、尺寸不同的屏幕，将density设为相同的值，显示效果看起来是否一致的呢？ %}

**答案是：不一致！**

因为相同分辨率下，屏幕的物理尺寸越大，像素块的物理尺寸就会越大

如果屏幕的像素块的物理尺寸是不一样的，那显示效果是不一致的

也就是同样 100px 的大小，像素块的物理尺寸是不同的，所以实际显示的尺寸也是不一样的

{% endnotel %}
