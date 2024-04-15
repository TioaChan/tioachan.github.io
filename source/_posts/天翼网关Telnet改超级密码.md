---
title: 天翼网关Telnet改超级密码
date: 2023-11-19 22:06:52
tags: ["天翼网关", "光猫", "超级密码"]
---

备忘防丢。

<!--more-->

## 命令

进入 telnet 后，执行如下三条命令修改：

```bash
sendcmd 1 DB set DevAuthInfo 0 Pass nE7jA%5m
sendcmd 1 DB save
sendcmd 1 DB saveasy
```

## 参考

-   [1] [【搬运总结】中兴 F650 天翼网关 3.0 免拆机 免 TTL 持久开 telnet-光猫/adsl/cable 无线一体机-恩山无线论坛](https://www.right.com.cn/forum/thread-3642668-1-1.html)
