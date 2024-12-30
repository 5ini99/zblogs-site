+++
date = '2024-12-17T11:17:53+08:00'
draft = false
title = 'Rax3000m Emmc Flashing Guide'
featured_image = "/images/thumbnails/rax3000m-emmc-0.png"
summary = "CMCC RAX3000M算力版是中国移动定制的一款路由器EMMC版本，64GB的emmc。"
tags = ["rax3000m", "emmc", "MT7981", "MT7981B"]
+++

CMCC RAX3000M是中国移动定制的一款路由器，分为两个版本，EMMC版本（算力版），NAND版本（普通版），区别是算力版有64GB的emmc，移动准备跑PCDN，普通版是128MB的闪存。

CMCC RAX3000M性价比很高，是搭载联发科MT7981B芯片目前最高的配置了，主要参数如下：

处理器：联发科MT7981B 双核1.3GHz
运行内存：512MB
eMMC存储：64GB
FEM功放：集成5路
无线协议：Wi-Fi 6 (802.11ax)
2.4G WiFi 2x2 574Mbps
5G WiFi 2x2 2402Mbps
网络接口：千兆网口 × 4
USB接口：USB 3.0 × 1

## CMCC RAX3000M
![rax3000m-emmc-1.jpg](/images/thumbnails/rax3000m-emmc-1.jpg)

### rax3000m版本判断
网上主流的说法是看路由器后面的标签来区分，找到路由器后面标签偏上的"制造商“，找到上面的字母CH：
![rax3000m-emmc-2.jpg](/images/thumbnails/rax3000m-emmc-2.jpg)
NAND： 只有CH
EMMC：CH后面还跟着EC

### 判断版本

但是这种方法不是绝对的，判断的唯一标准是开启ssh后，输入df -h命令查看你存储空间的大小，如果有一个50多G的分区，则说明是EMMC，否则是NAND

## 开启SSH

移动系统
![rax3000m-emmc-3.jpg](/images/thumbnails/rax3000m-emmc-3.jpg)
▲ 官方固件的系统完全没有可玩性、还有可能会被锁机，发挥不出这个硬件的性能必须刷机

EMMC 和 NAND的开启ssh步骤完全相同，都是导出配置->解密->修改配置->加密->导入配置 参考教程：https://blog.codee.top/rax3000m%E6%90%9E%E6%9C%BA%E7%9B%AE%E5%BD%95/

第一步：导出配置文件
首先在系统的：`配置管理->导出配置文件`得到一个`cfg_export_config_file.conf`的配置文件。

第二步：修改配置文件
把`cfg_export_config_file.conf`下载后上传到 `Linux系统`中，或者使用 `WSL子系统`，我这里使用的是`Ubuntu系统`。

解密配置文件：

```
openssl aes-256-cbc -d -pbkdf2 -k $CmDc#RaX30O0M@\!$ -in cfg_export_config_file.conf -out - | tar -zxvf -
```

用编辑器修改`etc/config/dropbear`文件，

```
vim etc/config/dropbear
```

![rax3000m-emmc-5.jpg](/images/thumbnails/rax3000m-emmc-5.jpg)

▲ 按i键进入插入模式把option enable '0'改为option enable '1'开启ssh服务，按esc然后：wq保存退出。

用编辑器修改/etc/shadow文件，清除root用户密码：

> vim /etc/shadow

![rax3000m-emmc-6.jpg](/images/thumbnails/rax3000m-emmc-6.jpg)

▲ 将root两个冒号间的密码删除然后：wq保存。

使用加密打包命令：

```
tar -zcvf - etc | openssl aes-256-cbc -pbkdf2 -k $CmDc#RaX30O0M@\!$ -out cfg_export_config_file_new.conf
```

![rax3000m-emmc-7.jpg](/images/thumbnails/rax3000m-emmc-7.jpg)

▲ 这里出现权限不够的报错，不用理会，实际已经加密打包好了

导入新配置文件
进入系统后台`配置管理->导入配置文件`，选择我们刚修改好的`cfg_export_config_file_new.conf`，重启路由器后就能使用root用户通过ssh访问了，默认地址：`192.168.10.1`

写入uboot文件
这个机子的uboot有3种，总的来说immortalwrt的uboot用的比较多，兼容性更好一点，采用这个。

> 参考immortalwrt刷入uboot：https://github.com/AngelaCooljx/Actions-rax3000m-emmc

下载uboot文件上传到路由器的/tmp/uboot目录下

![rax3000m-emmc-8.jpg](/images/thumbnails/rax3000m-emmc-8.jpg)

上传uboot文件

▲ 使用WinSCP工具把下载好的uboot文件上传到/tmp/uboot/目录，这里tmp目录里是没有uboot文件夹的，需要我们新建一个。

