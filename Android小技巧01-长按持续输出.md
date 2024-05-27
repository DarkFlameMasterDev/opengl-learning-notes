---
title: Android小技巧01-持续按钮
date: 2023-12-02 08:54:20
tags:
- Android
- 线程
- thread
- kotlin
- handler
thumbnail: "https://darkflamemasterdev.github.io/img/language_headimg.png"
---


### 问题

如果你要做一个功能：有 4 个按钮，按着按钮的时候，间隔 500ms 循环调用某个函数（比如打印一条 log，或者改某个数值），并且按下每个按钮，也不能影响其他按钮的按下状态，该怎么做？

### 解决思路

你当然会直接想到 setOnTouchListener ，但是很快就会发现，按钮的按下事件只会回调一次

{% notel orange 与多次回调的区别 %}

如果你调用过 C++ 的 glfw 或者 Java 的 JFrame KeyListener
你一定知道，按键是会一直回调的，当你按下一个按键的时候，他会一直给你回调输入的按键，频率基于采样率

```c++
if (glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS) {
  glfwSetWindowShouldClose(window, true);
}
```

我们可以直接判断回调事件，然后在每次事件后调用我们上面提到的“某个函数”

{% endnotel %}

但我们在 Android 这种只回调一次的情况该怎么办呢？

#### 新开子线程

每个按钮设置一个标志位，按下的时候，将标志位设为按下状态，然后再子线程里，循环输出这个结果

```kotlin
@Volatile
var is1Pressing = false
var is2Pressing = false
var is3Pressing = false
var is4Pressing = false
val ttt = Thread {
  while (true) {
    if (is1Pressing || is2Pressing || is3Pressing || is4Pressing)
      when {
        is1Pressing -> {
          log("thread name = ${Thread.currentThread().name}  button 1 is pressed")
        }

        is2Pressing -> {
          log("thread name = ${Thread.currentThread().name}  button 2 is pressed")
        }

        is3Pressing -> {
          log("thread name = ${Thread.currentThread().name}  button 3 is pressed")
        }

        is4Pressing -> {
          log("thread name = ${Thread.currentThread().name}  button 4 is pressed")
        }
      }
    Thread.sleep(500)
  }
}
```

代码结构就是这样，按下那个按钮，就更改哪个按键的状态

#### 使用主线程Handler

我们可以使用 postDelay 函数

```kotlin
private fun startPress(btn: Button) {
  pressStateSet(btn, true)
  handler.post(object : Runnable {
    override fun run() {
      log("isPressing = ${pressStateGet(btn)} time second = ${System.currentTimeMillis() / 1000 % 3600}")
      if (pressStateGet(btn)) {
        btnActionMap[btn.id]?.let {
          cameraBinding.cameraGLSurfaceView.processIn(it)
        }
        handler.postDelayed(this, 500)
      }
    }
  })
}

private fun stopPress(btn: Button) {
  pressStateSet(btn, false)
}
```

setOnTouchListener 传入 ACTION_DOWN 事件时，就调用这个函数，这个函数第 10 行，会把 runnable 自身 post 过去，除非 pressState 为 false

#### 模仿实现重复回调

{% notel orange 如果你需要实现重复回调的操作 %}

如果你需要实现多次回调的操作，就跟 C++ glfw 或者 Java Jframe 的样子，只需要将间隔书改为 16.6 ms（1s/60 ≈ 16.6ms）

当然，如果你的硬件刷新率为 120hz，也可以设成 8.8ms

{% endnotel %}
