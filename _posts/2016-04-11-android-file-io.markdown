---
layout: post
title: "Android File IO"
lang: zh
thumbnail: android-file-io-docopy.png
tags: [Android，File]
---
# <font color="#e3796b">问题咋来的呢<font>
最近解一个bug，需要重构移动复制的代码，本来参考了[Apache的FileUtils](https://commons.apache.org/proper/commons-io/apidocs/src-html/org/apache/commons/io/FileUtils.html),里面是这样子copy的  
<img alt="Apache FileUtils copyFile" src="{{site.baseurl}}/assets/images/android-file-io-docopy.png" width="100%" horizontal-align="center" style="margin: 0px 15px">
审核代码的同事之前也没有用过FileChannel的transfer方法，就搜看看，结果，一搜把我搜傻了。。。[Java 复制大文件方式(nio2 FileChannel 拷贝文件能力测试)](http://blog.csdn.net/zhuyijian135757/article/details/38471595)
这篇文章比较了很多种复制方法的速度，__FileChannel__、__Input/Output Stream__、__Files传入Path的copy方法__，这些概念把我绕晕了，所以决定理一理思路。

# <font color="#e3796b">一幅揭开迷雾的图</font>
图是从[JAVA文档](http://docs.oracle.com/javase/tutorial/essential/io/file.html)里找来的。这幅图把JAVA提供的文件IO操作的方法从左到右按从简单到复杂的顺序排列。上排是方法名，下排是特性介绍。理得很清楚。
<img alt="Java File IO Methods" src="{{site.baseurl}}/assets/images/android-file-io-methods.gif" width="70%" align="center" style="margin: 0px 15px">

# <font color="#e3796b">Andorid API 并没有包含所有的Java API</font>
<img alt="Java Files IO Methods" src="{{site.baseurl}}/assets/images/android-file-io-filesMethod.png" width="50%" align="center" style="margin: 0px 15px">
Java的Files类里有很多移动复制的方法，但是，Android并不包含java.nio.file这个包，所以并没有Files这个类可以用。

# <font color="#e3796b">FileChannel的独特之处</font>
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


__其实Channel的TransferTo()和TransferFrom()可以理解成另一种read()和write()实现__

TransferTo()调用Libcore.os.sendfile()，直接在两个文件之间复制数据，不用经过内存
```C++
public long transferTo(long position, long count, WritableByteChannel target) throws IOException {
    checkOpen();
    if (!target.isOpen()) {
        throw new ClosedChannelException();
    }
    checkReadable();
    if (target instanceof FileChannelImpl) {
        ((FileChannelImpl) target).checkWritable();
    }
    if (position < 0 || count < 0) {
        throw new IllegalArgumentException("position=" + position + " count=" + count);
    }

    if (count == 0 || position >= size()) {
        return 0;
    }
    count = Math.min(count, size() - position);

    // Try sendfile(2) first...
    boolean completed = false;
    if (target instanceof SocketChannelImpl) {
        FileDescriptor outFd = ((SocketChannelImpl) target).getFD();
        try {
            begin();
            try {
                MutableLong offset = new MutableLong(position);
                long rc = Libcore.os.sendfile(outFd, fd, offset, count);
                completed = true;
                return rc;
            } catch (ErrnoException errnoException) {
                // If the OS doesn't support what we asked for, we want to fall through and
                // try a different approach. If it does support it, but it failed, we're done.
                if (errnoException.errno != ENOSYS && errnoException.errno != EINVAL) {
                    throw errnoException.rethrowAsIOException();
                }
            }
        } finally {
            end(completed);
        }
    }
    // ...fall back to write(2).
    ByteBuffer buffer = null;
    try {
        buffer = map(MapMode.READ_ONLY, position, count);
        return target.write(buffer);
    } finally {
        NioUtils.freeDirectBuffer(buffer);
    }
}  
```

但是通过代码可以看到，这种直接在文件间读写的方式只支持向Socket写，否则，直接调用write()而已.

```C++
public long transferFrom(ReadableByteChannel src, long position, long count) throws IOException {
    checkOpen();
    if (!src.isOpen()) {
        throw new ClosedChannelException();
    }
    checkWritable();
    if (position < 0 || count < 0 || count > Integer.MAX_VALUE) {
        throw new IllegalArgumentException("position=" + position + " count=" + count);
    }
    if (position > size()) {
        return 0;
    }

    // Although sendfile(2) originally supported writing to a regular file.
    // In Linux 2.6 and later, it only supports writing to sockets.

    // If our source is a regular file, mmap(2) rather than reading.
    // Callers should only be using transferFrom for large transfers,
    // so the mmap(2) overhead isn't a concern.
    if (src instanceof FileChannel) {
        FileChannel fileSrc = (FileChannel) src;
        long size = fileSrc.size();
        long filePosition = fileSrc.position();
        count = Math.min(count, size - filePosition);
        ByteBuffer buffer = fileSrc.map(MapMode.READ_ONLY, filePosition, count);
        try {
            fileSrc.position(filePosition + count);
            return write(buffer, position);
        } finally {
            NioUtils.freeDirectBuffer(buffer);
        }
    }

    // For non-file channels, all we can do is read and write via userspace.
    ByteBuffer buffer = ByteBuffer.allocate((int) count);
    src.read(buffer);
    buffer.flip();
    return write(buffer, position);
}
```

transferFrom()的注释里写“Callers should only be using transferFrom for large transfers,so the mmap(2) overhead isn't a concern.”建议传大文件时才使用该方法，mmap过度使用会有什么问题呢？看到一些说法“you should allocate memory as larger as possible in each time. (avoid mutiple times of small mmap)”但是没有实践过mmap，没有啥感觉，先看下，以后碰到再体会。
