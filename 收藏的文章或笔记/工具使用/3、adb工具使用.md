## 一、概念和一些参考链接
https://blog.csdn.net/qq_64079631/article/details/129652699
https://blog.csdn.net/x2584179909/article/details/108319973
https://juejin.cn/s/华为彻底删除谷歌服务框架

adb是Android Debug Bridge的缩写，**是一种用于与Android设备通信的命令行工具。它可以通过USB连接或Wi-Fi连接，允许开发者在计算机和Android设备之间进行文件传输、安装应用程序、调试应用程序等操作**。要使用adb，需要先将Android设备与计算机连接，并确保设备允许USB调试模式。然后，在计算机的命令行界面上输入adb命令，后面加上相应的参数即可执行各种操作。例如，adb devices 命令可以列出已连接的Android设备，adb install 命令可以将一个apk文件安装到Android设备上。可以通过 adb --help 命令来获取更多的adb命令及其用法。
 
## 二、安装和使用
1. 在你的电脑上配置：[https://developer.android.com/studio/releases/platform-tools](https://developer.android.com/studio/releases/platform-tools)
2. 打开手机的开发者模式，并允许使用usb进行调试
3. 相关命令
```bash
获取当前启动app的包名和启动名
adb shell dumpsys window | findstr mFocusedApp 
 
卸载软件
adb uninstall <软件名>
adb uninstall -k <软件名>
如果加 -k参数，  为卸载软件 但是保留配置和缓存文件
开启adb服务
adb start-server
关闭adb服务，杀掉进程
adb kill-server
连接设备
adb connect 设备ip（如：192.168.1.61）
如果是USB连接，直接会连接ADB，如果是想通过网络连接（有线或者无线）,则需要在同一个局域网，通过IP连接。上面192.168.1.61替换成想要连接设备的IP即可
断开设备
adb disconnect 设备ip（如：192.168.1.61）
清除应用数据与缓存
adb shell pm clear （apk包名）
获取文件的读写权限
adb remount
有些设备并不能直接adb remount，必须要先以root身份进入，先执行adb root，在执行adb remount
查询已安装包名列表
adb shell pm list package
对com.xx.mm包使用monkey命令
adb shell monkey -p com.xx.mm --throttle 200 50000
查找monkey进程信息
adb shell ps | find "monkey"
杀掉monkey进程，例子中的数字是monkey的PID进程号
adb shell kill 23770
重启手机
adb shell reboot
打开svc帮助界面
adb shell svc
查询wifi操作帮助
adb shell svc wifi
关闭wifi
adb shell svc wifi disable
打开wifi
adb shell svc wifi enable
擦除data，即恢复出厂设置
adb shell wipe data
指定查询"mF"的activity信息
adb shell dumpsys activity | find "mF"
启动指定activity
adb shell am start -n com.android.browser/.BrowserActivyty
查看am命令的帮助信息
adb shell am
清空logcat日志
adb logcat -c 
查看bug报告
adb bugreport
获取设备的ID和序列号
adb get-serialno
```


