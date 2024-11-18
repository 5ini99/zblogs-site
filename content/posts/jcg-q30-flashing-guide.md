+++
date = '2024-11-18T17:09:18+08:00'
draft = false
title = 'Jcg Q30/MR3000D-CIQ(512M)刷机指南'
featured_image = "/images/thumbnails/jcg-q30-1.jpg"
summary = "JCG Q30 Pro/MR3000D-CIQ(512M) 是一款采用了联发科技 Filogic 820 / MT7981 SoC 的 AX3000 WiFi 6 无线路由器，目前售价相对亲民，性价比较高。"
tags = ["JCG-Q30", "MR3000D-CIQ", "MT7981", "MT7981B"]
+++

原厂固件默认采用移动 DNS，且自带上报插件，加之移动定制路由器有锁机的传统艺能，刷机似乎是必需的。由天灵 @1715173329 维护的 OpenWrt 分支 ImmortalWrt 已经在 23.05-SNAPSHOT 开始提供了对这款机型，以及其他一些采用 MTK Filogic SoC 设备的支持。

## 刷入过渡固件

登录路由器默认后台 http://192.168.10.1 。密码和路由器背面/包装盒上的贴纸一致。

进入高级设置中的升级固件，选择先前下载的过渡固件 immortalwrt_mediatek_mt7981_mt7981_spim_nand_rfb_squashfs_sysupgrade.bin ，取消勾选保留配置，直接升级。

![jcg-q30-0.png](/images/thumbnails/jcg-q30-0.png)

## 刷入 ImmortalWrt 固件

上一步刷完固件之后，等待大约两分钟，然后断电，按住机身背部的 Reset 按键，上电开机。等待不到 10s 左右，红灯闪烁三下然后变成蓝灯，代表已进入 U-Boot 的恢复模式 WebUI。

![jcg-q30-2.png](/images/thumbnails/jcg-q30-2.png)

然后浏览器打开 http://192.168.1.1/ ，点击 upload 上传 Factory 固件。注意检查固件 MD5 是否正确。

上传完成后点击 Update

![jcg-q30-3.png](/images/thumbnails/jcg-q30-3.png)

![jcg-q30-4.png](/images/thumbnails/jcg-q30-4.png)