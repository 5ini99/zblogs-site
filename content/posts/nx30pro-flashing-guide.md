+++
date = '2024-11-18T16:01:51+08:00'
draft = false
title = '华三Nx30pro刷机指南'
featured_image = "/images/thumbnails/nx30pro-1.jpg"
summary = "华三H3C NX30Pro这台路由器刷机是非常简单的，Arm A53 双核 1.3Ghz，256M 的内存，也都是内置功放 Wi-Fi 信号没差，而且刷机也很方便，只需要学会几个常用软件和命令的使用即可，不用掌握什么复杂的技术。"
tags = ["NX30Pro", "MT7981", "MT7981B"]
+++

## 登录路由器

NX30 Pro 默认开启了 telnet，默认的地址是 192.168.124.1，端口是99，用户名是 H3C，密码就你设置的路由器后台密码。

Mac可通过 brew install telnet 安装telnet

安装 telnet 后连接：

> telnet 192.168.124.1 99

输入用户名和密码，进入系统后安装SSH：

```
curl -o /tmp/dropbear.ipk https://downloads.openwrt.org/releases/packages-19.07/aarch64_cortex-a53/base/dropbear_2019.78-2_aarch64_cortex-a53.ipk
opkg install /tmp/dropbear.ipk
/etc/init.d/dropbear enable
/etc/init.d/dropbear start
```

![nx30pro-2.png](/images/thumbnails/nx30pro-2.png)

## 备份原系统

将系统备份到 tmp路径下，执行命令，可能要等几分钟

> dd if=/dev/mtd5 of=/tmp/backup.img

![nx30pro-3.png](/images/thumbnails/nx30pro-3.png)

将备份镜像下载到本地，如果后期刷回官方需要用到。

windows 可以使用 winscp，或 filezilla 等软件scp连接将backup.img下载到本地，我这边直接使用scp命令将备份下载到本地。

![nx30pro-4.png](/images/thumbnails/nx30pro-4.png)

## 刷入uboot

1. 方法一：使用uboot文件

将uboot.bin文件存入路由器/tmp文件夹中

> scp -O -o HostKeyAlgorithms=+ssh-rsa -o PubkeyAcceptedKeyTypes=+ssh-rsa ~/Downloads/uboot.bin H3C@192.168.124.1:/tmp/

![nx30pro-5.png](/images/thumbnails/nx30pro-5.png)

![nx30pro-6.png](/images/thumbnails/nx30pro-6.png)

确保uboot文件已放入服务器该文件夹下，然后执行以下命令，如图所示完成了写入 uboot。（请确保MD5 校验结果和图中相同，再敲回车执行，否则会变砖）

```
cd /tmp
md5sum uboot.bin
mtd write /tmp/uboot.bin FIP
```

![nx30pro-7.png](/images/thumbnails/nx30pro-7.png)

2. 方法二：网络下载uboot

在终端直接复制执行，就能完成 uboot 的写入。（请确保MD5 校验结果和图中相同，再敲回车执行，否则会变砖）

```
cd /tmp
curl -L https://share.qust.me/d/%E8%B7%AF%E7%94%B1%E5%99%A8/NX30Pro/uboot.bin -o uboot.bin
md5sum uboot.bin
mtd write /tmp/uboot.bin FIP
```

3. 上传openwrt

路由器断电后，先按住背后 Reset 恢复按钮不放，再插电，等待 10s 左右松开背后 Reset，路由器就进入了 uboot，电脑用网线连接路由器 LAN1，并设置好静态 IP：IP地址填 192.168.1.2，子网掩码 255.255.255.0，网关 192.168.1.1，DNS 192.168.1.1。

![nx30pro-8.png](/images/thumbnails/nx30pro-8.png)
![nx30pro-9.png](/images/thumbnails/nx30pro-9.png)

浏览器打开 192.168.1.1 就能打开 uboot 后台。

![nx30pro-10.png](/images/thumbnails/nx30pro-10.png)

4. 刷回原版

刷回官方非常简单，进入 uboot 后选择之前备份的 backup.img 文件更新即可，系统就会重启进入官方的系统。