+++
date = '2024-11-18T09:55:53+08:00'
draft = false
title = '360T7刷机指南'
featured_image = "/images/thumbnails/360T7-1.png"
summary = "科普：360T7是中国电信运营商的订制机、360T7m是中国移动运营商的订制机、360T7u是中国联通运营商的订制机，三款机器配置上无任何区别，360T7是目前市面上的第一款MT7981（filogic 820）运营商定制机，具有价格便宜、信号强（远强于同价位其它垃圾高通和bcm方案路由器）、可玩性高。"
+++

> 科普知识：360T7是中国电信运营商的订制机、360T7m是中国移动运营商的订制机、360T7u是中国联通运营商的订制机，三款机器配置上无任何区别。

![360T7-1.png](/images/thumbnails/360T7-0.jpeg)

360T7是目前市面上的第一款MT7981（filogic 820）运营商定制机，具有价格便宜、信号强（远强于同价位其它垃圾高通和bcm方案路由器）、可玩性高（刷机方法请看此处）的特点，其配置如下：

- CPU：MT7981B 双核A53 1.3GHz
- RAM：256M DDR3
- FLASH：128M SPI NAND
- 无线phy：MT7976CN AX3000
- 交换机：MT7531A 2xHSGMII

## 拆机

由于目前360T7没有免拆方案必须拆机刷机，撕开底部标签可以看到有两颗螺丝，拧掉螺丝然后顺着缝隙用撬棒大力出奇迹就开了。
![360T7-2.png](/images/thumbnails/360T7-2.jpg)
![360T7-3.png](/images/thumbnails/360T7-3.jpg)

## 原厂固件分区表

| 起始地址             | 结束地址            | 名称       |
|--------------------|--------------------|------------|
| 0x000000000000      | 0x000000100000      | bl2        |
| 0x000000100000      | 0x000000180000      | u-boot-env |
| 0x000000180000      | 0x000000380000      | Factory    |
| 0x000000380000      | 0x000000580000      | fip        |
| 0x000000580000      | 0x000002980000      | ubi        |
| 0x000002980000      | 0x000004d80000      | firmware-1 |
| 0x000004d80000      | 0x000007180000      | plugin     |
| 0x000007180000      | 0x000007280000      | config     |
| 0x000007280000      | 0x000007300000      | factory    |
| 0x000007300000      | 0x000007a00000      | log        |

其中，Factory为无线EEPROM分区；fip为uboot分区；ubi和firmware-1为固件分区，分别36M，均为ubi格式；plugin为原厂插件分区，有36M，也是ubi格式；最后一个小写字母开头的factory分区为原厂固件信息分区，保存有机器编号，MAC地址等信息。

原厂uboot在开机时会分别检查ubi和firmware-1分区内是否存在固件，如果某个分区未检查通过，则uboot会自动将另一个分区的内容复制过去。

因此，当使用原厂uboot启动时，只能使用一个ubi分区存放固件，固件总体积（含kernel+rootfs+rootfs_data）将限制在36M内，但你仍然可以使用plugin分区（36M）存放其它数据。

## 开启telnet和uboot控制台

此款路由器内置的原厂固件为基于mtk-sdk Linux 5.4内核的OpenWrt，uboot和OpenWrt的控制台终端均不可操作，但可以使用TTL进入OpenWrt的failsafe模式，从而开启telnet和uboot控制台。具体操作方法如下：

1. 拆机后找到下图红框内的UART串口。线序由上到下为RXD，TXD，GND，波特率115200
![360T7-4.png](/images/thumbnails/360T7-4.jpg)

2. 打开串口助手，上电，等待机器启动后，不断按下 f和回车键 ，直到出现下面的提示后，即可进入failsafe模式(如果不显示如下的代码，没有f选项的情况下登录到路由器固件升级界面，升级`360T7-v4.2.4.7959_upgrade.bin`固件以后再次启动即可提示f模式)

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

## 原厂uboot内刷写固件

建议在原厂uboot控制台内使用mtkupgrade工具刷写固件，使用方法如下图所示

![360T7-5.jpg](/images/thumbnails/360T7-5.jpg)

## 原厂OpenWrt系统内刷写固件

直接使用mtd工具即可，例如

> mtd -r write openwrt-squashfs-factory.bin ubi
