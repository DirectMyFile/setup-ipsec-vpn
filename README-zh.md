﻿# IPsec VPN 服务器一键安装脚本 &nbsp;[![Build Status](https://static.ls20.com/travis-ci/setup-ipsec-vpn.svg)](https://travis-ci.org/hwdsl2/setup-ipsec-vpn)

*其他语言版本: [English](README.md), [简体中文](README-zh.md).*

使用 Linux Shell 脚本一键快速搭建 IPsec VPN 服务器。同时支持 IPsec/L2TP 和 Cisco IPsec 协议，可用于 Ubuntu，Debian 和 CentOS 系统。你只需提供自己的 VPN 登录凭证，然后运行脚本自动完成安装。

我们将使用 <a href="https://libreswan.org/" target="_blank">Libreswan</a> 作为 IPsec 服务器，以及 <a href="https://github.com/xelerance/xl2tpd" target="_blank">xl2tpd</a> 作为 L2TP 提供者。

<a href="https://blog.ls20.com/ipsec-l2tp-vpn-auto-setup-for-ubuntu-12-04-on-amazon-ec2/" target="_blank">**&raquo; 相关教程： IPsec VPN Server Auto Setup with Libreswan**</a>

#### 目录

- [功能特性](#功能特性)
- [系统要求](#系统要求)
- [安装说明](#安装说明)
  - [Ubuntu & Debian](#ubuntu--debian)
  - [CentOS & RHEL](#centos--rhel)
- [下一步](#下一步)
- [重要提示](#重要提示)
- [升级Libreswan](#升级libreswan)
- [问题和反馈](#问题和反馈)
- [卸载说明](#卸载说明)
- [另见](#另见)
- [作者](#作者)
- [授权协议](#授权协议)

## 功能特性

- **新:** 增加支持更高效的 `IPsec/XAuth ("Cisco IPsec")` 模式
- **新:** 现在可以下载 VPN 服务器的预构建 [Docker 镜像](#另见)
- 全自动的 IPsec VPN 服务器配置，无需用户输入
- 封装所有的 VPN 流量在 UDP 协议，不需要 ESP 协议支持
- 可直接作为 Amazon EC2 实例创建时的用户数据使用
- 自动确定服务器的公网 IP 以及私有 IP 地址
- 包括基本的 IPTables 防火墙规则和 `sysctl.conf` 优化设置
- 测试通过： Ubuntu 16.04/14.04/12.04， Debian 8 和 CentOS 6/7

## 系统要求

一个新创建的 <a href="https://aws.amazon.com/ec2/" target="_blank">Amazon EC2</a> 实例，使用这些 AMI 之一：
- <a href="https://cloud-images.ubuntu.com/locator/" target="_blank">Ubuntu 16.04 (Xenial), 14.04 (Trusty) or 12.04 (Precise)</a>
- <a href="https://wiki.debian.org/Cloud/AmazonEC2Image" target="_blank">Debian 8 (Jessie) EC2 Images</a>
- <a href="https://aws.amazon.com/marketplace/pp/B00O7WM7QW" target="_blank">CentOS 7 (x86_64) with Updates</a>
- <a href="https://aws.amazon.com/marketplace/pp/B00NQAYLWO" target="_blank">CentOS 6 (x86_64) with Updates</a>

请参见 <a href="https://blog.ls20.com/ipsec-l2tp-vpn-auto-setup-for-ubuntu-12-04-on-amazon-ec2/#vpnsetup" target="_blank">详细步骤</a> 以及 <a href="https://aws.amazon.com/cn/ec2/pricing/" target="_blank">EC2 定价细节</a>。

**-或者-**

一个专用服务器或者虚拟专用服务器 (VPS)，全新安装以上操作系统之一。另外也可使用 Debian 7 (Wheezy)，但是必须首先运行<a href="extras/vpnsetup-debian-7-workaround.sh" target="_blank">另一个脚本</a>。 OpenVZ VPS 不受支持，用户可以尝试使用 <a href="https://github.com/breakwa11/shadowsocks-rss" target="_blank">ShadowsocksR</a> 或者 <a href="https://github.com/Nyr/openvpn-install" target="_blank">OpenVPN</a>。

这也包括各种云计算服务中的 Linux 虚拟机，比如 Google Compute Engine, Amazon EC2, Microsoft Azure, IBM SoftLayer, VMware vCloud Air, Rackspace, DigitalOcean, Vultr 和 Linode。

<a href="azure/README-zh.md" target="_blank">
    <img src="docs/images/azure-deploy-button.png" alt="Deploy to Azure" />
</a> <a href="http://dovpn.carlfriess.com/" target="_blank">
    <img src="docs/images/do-install-button.png" alt="Install on DigitalOcean" />
</a>

<a href="https://blog.ls20.com/ipsec-l2tp-vpn-auto-setup-for-ubuntu-12-04-on-amazon-ec2/#gettingavps" target="_blank">**&raquo; 我想建立并使用自己的 VPN ，但是没有可用的服务器**</a>

:warning: **不要** 在你的 PC 或者 Mac 上运行这些脚本！它们只能用在服务器上！

## 安装说明

### Ubuntu & Debian

首先，更新你的系统： 运行 `apt-get update && apt-get dist-upgrade` 并重启。这一步是可选的，但推荐。

要安装 VPN，请从以下选项中选择一个：

**选项 1:** 使用脚本随机生成的 VPN 登录凭证 （完成后会在屏幕上显示）：

```bash
wget https://git.io/vpnsetup -O vpnsetup.sh && sudo sh vpnsetup.sh
```

**选项 2:** 编辑脚本并提供你自己的 VPN 登录凭证：

```bash
wget https://git.io/vpnsetup -O vpnsetup.sh
nano -w vpnsetup.sh
[替换为你自己的值： YOUR_IPSEC_PSK, YOUR_USERNAME 和 YOUR_PASSWORD]
sudo sh vpnsetup.sh
```

**选项 3:** 将你自己的 VPN 登录凭证定义为环境变量：

```bash
# 所有变量值必须用 '单引号' 括起来
# *不要* 在值中使用这些字符：  \ " '
wget https://git.io/vpnsetup -O vpnsetup.sh && sudo \
VPN_IPSEC_PSK='你的IPsec预共享密钥' \
VPN_USER='你的VPN用户名' \
VPN_PASSWORD='你的VPN密码' sh vpnsetup.sh
```

Linode 用户可以使用 <a href="https://www.linode.com/stackscripts/view/37239" target="_blank">这个 StackScript</a> 进行自动部署 （<a href="https://www.linode.com/docs/platform/stackscripts" target="_blank">参见教程</a>）。

如需在 DigitalOcean 上安装，可以参考这个<a href="https://usefulpcguide.com/17318/create-your-own-vpn/" target="_blank">分步指南</a>，由 Tony Tran 编写。

**注：** 如果无法通过 `wget` 下载，你也可以打开 <a href="vpnsetup.sh" target="_blank">vpnsetup.sh</a> (或者 <a href="vpnsetup_centos.sh" target="_blank">vpnsetup_centos.sh</a>)，然后点击右方的 **`Raw`** 按钮。按快捷键 `Ctrl-A` 全选， `Ctrl-C` 复制，然后粘贴到你喜欢的编辑器。

### CentOS & RHEL

首先，更新你的系统： 运行 `yum update` 并重启。这一步是可选的，但推荐。

按照与上面相同的步骤，但是将 `https://git.io/vpnsetup` 换成 `https://git.io/vpnsetup-centos`。

## 下一步

配置你的计算机或其它设备使用 VPN 。请参见：

<a href="docs/clients-zh.md" target="_blank">配置 IPsec/L2TP VPN 客户端</a>   
<a href="docs/clients-xauth-zh.md" target="_blank">配置 IPsec/XAuth ("Cisco IPsec") VPN 客户端</a>

开始使用自己的专属 VPN ! :sparkles::tada::rocket::sparkles:

## 重要提示

**Windows 用户** 在首次连接之前需要<a href="docs/clients-zh.md#regkey" target="_blank">修改一次注册表</a>，以解决 VPN 服务器 和/或 客户端与 NAT （比如家用路由器）的兼容问题。如果在连接过程中遇到错误，请参见 <a href="docs/clients-zh.md#故障排除" target="_blank">故障排除</a>。

**Android 6 (Marshmallow) 用户** 请参考此文档中的注释： <a href="docs/clients-zh.md#android" target="_blank">配置 IPsec/L2TP VPN 客户端</a>。

如果需要添加，修改或者删除 VPN 用户账户，请参见 <a href="docs/manage-users-zh.md" target="_blank">管理 VPN 用户</a>。

在 VPN 已连接时，客户端配置为使用 <a href="https://developers.google.com/speed/public-dns/" target="_blank">Google Public DNS</a>。如果偏好其它的域名解析服务，请编辑 `/etc/ppp/options.xl2tpd` 和 `/etc/ipsec.conf` 并替换 `8.8.8.8` 和 `8.8.4.4`。然后重启服务器。

对于有外部防火墙的服务器（比如 <a href="https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-network-security.html" target="_blank">EC2</a>/<a href="https://cloud.google.com/compute/docs/networking#firewalls" target="_blank">GCE</a>），请打开 UDP 端口 500 和 4500，以及 TCP 端口 22 （用于 SSH）。

如果需要打开服务器上的其它端口，请编辑 `/etc/iptables.rules` 和/或 `/etc/iptables/rules.v4` (Ubuntu/Debian)，或者 `/etc/sysconfig/iptables` (CentOS)。然后重启服务器。

在使用 `IPsec/L2TP` 连接时，VPN 服务器在虚拟网络 `192.168.42.0/24` 内具有 IP `192.168.42.1`。

这些脚本在更改现有的配置文件之前会先做备份，使用 `.old-日期-时间` 为文件名后缀。

## 升级Libreswan

提供两个额外的脚本 <a href="extras/vpnupgrade.sh" target="_blank">vpnupgrade.sh</a> 和 <a href="extras/vpnupgrade_centos.sh" target="_blank">vpnupgrade_centos.sh</a>，可用于升级 Libreswan （<a href="https://libreswan.org" target="_blank">网站</a> | <a href="https://lists.libreswan.org/mailman/listinfo/swan-announce" target="_blank">通知列表</a>）。请在运行前根据需要修改 `swan_ver` 变量。检查已安装版本： `ipsec --version`

## 问题和反馈

- 有问题需要提问？请先搜索已有的留言，在<a href="https://gist.github.com/hwdsl2/9030462#comments" target="_blank">这个 Gist</a> 以及<a href="https://blog.ls20.com/ipsec-l2tp-vpn-auto-setup-for-ubuntu-12-04-on-amazon-ec2/#disqus_thread" target="_blank">我的博客</a>。
- Libreswan (IPsec) 的相关问题可在<a href="https://lists.libreswan.org/mailman/listinfo/swan" target="_blank">邮件列表</a>提问。也可以参见这些文章：<a href="https://libreswan.org/wiki/Main_Page" target="_blank">[1]</a> <a href="https://wiki.gentoo.org/wiki/IPsec_L2TP_VPN_server" target="_blank">[2]</a> <a href="https://wiki.archlinux.org/index.php/L2TP/IPsec_VPN_client_setup" target="_blank">[3]</a> <a href="https://help.ubuntu.com/community/L2TPServer" target="_blank">[4]</a> <a href="https://libreswan.org/man/ipsec.conf.5.html" target="_blank">[5]</a>。
- 如果你发现了一个可重复的程序漏洞，请提交一个 <a href="https://github.com/hwdsl2/setup-ipsec-vpn/issues?q=is%3Aissue" target="_blank">GitHub Issue</a>。

## 卸载说明

请参见 <a href="docs/uninstall-zh.md" target="_blank">卸载 VPN</a>。

## 另见

- <a href="https://github.com/hwdsl2/docker-ipsec-vpn-server" target="_blank">IPsec VPN Server on Docker</a>
- <a href="https://github.com/jlund/streisand" target="_blank">Streisand</a>
- <a href="https://github.com/SoftEtherVPN/SoftEtherVPN" target="_blank">SoftEther VPN</a>
- <a href="https://github.com/breakwa11/shadowsocks-rss" target="_blank">ShadowsocksR</a>
- <a href="https://github.com/Nyr/openvpn-install" target="_blank">OpenVPN Install</a>
- <a href="https://github.com/ftao/vpn-deploy-playbook" target="_blank">VPN Deploy Playbook</a>
- <a href="https://github.com/sockeye44/instavpn" target="_blank">Insta VPN</a>
- <a href="https://github.com/quericy/one-key-ikev2-vpn" target="_blank">One Key IKEv2 VPN</a>

## 作者

**Lin Song** (linsongui@gmail.com)   
- 最后一年的美国在读博士生，专业是电子与计算机工程 (ECE)
- 现在正在积极寻找新的工作机会，比如软件或系统工程师
- 在 LinkedIn 上与我联系： <a href="https://www.linkedin.com/in/linsongui" target="_blank">https://www.linkedin.com/in/linsongui</a>

感谢本项目所有的 <a href="https://github.com/hwdsl2/setup-ipsec-vpn/graphs/contributors" target="_blank">贡献者</a>！

## 授权协议

版权所有 (C) 2014-2016&nbsp;Lin Song&nbsp;&nbsp;&nbsp;<a href="https://www.linkedin.com/in/linsongui" target="_blank"><img src="https://static.licdn.com/scds/common/u/img/webpromo/btn_viewmy_160x25.png" width="160" height="25" border="0" alt="View my profile on LinkedIn"></a>   
基于 <a href="https://github.com/sarfata/voodooprivacy" target="_blank">Thomas Sarlandie 的工作</a> (版权所有 2012)

这个项目是以 <a href="http://creativecommons.org/licenses/by-sa/3.0/" target="_blank">知识共享署名-相同方式共享3.0</a> 许可协议授权。   
必须署名： 请包括我的名字在任何衍生产品，并且让我知道你是如何改善它的！
