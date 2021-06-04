---
title: kotlin的let、apply、run、also、with、repeat、takeIf、takeUnless
date: 2018-01-04 17:44:27
tags: 
- Android
- 源码
- kotlin
categories: Android
---

直接上源码
```
/**
 * Calls the specified function [block] and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <R> run(block: () -> R): R = block()

/**
 * Calls the specified function [block] with `this` value as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> T.run(block: T.() -> R): R = block()

/**
 * Calls the specified function [block] with the given [receiver] as its receiver and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()

/**
 * Calls the specified function [block] with `this` value as its receiver and returns `this` value.
 */
@kotlin.internal.InlineOnly
public inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }

/**
 * Calls the specified function [block] with `this` value as its argument and returns `this` value.
 */
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.also(block: (T) -> Unit): T { block(this); return this }

/**
 * Calls the specified function [block] with `this` value as its argument and returns its result.
 */
@kotlin.internal.InlineOnly
public inline fun <T, R> T.let(block: (T) -> R): R = block(this)

/**
 * Returns `this` value if it satisfies the given [predicate] or `null`, if it doesn't.
 */
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? = if (predicate(this)) this else null

/**
 * Returns `this` value if it _does not_ satisfy the given [predicate] or `null`, if it does.
 */
@kotlin.internal.InlineOnly
@SinceKotlin("1.1")
public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? = if (!predicate(this)) this else null

/**
 * Executes the given function [action] specified number of [times].
 *
 * A zero-based index of current iteration is passed as a parameter to [action].
 */
@kotlin.internal.InlineOnly
public inline fun repeat(times: Int, action: (Int) -> Unit) {
    for (index in 0..times - 1) {
        action(index)
    }
```
先说区别，apply  also  会返回自身，用法有点像建造者模式。而let、run可以在最后一行返回任意东西
#run
```
        val i = 4.0
        val j = i.run {
            println(this)
            this+2
        }
        println(j)
```
#let
```
        val i = 4.0
        val j = i.let {
            println(it)
            it+2
        }
        println(j)
```
输出都是4.0   6.0
可以看出run和let都可以在最后一行进行return。不同在于run是用this进行引用i，而let是用it。
#apply
```
        val i = 4.0
        val j = i.apply {
           println(this)
            println(toInt())
        }
        println(i==j)
```
#also  SinceKotlin1.1
```
        val i = 4.0
        val j = i.also {
            println(it)
            println(it.toInt())
        }
        println(i==j)
```
返回的是4.0   4   true
可以看到，apply和also也都可以对i进行操作，区别是apply是用this来引用i，而also是用it。是不是跟上面的run和let很像？  apply和also返回的是自身，run和let是可以随意返回的。
#with
```
        val i = 4.0
        val b = with(i){
            when  {
                this>0 -> true
                else -> false
            }
        }
        println(b)
```
输出的是true
with不是一个扩展，可以在方法体内通过this调用，并且可以返回任意。

#repeat
```
        val list = mutableListOf<Int>()
        kotlin.repeat(3){
            list.add(it)
        }
        println(list)
```
repeat就是重复做一些什么事情咯,传3就是从0,1,2  每次操作一次list.add(it),it就是那个0,1,2

#takeIf takeUnless
```
        val i = 4.0
        val j = i.takeIf { it>0 }
        val k = i.takeUnless { it>0 }
        println(j)
        println(k)
```
takeIf需要在方法体内最后一行返回个Boolean，如果满足条件，则返回自身，否则返回null;
takeUnless相反。








