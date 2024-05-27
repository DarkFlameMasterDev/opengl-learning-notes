---
title: kotlin 协程-01 delay 函数（没写完）
date: 2023-10-25 15:01:33
tags:
thumbnail: "https://darkflamemasterdev.github.io/img/language_headimg.png"
---

#### 读代码困难

协程代码里有大量的 `Lambda` ，我每次看 `kotlin` 的 `Lambda` ，都有点头疼，但是不能退缩！！！

#### 看源码

我们先看这 `delay` 函数的定义

```kotlin
// Delay.kt

public suspend fun delay(timeMillis: Long) {
    if (timeMillis <= 0) return // don't delay
    return suspendCancellableCoroutine sc@ { cont: CancellableContinuation<Unit> ->
        // if timeMillis == Long.MAX_VALUE then just wait forever like awaitCancellation, don't schedule.
        if (timeMillis < Long.MAX_VALUE) {
            cont.context.delay.scheduleResumeAfterDelay(timeMillis, cont)
        }
    }
}
```

这里返回了一个 `Lambda` ，这个 `Lambda` 接受一个 `CancellableContinuation<Unit>` ，并调用了 `scheduleResumeAfterDelay` 方法

接下我们看看这个 `Lambda` 怎么定义的：

```kotlin
// CancellableCoroutine.kt

public suspend inline fun <T> suspendCancellableCoroutine(
    crossinline block: (CancellableContinuation<T>) -> Unit
): T =
    suspendCoroutineUninterceptedOrReturn { uCont ->
        val cancellable = CancellableContinuationImpl(uCont.intercepted(), resumeMode = MODE_CANCELLABLE)
        /*
         * For non-atomic cancellation we setup parent-child relationship immediately
         * in case when `block` blocks the current thread (e.g. Rx2 with trampoline scheduler), but
         * properly supports cancellation.
         */
        cancellable.initCancellability()
        block(cancellable)
        cancellable.getResult()
    }
```

这个 `suspendCancellableCoroutine` 接受了另一个 `Lambda` `block` ，这个 `block` 就是上面 `Delay.kt` 里面大括号里面的所有代码。

`suspendCancellableCoroutine` 通过 `suspendCoroutineUninterceptedOrReturn` 方法来执行逻辑

而且这是一个 `inline` 方法，这到不太影响我们来梳理代码逻辑，注意返回值即可

我们继续看 `suspendCoroutineUninterceptedOrReturn` ：

```kotlin
// Intrinsic.kt

public suspend inline fun <T> suspendCoroutineUninterceptedOrReturn(
    crossinline block: (Continuation<T>) -> Any?
): T {
    contract { callsInPlace(block, InvocationKind.EXACTLY_ONCE) }
    throw NotImplementedError("Implementation of suspendCoroutineUninterceptedOrReturn is intrinsic")
}
```

这里我们直接进入了 `Intrinsic.kt` 里，说明这里已经到了底层了，就没必要看了，我们继续回到 `CancellableCoroutine.kt` 

我们看到他新建了一个 `cancellable` ，然后调用 `initCancellability` ，然后执行了 `block` ，也就是执行了 `cancellable.context.delay.scheduleResumeAfterDelay(timeMillis, cancellable)`

前面都是初始化工作，关键就是这个，到此我们观察代码，最终定位到了这里，也就是一开始我们看到的那一点点代码

> 并且这些定位工作是值得的，他让我们可以放心地去专注于我们的主要逻辑了

我们点进 `scheduleResumeAfterDelay` 

```kotlin
@InternalCoroutinesApi
public interface Delay {

	...
	
    public fun scheduleResumeAfterDelay(
        timeMillis: Long,
        continuation: CancellableContinuation<Unit>
    )
    
    ...
}
```

发现这是一个 `Interface` 里的方法

那么我们就看看这个这个 `Interface` 是在哪实现的

我们进入这个 `delay` 

```kotlin
internal val CoroutineContext.delay: Delay get() = 
	get(ContinuationInterceptor) as? Delay ?: DefaultDelay
```

这里是在尝试将 `get(ContinuationInterceptor)` 的返回值转成 `Delay` ，失败就返回 `DefaultDelay`

看看这个 `DefaultDelay` 

```kotlin
// DefaultExecutor.kt

@PublishedApi
internal actual val DefaultDelay: Delay = initializeDefaultDelay()

private fun initializeDefaultDelay(): Delay {
    // Opt-out flag
    if (!defaultMainDelayOptIn) return DefaultExecutor
    val main = Dispatchers.Main
    /*
     * When we already are working with UI and Main threads, it makes
     * no sense to create a separate thread with timer that cannot be controller
     * by the UI runtime.
     */
    return if (main.isMissing() || main !is Delay) DefaultExecutor else main
}

@Suppress("PLATFORM_CLASS_MAPPED_TO_KOTLIN")
internal actual object DefaultExecutor : EventLoopImplBase(), Runnable {
    ...
}
```

可以看到，这里返回一个一个 `Dispacher.Main` 或者一个 `DefaultExecutor` 

而这个 `DefaultExecutor` 继承实现自 `EventLoopImplBase` ， `Runnable`

而 `EventLoopImpBase` 实现了 `Delay` 

```kotlin
// EventLoopCommon.kt

internal abstract class EventLoopImplBase: EventLoopImplPlatform(), Delay {
	...
	  override fun scheduleResumeAfterDelay(timeMillis: Long, continuation: CancellableContinuation<Unit>) {
        val timeNanos = delayToNanos(timeMillis)
        if (timeNanos < MAX_DELAY_NS) {
            val now = nanoTime()
            DelayedResumeTask(now + timeNanos, continuation).also { task ->
                /*
                 * Order is important here: first we schedule the heap and only then
                 * publish it to continuation. Otherwise, `DelayedResumeTask` would
                 * have to know how to be disposed of even when it wasn't scheduled yet.
                 */
                schedule(now, task)
                continuation.disposeOnCancellation(task)
            }
        }
    }
    ...
}
```



.
