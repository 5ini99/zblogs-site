+++
date = '2024-11-18T11:55:53+08:00'
draft = false
title = '思创CT3003刷机指南'
featured_image = "/images/thumbnails/ct3003-1.jpg"
summary = "Cetorn 3003 路由器采用 MT7981 高性能芯片，支持 2.4GHz 和 5GHz 双频 Wi-Fi，配备千兆网口和多天线设计，提供稳定高速网络，适合家庭及小型办公使用。"
tags = ["CT3003", "MT7981", "MT7981B"]
+++

## 拆机

现在网上都是免拆机刷机方案，对于最新的系统已不支持，只能通过拆机的方式进行ttl刷机，拆机方式与360T7大同小异，拧开底部的两条螺丝，撬就完了。
![ct3003-0.jpg](/images/thumbnails/ct3003-0.jpg)

## 原厂固件分区表

![ct3003-2.jpg](/images/thumbnails/ct3003-2.jpg)

它和360T7的分区布局有一点不同，实际上ubi开始后面的不用管，因为刷了改版108M的FIP后面的都会变，主要看的就是前面的分区，CT3003多了一个art分区，但是art+Factory总的大小和360T7的Factory大小一样，也就是如果要刷360T7的固件，这里需要修正一下两个分区的内容（因为也并不能直接用360T7的固件，这里就无所谓了，知道一下怎么判断就行）

## 开启telnet和uboot控制台

此款路由器内置的原厂固件为基于mtk-sdk Linux 5.4内核的OpenWrt，uboot和OpenWrt的控制台终端均不可操作，但可以使用TTL进入OpenWrt的failsafe模式，从而开启telnet和uboot控制台。具体操作方法如下：

1. 拆机后找到下图红框内的UART串口。线序由上到下为RXD，TXD，GND，波特率115200
![360T7-4.png](/images/thumbnails/360T7-4.jpg)

2. 打开串口助手，上电，等待机器启动后，不断按下 f和回车键 ，直到出现下面的提示后，即可进入failsafe模式

```
[   10.205973] wed_get_slot_map(): assign slot_id:0 for entry: 0!
[   10.211812] wed_get_slot_map(): assign slot_id:1 for entry: 1!
[   10.218061] kmodloader: done loading kernel modules from /etc/modules-boot.d/*
[   10.235722] init: - preinit -
[   10.539480] mtk_soc_eth 15100000.ethernet eth0: configuring for fixed/2500base-x link mode
[   10.547859] mtk_soc_eth 15100000.ethernet eth0: Link is Up - 2.5Gbps/Full - flow control rx/tx
Press the [f] key and hit [enter] to enter failsafe mode
Press the [1], [2], [3] or [4] key and hit [enter] to select the debug level
```
3. 在failsafe模式依次执行以下操作

```
# 开启uboot控制台菜单
fw_setenv bootmenu_delay 3

# 挂载rootfs并开启telnet
mount_root
sed -i 's/.*local debug=.*/\tlocal debug=1/' /etc/init.d/telnet

# 修改root密码
passwd root
```

4. 通过网络备份原厂固件（可选）

```
# 将电脑的IP地址设置为192.168.1.2，插入路由器LAN口
# 使用nc监听3333端口并写入all.bin
# Windows系统可以使用netcat
# nc -l -p 3333 > all.bin

# 在路由器failsafe模式下开启网络
ifconfig eth0 0.0.0.0
brctl addbr br-lan
ifconfig br-lan 192.168.1.1 netmask 255.255.255.0 up
brctl addif br-lan eth0

# 读取/dev/mtd0，使用nc发送到192.168.1.2:3333
cat /dev/mtd0 | nc 192.168.1.2 3333
```

5. 重启路由器

> reboot