然后ssh进入该目录输入命令：

```
md5sum mt7981-cmcc_rax3000m-emmc-gpt.bin
md5sum mt7981-cmcc_rax3000m-emmc-bl2.bin
md5sum mt7981-cmcc_rax3000m-emmc-fip.bin
```

对比下MD5值为：

```
e6ceec4b9d3e86ef538c8b45c1b6ffed  mt7981-cmcc_rax3000m-emmc-gpt.bin
5b061eed5827146b0a14b774c3c57ab2  mt7981-cmcc_rax3000m-emmc-bl2.bin
f1e0b2f1618857ad4e76c8e1b91e7214  mt7981-cmcc_rax3000m-emmc-fip.bin
```


确保md5值一致，否则请停止下面的写入操作。

**下面命令是刷入的emmc版本的uboot，nand版本请不要乱刷**
```
dd if=mt7981-cmcc_rax3000m-emmc-gpt.bin of=/dev/mmcblk0 bs=512 seek=0 count=34 conv=fsync
echo 0 > /sys/block/mmcblk0boot0/force_ro
dd if=/dev/zero of=/dev/mmcblk0boot0 bs=512 count=8192 conv=fsync
dd if=mt7981-cmcc_rax3000m-emmc-bl2.bin of=/dev/mmcblk0boot0 bs=512 conv=fsync
dd if=/dev/zero of=/dev/mmcblk0 bs=512 seek=13312 count=8192 conv=fsync
dd if=mt7981-cmcc_rax3000m-emmc-fip.bin of=/dev/mmcblk0 bs=512 seek=13312 conv=fsync
```
![rax3000m-emmc-9.jpg](/images/thumbnails/rax3000m-emmc-9.jpg)
写入uboot

▲ 没有报错证明刷写uboot成功了！

![rax3000m-emmc-10.jpg](/images/thumbnails/rax3000m-emmc-10.jpg)
刷入OpenWrt固件

▲ 刷机过程很简单了，路由器连接电脑，电脑固定ip为`192.168.1.x`网段地址，浏览器进入`192.168.1.1`选择OpenWrt固件上传后，点击`Update`等大概3分钟，电脑设置DHCP观察有没有获取到ip地址即可。

![rax3000m-emmc-11.jpg](/images/thumbnails/rax3000m-emmc-11.jpg)
OpenWrt

▲ 默认用户和密码：`root` 后台地址：`192.168.6.1`

EMMC分区
进入系统后发现emmc没有挂载上，默认是没有分区的，有一部分闲置空间，需要通过cfdisk命令创建新分区：
```
opkg update
opkg install cfdisk
```
![rax3000m-emmc-12.jpg](/images/thumbnails/rax3000m-emmc-12.jpg)

cfdisk

▲ 使用ssh工具连接路由器，安装cfdisk命令

创建新分区
```
cfdisk /dev/mmcblk0
```
![rax3000m-emmc-13.jpg](/images/thumbnails/rax3000m-emmc-13.jpg)

▲ 找到最下面的Fress Space，选择New

![rax3000m-emmc-14.jpg](/images/thumbnails/rax3000m-emmc-14.jpg)

▲ 默认分配了最大的剩余空间，回车即可

![rax3000m-emmc-15.jpg](/images/thumbnails/rax3000m-emmc-15.jpg)

▲ 移动到write然后回车，输入yes回车，按q退出即可

![rax3000m-emmc-16.jpg](/images/thumbnails/rax3000m-emmc-16.jpg)

▲ 系统会自动挂载，进入挂载点查看有一个新的空间

USB连接手机共享网络
不方便接入宽带的地方可以用手机共享热点，路由器的USB口充当WAN口把手机当做网关出口

本固件已经编译了usb共享网络的插件，如果你的路由器没有这个插件，可以手动安装，步骤如下：

先更新软件库：

```
opkg update
```

安装以下组件：

```
kmod-usb-net
mod-usb-net-rndis
kmod-usb-net-cdc-ether
usbutils
```

通过以下命令一次性安装

```
opkg install kmod-usb-net kmod-usb-net-rndis kmod-usb-net-cdc-ether usbutils
```

注意：必须插入usb设备（手机），开启usb共享网络才会识别到！！！

![rax3000m-emmc-19.jpg](/images/thumbnails/rax3000m-emmc-19.jpg)

▲ 路由器先插入手机的usb后，这里会出现接口，否则没有~

![rax3000m-emmc-20.jpg](/images/thumbnails/rax3000m-emmc-20.jpg)

▲ 接口的防火墙区域选择`WAN`保存即可

![rax3000m-emmc-22.jpg](/images/thumbnails/rax3000m-emmc-22.jpg)

▲ 可以看到usb这个接口获取到了ip地址，也有数据了，随便打开个网页可以上网
