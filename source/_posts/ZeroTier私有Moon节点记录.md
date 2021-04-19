---
title: ZeroTier 私有Moon节点记录
date: 2021-01-18 16:40:53
tags: ["ZeroTier", "Moon"]
---

未使用 Moon 私有节点时延迟基本 200+，使用 Moon 节点后延迟低至两位数，很爽有没有。

![](/images/20210118181128.png)

<!--more-->

## 部署 Moon 节点

### 安装 ZeroTier

```bash
curl -s https://install.zerotier.com/ | sudo bash
```

### 加入 network

```bash
zerotier-cli join <network id>
```

### 生成并修改 moon.json 文件

```bash
cd /var/lib/zerotier-one
zerotier-idtool initmoon identity.public > moon.json
vi moon.json
```

修改"stableEndpoints"为公网的 IP,例如

```
"stableEndpoints": ["8.8.8.8/9993"]
```

### 生成签名文件

```bash
zerotier-idtool genmoon moon.json
```

执行之后会生产一个 `000000xxxx.moon` 的文件，将这个文件复制出来。

## 加入 Moon 节点

在 ZeroTier 安装目录新建一个名为`moons.d`的文件夹，将上步下载下来的`000000xxxx.moon`文件复制至该目录下，重启 ZeroTier 即可。

不同系统的安装路径如下：

| 系统            | 路径                                      |
| --------------- | ----------------------------------------- |
| Windows         | C:/ProgramData/ZeroTier/One               |
| Macintosh       | /Library/Application Support/ZeroTier/One |
| Linux           | /var/lib/zerotier-one                     |
| FreeBSD/OpenBSD | /var/db/zerotier-one                      |

### OpenWrt

特别的，`OpenWrt`需要额外操作：编辑`zerotier`文件，在`add_join() {}`节上面插入如下代码：

```bash
vi /etc/init.d/zerotier
```

```bash
mkdir -p $CONFIG_PATH/moons.d
cp /root/moons.d/* $CONFIG_PATH/moons.d/
```

![](/images/20210118175003.png)

然后将`000000xxxx.moon`文件复制至`/root/moons.d/`目录下。

![](/images/20210118175425.png)

最后重启 zerotier。

```bash
/etc/init.d/zerotier restart
```

### ？

Windows 下复制文件重启软件并没有成功加入 Moon 节点，使用下面的命令加入了 Moon。

```bash
zerotier-cli orbit <world ID> <seed>
```

-   world ID：生成的`moon.json`文件中`id`的值
-   seed：Moon 节点的公网 ip 及端口

完整的命令如下:

```bash
zerotier-cli orbit XXXXXXXXXX 8.8.8.8/9993
```

## 验证

在其他设备输入下面的命令列出 zerotier 的 peer 列表，当列表中含有 Moon 节点的 IP 的记录的角色为`MOON`时，该设备 zerotier 成功加入私有 Moon 节点。

```bash
zerotier-cli listpeers
```

![](/images/20210118180116.png)

## 测试

未使用 Moon 私有节点时延迟基本 200+，使用 Moon 节点后延迟低至两位数，很爽有没有。

![](/images/20210118181128.png)

## 流量转发

在跳板机上对 iptables 进行如下配置

```bash
iptables -A FORWARD -d 192.168.3.0/24 -j ACCEPT
iptables -t nat -A POSTROUTING -s 192.168.3.0/24 -j MASQUERADE
iptables -t nat -A POSTROUTING -d 192.168.3.0/24 -j MASQUERADE
iptables -A FORWARD -s 172.22.22.0/24 -j ACCEPT
iptables -t nat -I POSTROUTING -o xxxxxx -j MASQUERADE
iptables -I FORWARD -i xxxxxx -j ACCEPT
iptables -I FORWARD -o xxxxxx -j ACCEPT
iptables-save
```

---

将 192.168.3.0/24 替换为家庭网络网段，

将 172.22.22.0/24 替换为 ZeroTier 中配置的网段地址

将 xxxxxx 替换为 zerotier-cli listnetworks 中的网卡名称

---

## 参考

-   [1] [远程办公：ZeroTier 异地组网及私有 Moon 转发节点搭建*办公软件*什么值得买](https://post.smzdm.com/p/adwrepgk/)

-   [2] [ZeroTier 局域网互通 - 樱梦](https://233.sh/2020/03/11/zerotier-network-bridge/)
