---
title: RouterOS 7.12 启用NAT1
date: 2023-11-20 08:06:52
tags: ["RouterOS", "ROS", "NAT1", "FullCone"]
---

备忘防丢。

<!--more-->

## 命令

网上搜了一大堆，加完策略依旧是都是 PortRestrictedCone，找到一个规则里多了个`randomise-ports=yes`的，勾上之后我也是 FullCone 啦

```bash
/ip firewall nat add action=endpoint-independent-nat chain=srcnat protocol=udp randomise-ports=yes place-before=0 comment=FullCone_Nat
/ip firewall nat add action=endpoint-independent-nat chain=dstnat protocol=udp randomise-ports=yes place-before=0
```

![FullCone](/images/Snipaste_2023-11-20_09-17-12.png)

## 参考

-   [1] [MikroTik 开启 Endpoint-IndependentNAT](https://www.rosnas.com/3062.html)
