---
title: LEDE的ipset加入IPV6集合
date: 2022-09-08 12:02:52
tags: ["ipset"]
---

<!--more-->

## 地址库

```
git clone -b ip-lists https://github.com/gaoyifan/china-operator-ip
```

## 脚本

```shell
ipset -N $1 hash:net family inet6 2>/dev/null
echo "create $1 hash:net family inet6 hashsize 1024 maxelem 65536" > /tmp/mwan3.ipset
cat $2 | sed -e "s/^/add $1 /" >> /tmp/mwan3.ipset
ipset -! flush $1
ipset -! restore < /tmp/mwan3.ipset 2>/dev/null
rm -f /tmp/mwan3.ipset
```

## 使用

将上面的脚本文件保存，这里命名为了`genip6set.sh`.

```shell
/etc/mwan3helper/genipset.sh newct '/root/china-operator-ip/chinanet.txt'
/root/china-operator-ip/genip6set.sh newct6 '/root/china-operator-ip/chinanet6.txt'
/etc/mwan3helper/genipset.sh newcu '/root/china-operator-ip/unicom.txt'
/root/china-operator-ip/genip6set.sh newcu6 '/root/china-operator-ip/unicom6.txt'
```

## 验证

按上面定义的名字使用`ipset list [SETNAME]`进行验证，如果打印出ip地址则为成功。

```shell
ipset list newcu6
```

## 注意

ipset创建ipv6集合需要特别指定family为inet6
```
ipset -N $1 hash:net
ipset -N $1 hash:net family inet6
```