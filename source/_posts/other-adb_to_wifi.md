---
title: ADB控制安卓WIFI连接（斐讯R1联网指南）
date: 2019-04-16 08:40:00
categories:
- Misc

tags: 
- Phicomm
- Android
- ADB
---

许多情况下，我们所调试的安卓设备并没有屏幕显示，或者阉割掉了系统设置模块，比如斐讯R1智能音箱。

这时候，使用adb的wifi控制就显得尤为重要，基于在Github的[adb-join-wifi](https://github.com/steinwurf/adb-join-wifi)项目，我们增加了802.1x的PEAP加密协议支持，并且引入了静态ip地址，以及删除网络配置等功能，修改后的项目地址为https://github.com/laddie132/adb-join-wifi
<!-- more -->

## 使用方法
首先，需要安装app，你可以手动编译该项目，也可以直接下载下面的安装包：
```
链接: https://pan.baidu.com/s/1F91iayP_0jRxky2Z5mNlww 提取码: ivgd
```

此外，还需要安装adb环境，执行以下几个命令可以完成不同的操作：

1. 连接到无密码WIFI
``` shell
adb shell am start -n com.steinwurf.adbjoinwifi/.MainActivity
    --esn connect -e ssid SSID
```

2. 连接到WEP/WPA加密WIFI
``` shell
adb shell am start -n com.steinwurf.adbjoinwifi/.MainActivity \
    --esn connect -e ssid SSID -e password_type WEP|WPA -e password PASSWORD
```

3. 连接到802.1x加密WIFI
``` shell
adb shell am start -n com.steinwurf.adbjoinwifi/.MainActivity \
    --esn connect -e ssid SSID -e password_type PEAP -e username USERNAME -e password PASSWORD
```

4. 连接到WIFI并且设置固定ip地址
``` shell
adb shell am start -n com.steinwurf.adbjoinwifi/.MainActivity \
    --esn connect -e ssid SSID -e password_type WEP|WPA|PEAP [-e username USERNAME] -e password PASSWORD \
    -e ip IP -e gateway GATEWAY --ei prefix PREFIX -e dns1 DNS1 -e dns2 DNS2
```

5. 删除某个WIFI配置
``` shell
adb shell am start -n com.steinwurf.adbjoinwifi/.MainActivity \
    --esn remove -e ssid SSID
```

6. 删除所有WIFI配置
``` shell
adb shell am start -n com.steinwurf.adbjoinwifi/.MainActivity \
    --esn remove
```

## 斐讯R1
有了上述的方法，我们可以实现无需`斐讯AI`软件，从而连接wifi的功能。

经测试，斐讯R1音箱长按功能键后开启网络配置功能，实际上是打开了一个没有密码wifi热点。因此，我们可以利用该热点，为斐讯R1安装上述app，并使用adb命令连接某个wifi，并且设置固定ip地址，便于后续使用。

> 不知道`斐讯AI`软件为斐讯R1联网是不是也是上述方法=_=

## 致谢
感谢[adb-join-wifi](https://github.com/steinwurf/adb-join-wifi)项目的开发者。