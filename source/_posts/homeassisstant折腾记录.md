---
title: home assisstant 折腾记录
date: 2021-04-28 15:00:00
tags: ["HomeAssisstant", "hassos"]
---

# 前言

捡了一个小米多模网关 3，入了 home assisstant 的坑，记录下折腾的记录。

![homeassisstant](/images/20210428153314.png)

<!--more-->

# hassos 安装

## 下载

由于手头有一个装了 pve 的蜗牛星际跑着黑裙，就决定直接使用 hassos 了。在[Github](https://github.com/home-assistant/operating-system/releases/latest)上下载 `Open Virtual Appliance` 类型的 `hassos_ova-x.xx.qcow2.xz`，解压上传到 pve 里。

## 创建

创建一个虚拟机，BIOS 选择`UEFI`，这里将`VMID`定义为了`200`，回到 pve 的 shell，执行 qm importdisk 命令将之前下载解压之后的 qcow2 文件导入到 VMID 为 200 的虚拟机里，存储在 local-lvm。
命令如下：

```bash
qm importdisk 200 hassos_ova-5.13.qcow2 local-lvm
```

回到 200 节点，将导入的硬盘添加上，并在 200 节点的选项——引导顺序项中，将第一引导顺序设置为此硬盘即可开机。

![硬件](/images/20210428151640.png)
![选项](/images/20210428151705.png)

# HACS

## 安装

根据[官方文档](https://hacs.xyz/docs/installation/installation/#home-assistant-os)，在 Supervisor 面板中安装并启动 SSH 插件，执行安装命令即可。

```bash
wget -q -O - https://install.hacs.xyz | bash -
```

## 运行

成功安装后，在 homeassisstan 的配置-集成中添加 HACS 集成，如果没有 HACS，尝试清除缓存（可以使用 Ctrl+F5 刷新）或在配置-服务配置中重启。

# 小米多模网关

进入 HACS，点击右下角的添加存储库，搜索`Xiaomi Gateway 3`并安装即可，安装之后同样需要在 homeassisstan 的配置-集成中添加 Xiaomi Gateway 3。

## 禁止升级

在设备实体页面，打开`Xiaomi Gateway 3 Firmware Lock`的开关

# 参考

-   [1] [hacs installation](https://hacs.xyz/docs/installation/installation/#home-assistant-os)

-   [2] [xiaomigateway3 installation](https://github.com/AlexxIT/XiaomiGateway3#install)
