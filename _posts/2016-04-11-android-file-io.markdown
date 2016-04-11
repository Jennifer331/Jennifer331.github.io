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
<img alt="Java Files IO Methods" src="{{site.baseurl}}/assets/images/android-file-io-filesMethod.png" width="50%" align="center" style="margin: 0px 15px">
Java的Files类里有很多移动复制的方法，但是，Android并不包含java.nio.file这个包，所以并没有Files这个类可以用。

# FileChannel的独特之处
__getChannel__

看代码时有看到用RandomAccessFile的getChannel()方法的，有用FileInputStream的getChannel()的，跟着代码看下去，其实都没有差，都只是把fd和读写模式传给FileChannelImpl罢了

------------

RandomAccessFile.java
<img alt="Java Files IO Methods" src="{{site.baseurl}}/assets/images/android-file-io-getChannel.png" width="50%" align="center" style="margin: 0px 15px">

-----------

FileInputStream.java
<img alt="Java Files IO Methods" src="{{site.baseurl}}/assets/images/android-file-io-FileInputStream-getChannel.png" width="50%" align="center" style="margin: 0px 15px">

--------------

NioUtils.java
<img alt="Java Files IO Methods" src="{{site.baseurl}}/assets/images/android-file-io-NioUtils.png" width="50%" align="center" style="margin: 0px 15px">


从代码看下去，FileInputeStream在使用Libcore.os.read()，而Channel使用的是Libcore.os.readv(),（具体实现看Posix.java）“readv和writev函数用于在一次函数调用中读，写多个非连续缓冲区。”，即“散布读，聚集写”，这里有一篇专门介绍[readv()和writev()函数](http://book.2cto.com/201212/11769.html)的文章。我的理解是Channel的这种实现可以从多个起点开工同时开始读写，而这些起点实际上都是连续的，所以，这样一下就读进了一个buffer？Java官方文档这样介绍“While stream I/O reads a character at a time, channel I/O reads a buffer at a time. ”
