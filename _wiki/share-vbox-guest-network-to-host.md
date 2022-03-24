---
layout: wiki
title: 共享虚拟机 vpn 网络
cate1: tools
cate2: vpn
description: 共享虚拟机 vpn 网络到宿主机
keywords: vpn, virtual box
---

## 使用场景

Virtual Box 虚拟机（Windows 7）内安装公司需要的监控软件和 VPN，需要将网络共享到宿主机（Windows 10）供宿主机使用。

## 配置

主要思路是网卡1桥接网络用于虚拟机通过桥接模式访问外网，网卡2采用仅主机（Host-Only）网络用于共享虚拟机内 VPN 到宿主机，宿主机通过增加公司网段路由经过 VPN 访问公司网段。

设置桥接网络网卡1：

![设置桥接网络网卡1](/images/wiki/share-vbox-guest-network-to-host-netcard1.png)

设置 host only 网卡2：

![设置 host only 网卡2](/images/wiki/share-vbox-guest-network-to-host-netcard2.png)

在虚拟机中连接 VPN 后，共享 VPN 网卡网络到 host only 网卡2上，并为 host only 网卡2指派固定 IP：

![共享 VPN 网卡网络](/images/wiki/share-vbox-guest-network-to-host-guest-config.png)

![分配网卡2网络IP](/images/wiki/share-vbox-guest-network-to-host-set-guest-netcard2-ip.png)

设置宿主机 host only 网卡的 IP，IP 地址需要与虚拟机中网卡2在同一网段：

![设置宿主机 host only 网卡的 IP](/images/wiki/share-vbox-guest-network-to-host-set-host-netcard2-ip.png)

最后在宿主机增加路由：

```txt
route add 172.0.0.0 mask 255.0.0.0 192.168.137.1
```

如果需要删除路由：

```txt
route delete 172.0.0.0 mask 255.0.0.0 192.168.137.1
```

好了，现在可以在宿主机访问 172.x.x.x 网段。
