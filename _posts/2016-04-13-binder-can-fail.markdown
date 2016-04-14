---
layout: post
title: "Binder也会挂"
lang: zh
tags: [Android File]
---
# <font color="#e3796v">问题咋来的呢</font>
自从学会了打断点这件事情，尤其是Android Studio强大到动动手指点击一下，连bitmap现在的状态都能直接绘制出来给你看！！！让我怎能不爱不释手。。。然而，有两个场景，真的让人很伤心。。。一是OTG的bug，手机“后屁股眼”（这个词是站柜实习的时候从东北佬那里学来的，哈哈，好贴切~）被占着，二是测试告诉你“复现不出来。。。”只能在log海里游啊游啊游啊游。。。

# <font color="#e3796v">JavaBinder: !!! FAILED BINDER TRANSACTION !!!</font>
<img alt="Faider Binder Transaction" src="{{site.baseurl}}/assets/images/binder-can-fail.png" width="50%" align="left" style="margin: 0px 15px">
看到这句log，我还以为是我们自己sdk里输出的，因为一堆堆感叹号怎么看都有点土土的感觉。。。然而在找不到其他入口的情况下，尝试着谷歌了一下。。。竟然，有答案。。。我的理解是，Activity转换中间的Binder挂掉了，导致它携带的数据异常。网上大部分这类的问题都是因为用Intent传送了很大的bitmap，但是我这里只是让它带了个小小的String而已，啊！只是个String啊！但是观察log发现，这时候不光我这个app的进程中的Binder在挂，其他的也在挂，哈哈，有可能底层有什么问题，测试胡乱操作又刚好玩出来了。

# <font color="#e3796v">TransactionTooLargeException</font>
关于在Activity间直接通过Intent传送大量数据会出现的异常，Android Developer 中[TransactionTooLargeException](http://developer.android.com/reference/android/os/TransactionTooLargeException.html)有详细阐述。一点摘要：

* The Binder transaction buffer 有一个固定大小, currently 1Mb, which is shared by all transactions in progress for the process.

* when a remote procedure call throws TransactionTooLargeException，可能会有两种后果 . 1.the client was unable to send its request to the service (most likely if the arguments were too large to fit in the transaction buffer), 2.the service was unable to send its response back to the client (most likely if the return value was too large to fit in the transaction buffer).但我们并不能确定到底会发生哪种后果。所以The client 要对可能失败的情况做处理。

* If you are implementing a service, it may help to 限制大小和复杂度 on the queries that clients can perform. For example, if the result set could become large, then don't allow the client to request more than a few records at a time. Alternately, 不一次性返回所有的数据, 只返回重要的信息，剩下的让client依情况再请求.
