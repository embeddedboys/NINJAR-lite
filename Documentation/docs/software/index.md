# <h1 align="center">软件文档</h1>

## 前言

本文将通过若干个实验，帮助读者快速掌握嵌入式Linux BSP开发基本知识</br>
本文并非按顺序讲解，读者可以按兴趣选择阅读。

实验环境：

| 设施 | 型号 |
| ---  | --- |
| NINJAR-lite | F1C200s ARM9 单核 64MB DDR1

> <p align="center">传送门</p>

## <h2>裸机</h2>
[`第一个裸机程序：blink`](noos/first_blink.md) </br>
[`初始化时钟： 程序跑的更快了`](noos/first_blink.md) </br>

### 外设
[`初始化UART: 跟pc打个招呼吧`]() </br>
[`初始化I2C: 连接更多的设备`]() </br>
[`初始化SPI: 连接更多的设备`]() </br>

### 进阶
使用

## <h2>uboot</h2>
### SPL
[`SPL：如何添加一个 Image Loader`](uboot/spl_image_loader.md) </br>

[`uboot：如何添加一个command`]() </br>
[`uboot：如何编写一个驱动`]() </br>

## <h2>linux</h2>
### <h3>内核开发</h3>
[`linux内核：神奇的initcall调用`](linux/amazing_initcall.md) </br>
[`linux内核：reboot & poweroff 流程修改`]() </br>
[`linux内核：添加一个系统调用`]() </br>
[`linux内核：中断流程`]() </br>

### <h3>内核接口</h3>
[`linux驱动：`]

### <h3>驱动开发</h3>
[`linux驱动：编写一个 cpufreq 驱动`]() </br>
[`linux驱动：编写一个 字符设备驱动`]() </br>
[`linux驱动：编写一个 Misc设备驱动`]() </br>
[`linux驱动：编写一个 led 驱动`]() </br>
[`linux驱动：编写一个 按键 驱动`]() </br>
[`linux驱动：编写一个 I2C 控制器驱动`]() </br>
[`linux驱动：编写一个 SPI 控制器驱动`]() </br>
[`linux驱动：编写一个 DMA 控制器驱动`]() </br>
[`linux驱动：编写一个 FrameBuffer 驱动`]() </br>

## <h2>rootfs 文件系统</h2>
### <h3>Raw</h3>
[`rootfs：jffs2 的使用`](rootfs/ubifs_usage.md) </br>
[`rootfs：UBIFS 的使用`](rootfs/ubifs_usage.md) </br>
[`rootfs：SquashFS 的使用`](rootfs/ubifs_usage.md) </br>
[`rootfs：overlayfs 的使用`](rootfs/ubifs_usage.md) </br>

### <h3>Buildroot</h3>

### <h3>Yocto</h3>

### <h3>打包</h3>
[`手动：将固件打包成SPI-Nor格式`]() </br>
[`手动：将固件打包成SPI-Nand格式`]() </br>
[`手动：将固件打包成sdcard/eMMC格式`]() </br>

## <h2>烧录</h2>