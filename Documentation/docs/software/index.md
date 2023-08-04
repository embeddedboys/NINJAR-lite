# <h1 align="center">软件文档</h1>

## 前言

本文将通过若干个实验，帮助读者快速掌握嵌入式Linux BSP开发基本知识</br>
本文并非按顺序讲解，读者可以按兴趣选择阅读。

实验环境：

| 设施 | 型号 |
| ---  | --- |
| `NINJAR-lite` | `F1C200s ARM9 单核 64MB DDR1`

> <p align="center">传送门</p>

## <h2>裸机</h2>
[`第一个裸机程序：blink`](noos/first_blink.md) </br>
[`初始化时钟： 程序跑的更快了`](noos/first_blink.md) </br>

### 外设
[`初始化UART: 跟pc打个招呼吧`](../default.md) </br>
[`初始化I2C: 连接更多的设备`](../default.md) </br>
[`初始化SPI: 连接更多的设备`](../default.md) </br>

### 进阶
[`使用jlink调试程序`](../default.md)</br>

## <h2>uboot</h2>
### SPL
[`SPL：如何添加一个 Image Loader`](uboot/spl_image_loader.md) </br>

[`uboot: 如何添加一个command`](../default.md) </br>
[`uboot: 如何编写一个驱动`](../default.md) </br>

## <h2>linux</h2>

### <h3>应用开发</h3>

[`linux应用开发: LVGL : 移植`](../default.md) </br>
[`linux应用开发: Rust : 环境搭建`](app/rust_setup.md) </br>

### <h3>内核开发</h3>
[`linux内核开发: 神奇的initcall调用`](linux/amazing_initcall.md) </br>
[`linux内核开发: reboot & poweroff 流程修改`](../default.md) </br>
[`linux内核开发: 添加一个系统调用`](../default.md) </br>
[`linux内核开发: 中断流程`](../default.md) </br>

### <h3>内核接口</h3>
[`linux内核接口: ioctl`](../default.md) </br>
[`linux内核接口: procfs`](../default.md) </br>
[`linux内核接口: sysfs`](../default.md) </br>
[`linux内核接口: 中断 interrupt`](../default.md) </br>
[`linux内核接口: 等待队列 waitqueue`](../default.md) </br>
[`linux内核接口: 工作队列 workqueue`](../default.md) </br>
[`linux内核接口: 链表 link list`](../default.md) </br>
[`linux内核接口: 内核线程 kernel thread`](../default.md) </br>
[`linux内核接口: 小任务 tasklet`](../default.md) </br>
[`linux内核接口: 自旋锁 spinlock`](../default.md) </br>
[`linux内核接口: 互斥体 mutex`](../default.md) </br>
[`linux内核接口: 原子量 atom`](../default.md) </br>
[`linux内核接口: 定时器 timer`](../default.md) </br>
[`linux内核接口: 完成 completion`](../default.md) </br>

### <h3>驱动接口</h3>
[`linux驱动接口: poll`](../default.md) </br>
[`linux驱动接口: e-poll`](../default.md) </br>
[`linux驱动接口: select`](../default.md) </br>
[`linux驱动接口: softirq`](../default.md) </br>
[`linux驱动接口: threaded irq`](../default.md) </br>

### <h3>驱动适配</h3>
[`linux驱动适配: i2c: iio: bmp280`](../default.md) </br>

[`linux驱动适配: spi: w5500`](../default.md) </br>

[`linux驱动适配: i2s: `](../default.md) </br>

[`linux驱动适配: fbtft: ssd1306`](../default.md) </br>
[`linux驱动适配: fbtft: st7735r`](../default.md) </br>
[`linux驱动适配: fbtft: st7789v`](../default.md) </br>
[`linux驱动适配: fbtft: ssd1327`](../default.md) </br>
[`linux驱动适配: fbtft: nv3030b`](../default.md) </br>

### <h3>驱动开发</h3>
[`linux驱动开发: 编写一个 cpufreq 驱动`](../default.md) </br>
[`linux驱动开发: 编写一个 字符设备驱动`](../default.md) </br>
[`linux驱动开发: 编写一个 Misc设备驱动`](../default.md) </br>
[`linux驱动开发: 编写一个 led 驱动`](../default.md) </br>
[`linux驱动开发: 编写一个 按键 驱动`](../default.md) </br>

[`linux驱动开发: 编写一个 I2C 控制器驱动`](../default.md) </br>
[`linux驱动开发: 编写一个 I2C 设备驱动`](../default.md) </br>

[`linux驱动开发: 编写一个 SPI 控制器驱动`](../default.md) </br>
[`linux驱动开发: 编写一个 SPI 设备驱动`](../default.md) </br>

[`linux驱动开发: 编写一个 DMA 控制器驱动`](../default.md) </br>
[`linux驱动开发: 编写一个 FrameBuffer 设备驱动`](../default.md) </br>
[`linux驱动开发: 编写一个 USB 设备驱动`](../default.md) </br>

### <h3>电源管理</h3>
[`linux电源管理: regulator子系统`](../default.md) </br>

## <h2>rootfs 文件系统</h2>
### <h3>Raw</h3>
[`rootfs: jffs2 的使用`](rootfs/ubifs_usage.md) </br>
[`rootfs: UBIFS 的使用`](rootfs/ubifs_usage.md) </br>
[`rootfs: SquashFS 的使用`](rootfs/ubifs_usage.md) </br>
[`rootfs: overlayfs 的使用`](rootfs/ubifs_usage.md) </br>

### <h3>Buildroot</h3>
[`Buildroot: 常用开发操作`](../default.md) </br>

### <h3>Yocto</h3>
[`Yocto: 什么是Yocto？`](../default.md) </br>
[`Yocto: 新建一个Yocto工程`](../default.md) </br>
[`Yocto: 如何在现有的工程上开发`](../default.md) </br>
[`Yocto: 如何添加一个layer`](../default.md) </br>
[`Yocto: 如何编写一个Recipe`](../default.md) </br>

### <h3>打包</h3>
[`手动: 将固件打包成SPI-Nor格式`](../default.md) </br>
[`手动: 将固件打包成SPI-Nand格式`](../default.md) </br>
[`手动: 将固件打包成sdcard/eMMC格式`](../default.md) </br>

## <h2>烧录</h2>

[`烧录: 通过g_mass_storage驱动烧写板上flash`](../default.md) </br>