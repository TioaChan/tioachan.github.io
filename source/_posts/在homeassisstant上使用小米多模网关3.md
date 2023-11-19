---
title: 在home assisstant上使用小米多模网关3
date: 2021-08-03 20:54:00
tags: ["HomeAssisstant","小米多模网关3","ZNDMWG03LM"]
---

# 前言

简单记录一下在home assisstant上使用小米多模网关3的过程。

![homeassisstant](/images/20210803205643.png)

<!--more-->

# 准备

## 确认固件版本

首先需要确认手中网关的固件版本，正常接入米家App后在设备页固件升级中查看版本号，接着确认支持接入home assisstant的固件版本号，可以在[Github](https://github.com/AlexxIT/XiaomiGateway3#supported-firmwares)中查看。

本人购买此网关时初始版本号为`v1.4.5_0016`，后续通过telnet升级至`v1.5.0_0026`，升级过程见后文。
![固件版本](/images/Screenshot_2021-08-03-21-04-51-803_com.xiaomi.sma.jpg)

拆机降级不在此文介绍。

## 通过HACS安装Xiaomi Gateway 3集成

打开HACS，点击浏览并添加集成按钮，键入`Xiaomi Gateway 3`搜索并安装集成即可。

需要注意的是，发文时Xiaomi Gateway 3 v1.3.0 版本存在问题，所以安装时选择上一个版本，即v1.2.3。

安装完毕后重启 Home Assistant 服务，HACS的安装方式请查看本站`HomeAssisstant`tag下的第一篇文章。


# 接入

## 登录账号

服务重启完毕后，在配置-集成中添加`Xiaomi Gateway 3`集成，如果搜不到此集成请`Ctrl+F5`刷新页面。

首次使用`Xiaomi Gateway 3`集成需要添加米家账号。在上一步选择`Xiaomi Gateway 3`集成后在接下来的`选择动作`弹窗里选择`Add Mi Cloud Account`，输入小米账号密码、选择服务器后提交。

## 接入

再次重复添加`Xiaomi Gateway 3`集成，本次就能在`选择动作`弹窗里看到需要接入的网关了，选择此项，完成接入。

如果在接入过程中出现问题，可以尝试更换开启telnet的命令，可以在[Github Gist](https://gist.github.com/zvldz/1bd6b21539f84339c218f9427e022709)中查看。

![Xiaomi Gateway 3](/images/20210803212649.png)


## 禁止自动升级

回到管理集成页面，点击刚才添加的网关集成卡片，通过卡片中的`X个设备`进入设备页，选择网关设备，打开`Firmware Lock`的开关。

![禁止自动升级](/images/20210803205643.png)

# 通过telnet升级

请查看[Github](https://github.com/zvldz/mgl03_fw/tree/main/firmware#the-easy-way)

# 参考

-   [1] [xiaomigateway3 installation](https://github.com/AlexxIT/XiaomiGateway3#install)
