---
title: Kotlin学习笔记part1-i_introduction
date: 2018-01-22 13:51:12
tags: Kotlin
---

这是「Kotlin学习笔记」的第1篇，[第0篇在这里](http://blog.sealynn0.cn/2018/01/22/Kotlin-learning-diary0/)。

<!-- more -->

### _0_Hello_World

具体请看上一篇文章的内容。

### _1_Java_To_Kotlin_Converter

这个任务的要求就是将一段Java代码转换成Kotlin代码，提示可以直接将Java代码复制粘贴，然后使用Intellij提供的Convert Java File to Kotlin File功能（仅仅是这个任务允许这样做），非常便捷。

```
//Java
public class JavaCode1 extends JavaCode {
    public String task1(Collection<Integer> collection) {
        StringBuilder sb = new StringBuilder();
        sb.append("{");
        Iterator<Integer> iterator = collection.iterator();
        while (iterator.hasNext()) {
            Integer element = iterator.next();
            sb.append(element);
            if (iterator.hasNext()) {
                sb.append(", ");
            }
        }
        sb.append("}");
        return sb.toString();
    }
}
//Kotlin
    fun task1(collection: Collection<Int>): String {
        val sb = StringBuilder()
        sb.append("{")
        val iterator = collection.iterator()
        while (iterator.hasNext()) {
            val element = iterator.next()
            sb.append(element)
            if (iterator.hasNext()) {
                sb.append(", ")
            }
    }
    sb.append("}")
    return sb.toString()
}
```
这一段代码两者之间没有明显的差别，但是在下一个任务中可以看到Kotlin中这一段代码可以精简成一行代码。

***
*这篇文章是在1.22号创建，在那天了写了一点后就 打机、出去玩、还有就是写星空直播的安卓端Demo去了...  
不专注的辣鸡  
**今天（2.5号）回来继续完成这篇文章~***
***

### _2_Named_Arguments

