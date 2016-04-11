---
layout: post
title: "Android File IO"
lang: zh
thumbnail: android-file-io-docopy.png
tags: [Android File]
---
# 问题咋来的呢
最近解一个bug，需要重构移动复制的代码，本来参考了[Apache的FileUtils](https://commons.apache.org/proper/commons-io/apidocs/src-html/org/apache/commons/io/FileUtils.html),里面是这样子copy的  
<img alt="Apache FileUtils copyFile" src="{{site.baseurl}}/assets/images/android-file-io-docopy.png" width="100%" horizontal-align="center" style="margin: 0px 15px">
审核代码的同事之前也没有用过FileChannel的transfer方法，就搜看看，结果，一搜把我搜傻了。。。[Java 复制大文件方式(nio2 FileChannel 拷贝文件能力测试)](http://blog.csdn.net/zhuyijian135757/article/details/38471595)
这篇文章比较了很多种复制方法的速度，__FileChannel__、__Input/Output Stream__、__Files传入Path的copy方法__，这些概念把我绕晕了，所以决定理一理思路。

# 一幅揭开迷雾的图
图是从[JAVA文档](http://docs.oracle.com/javase/tutorial/essential/io/file.html)里找来的。这幅图把JAVA提供的文件IO操作的方法从左到右按从简单到复杂的顺序排列。上排是方法名，下排是特性介绍。理得很清楚。
<img alt="Java File IO Methods" src="{{site.baseurl}}/assets/images/android-file-io-methods.gif" width="70%" align="center" style="margin: 0px 15px">

# Andorid API 并没有包含所有的Java API
<img alt="Java Files IO Methods" src="{{site.baseurl}}/assets/images/android-file-io-filesMethod.png" width="50%" align="left" style="margin: 0px 15px">
Java的Files类里有很多移动复制的方法，但是，Android并不包含java.nio.file这个包，所以并没有Files这个类可以用。
