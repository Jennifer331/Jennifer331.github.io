---
layout: post
title: "adb的usb和tcpip连接方式"
lang: zh
tags: [Android，OTG]
---
# <font color="#e3796b">问题咋来的呢</font>
解一个OTG的bug，第一次接触这个概念，[USB On-The-Go](https://zh.wikipedia.org/wiki/USB_On-The-Go) 通常缩写为USB OTG，是USB2.0规格的补充标准。它可使USB设备，例如播放器或手机，从USB周边设备变为USB主机，与其他USB设备连接通信。在正常情况下，这些支持OTG的USB设备和USB主机（如台式机或者手提电脑），仍然作为USB周边设备使用。  
Android Developer关于这方面的文档在[Develop->API Guides->Usb Host and Accessory](http://developer.android.com/guide/topics/connectivity/usb/index.html)目录下  
这下又牵引出一些以前我没有接触过的概念

# <font color="#e3796b">Usb被占用，咋Debug呢</font>
可以使用tcpip通道进行调试，__adb usb__ 和 __adb tcpip__ 可以在两种adb传输通道间切换。

* Connect the Android-powered device via USB to your computer.

* From your SDK platform-tools/ directory, enter __adb tcpip 5555__ at the command prompt.

* Enter __adb connect <device-ip-address>:5555__ You should now be connected to the Android-powered device and can issue the usual adb commands like __adb logcat__.

* To set your device to listen on USB, enter __adb usb__ .

# <font color="#e3796b">Android SDCard vs USB Storage</font>
待了解

# <font color="#e3796b">MTP,PTP,USB Mass Storage</font>
[Android USB Connections Explained: MTP, PTP, and USB Mass Storage](http://www.howtogeek.com/192732/android-usb-connections-explained-mtp-ptp-and-usb-mass-storage/)摘录:

* Modern Android Devices Don’t Support __USB Mass Storage__ (因为当它被挂载到电脑以后，手机就访问不到了)

* __MTP(Media Transfer Protocal)__ This protocol works very differently from USB mass storage. Rather than exposing your Android device’s raw file system to Windows, MTP operates at the file level. Your Android device doesn’t expose its entire storage device to Windows. Instead, when you connect a device to your computer, the computer queries the device and the device responds with a list of files and directories it offers. The computer can download a file — it will request the file from the device, and the device will send the file over the connection. If a computer wants to upload a file, it sends the file to the device and the device chooses to save it. When you delete a file, your computer sends a signal to the device saying, “please delete this file,” and the device can delete it.

* MTP is actually based on PTP, but adds more features, or “extensions.”

* Apple’s Mac OS X does support PTP, so you can use PTP mode to transfer photos from an Android device to a Mac over a USB connection without any special software.
