---
title: solution of gsonformat
date: 2021-02-02 14:25:36
tags: 
    - Android
categories: 
    - Dev
---

最近更新了Android Studio版本到4.1之后。每次打开都会出现一下提示。

```
Plugin Error: Plugin "GsonFormat" is incompatible (supported only in IntelliJ IDEA).
```

大概的意思GsonFormat插件和Android Studio版本不兼容，或者说只兼容IntelliJ IDEA，那解决办法一是要卸载GsonFormat和找到一个替代方案。

<!-- more -->

## 卸载GsonFormat

本来想通过File - Settings - Plugins这个路径下找到GsonFormat，并完成对其的卸载，然而问题是，我找不到，并没有显示在这。

那能否在磁盘上找到这个插件直接对其删除？通过一同检索，AS默认是把插件保存在

```
C:\Users\用户名\AppData\Roaming\Google\AndroidStudio4.1\plugins
```

这个路径下，找到GsonFormat.jar直接删除，重启就不会再有提示了。

## 替代方案

可以替代GsonFormat的插件有很多，比如说GsonFormatPlus，直接去Marketplace插件搜就好。但最近我在用纯Kotlin重构我的个人[玩Android](https://github.com/MichaelLynn1996/WanAndroid)项目，顺便学习。直接用Convert Java File to Kotlin File直接转换并不会将原来的bean转化为data class，也是直接去Marketplace搜Gson，结合百度发现了一款不错的插件[JsonToKotlinClass](https://github.com/wuseal/JsonToKotlinClass)。用起来还不错，感觉比原来GsonFormat还好用一些。

