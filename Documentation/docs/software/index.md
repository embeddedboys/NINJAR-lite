# <h1 align="center">软件文档</h1>

目录
----------------------

- [前言](#前言)
- [裸机](#裸机)
- [uboot](#uboot)
- [linux](#linux)
- [rootfs](#rootfs)
- [烧录](#烧录)

## 前言

本文将通过若干个实验，帮助读者快速掌握嵌入式Linux BSP开发基本知识</br>
本文并非按顺序讲解，读者可以按兴趣选择阅读。

实验环境：

| 设施 | 型号 |
| ---  | --- |
| `NINJAR-lite` | `F1C200s ARM9 64MB DDR1` |
| `Linux` | 6.5.1 stable |
| `uboot` | v2023.07.02 |

> <p align="center">传送门</p>

## <h2>裸机</h2>
[`【未完】第一个裸机程序：blink`](noos/first_blink.md) </br>
[`【待添加】初始化时钟： 程序跑的更快了`](../default.md) </br>

### 外设
[`【待添加】初始化UART: 跟pc打个招呼吧`](../default.md) </br>
[`【待添加】初始化I2C: 连接更多的设备`](../default.md) </br>
[`【待添加】初始化SPI: 连接更多的设备`](../default.md) </br>

### 进阶
[`【待添加】使用debugger调试程序`](../default.md)</br>

## <h2>Bring Up</h2>
[`【待添加】Bring Up: 如何适配主线linux`](../default.md) </br>
[`【待添加】Bring Up: 如何适配主线uboot`](../default.md) </br>
[`【待添加】Bring Up: 如何适配主线buildroot`](../default.md) </br>
[`【待添加】Bring Up: 如何适配主线yocto`](../default.md) </br>

## <h2>uboot</h2>
### SPL
[`【未完】SPL：如何添加一个 Image Loader`](uboot/spl_image_loader.md) </br>

[`【待添加】uboot: 如何添加一个command`](../default.md) </br>
[`【待添加】uboot: 如何编写一个驱动`](../default.md) </br>

### 驱动实战
[`【未完】 uboot驱动实战: 为SPI TFT添加显示支持`](uboot/driver_spi_tft.md) </br>

## <h2>linux</h2>

### <h3>应用开发</h3>

[`【待添加】linux应用开发: LVGL : 移植`](../default.md) </br>
[`【未完】linux应用开发: Rust : 环境搭建`](app/rust_setup.md) </br>
[`【未完】linux应用开发: 通过 i2c-dev 节点访问设备`](app/i2c-dev.md) </br>
[`【未完】linux应用开发: v4l2 数据获取流程`](app/v4l2.md) </br>

### <h3>内核开发</h3>
[`【未完】linux内核开发: 神奇的initcall调用`](linux/amazing_initcall.md) </br>
[`【待添加】linux内核开发: reboot & poweroff 流程修改`](../default.md) </br>
[`【待添加】linux内核开发: 添加一个系统调用`](../default.md) </br>
[`【待添加】linux内核开发: 中断流程`](../default.md) </br>

### <h3>内核接口</h3>
[`【待添加】linux内核接口: ioctl`](../default.md) </br>
[`【待添加】linux内核接口: procfs`](../default.md) </br>
[`【待添加】linux内核接口: sysfs`](../default.md) </br>
[`【待添加】linux内核接口: 中断 interrupt`](../default.md) </br>
[`【待添加】linux内核接口: 等待队列 waitqueue`](../default.md) </br>
[`【待添加】linux内核接口: 工作队列 workqueue`](../default.md) </br>
[`【待添加】linux内核接口: 链表 link list`](../default.md) </br>
[`【待添加】linux内核接口: 内核线程 kernel thread`](../default.md) </br>
[`【待添加】linux内核接口: 小任务 tasklet`](../default.md) </br>
[`【待添加】linux内核接口: 自旋锁 spinlock`](../default.md) </br>
[`【待添加】linux内核接口: 互斥体 mutex`](../default.md) </br>
[`【待添加】linux内核接口: 原子量 atom`](../default.md) </br>
[`【待添加】linux内核接口: 定时器 timer`](../default.md) </br>
[`【待添加】linux内核接口: 完成 completion`](../default.md) </br>

### <h3>内核配置</h3>
[`linux内核配置: USB声卡驱动`](linux/sound_card.md) </br>

### <h3>内核子系统</h3>
[`【待添加】Clock 子系统`](../default.md) </br>
[`【待添加】Interrupt 子系统`](../default.md) </br>
[`【待添加】电源管理 子系统`](../default.md) </br>
[`【待添加】GPIO 子系统`](../default.md) </br>
[`【待添加】Pinctrl 子系统`](../default.md) </br>
[`【待添加】Display 子系统`](../default.md) </br>
[`【待添加】V4L2 子系统`](../default.md) </br>
[`【待添加】ALSA 子系统`](../default.md) </br>
[`【待添加】Input 子系统`](../default.md) </br>
[`【待添加】I2C 子系统`](../default.md) </br>
[`【待添加】SPI 子系统`](../default.md) </br>
[`【待添加】IIO 子系统`](../default.md) </br>
[`【待添加】Regmap 子系统`](../default.md) </br>

### <h3>驱动接口</h3>
[`【待添加】linux驱动接口: poll`](../default.md) </br>
[`【待添加】linux驱动接口: e-poll`](../default.md) </br>
[`【待添加】linux驱动接口: select`](../default.md) </br>
[`【待添加】linux驱动接口: softirq`](../default.md) </br>
[`【待添加】linux驱动接口: threaded irq`](../default.md) </br>

### <h3>驱动适配</h3>
[`【待添加】linux驱动适配: i2c: iio: bmp280`](../default.md) </br>

[`【待添加】linux驱动适配: spi: w5500`](../default.md) </br>

[`【待添加】linux驱动适配: i2s: `](../default.md) </br>

[`【待添加】linux驱动适配: fbtft: ssd1306`](../default.md) </br>
[`【待添加】linux驱动适配: fbtft: st7735r`](../default.md) </br>
[`【未完】linux驱动适配: fbtft: st7789v`](linux/driver/fbtft_st7789v.md) </br>
[`【待添加】linux驱动适配: fbtft: ssd1327`](../default.md) </br>
[`【待添加】linux驱动适配: fbtft: nv3030b`](../default.md) </br>
[`【未完】linux驱动适配: sun4i-lradc-keys`](linux/driver/sun4i-lradc-keys.md) </br>
[`【未完】linux驱动适配: rtl8188eus`](linux/driver/rtl8188eus.md) </br>
[`【未完】linux驱动适配: USB Gadget g_ether`](linux/driver/usb_gadget_ether.md) </br>

### <h3>驱动开发</h3>
[`【待添加】linux驱动开发: 编写一个 cpufreq 驱动`](../default.md) </br>
[`【待添加】linux驱动开发: 编写一个 字符设备驱动`](../default.md) </br>
[`【待添加】linux驱动开发: 编写一个 Misc设备驱动`](../default.md) </br>
[`【待添加】linux驱动开发: 编写一个 led 驱动`](../default.md) </br>
[`【待添加】linux驱动开发: 编写一个 按键 驱动`](../default.md) </br>

[`【待添加】linux驱动开发: 编写一个 I2C 控制器驱动`](../default.md) </br>
[`【待添加】linux驱动开发: 编写一个 I2C 设备驱动`](../default.md) </br>

[`【待添加】linux驱动开发: 编写一个 SPI 控制器驱动`](../default.md) </br>
[`【待添加】linux驱动开发: 编写一个 SPI 设备驱动`](../default.md) </br>

[`【待添加】linux驱动开发: 编写一个 DMA 控制器驱动`](../default.md) </br>
[`【待添加】linux驱动开发: 编写一个 FrameBuffer 设备驱动`](../default.md) </br>
[`【待添加】linux驱动开发: 编写一个 USB 设备驱动`](../default.md) </br>

### <h3>电源管理</h3>
[`【待添加】linux电源管理: regulator子系统`](../default.md) </br>

### <h3>杂项</h3>
[`【未完】linux杂项: 命令行邮箱开发环境配置`](linux/misc/contributor.md) </br>
[`【未完】linux杂项: 向社区贡献代码`](linux/misc/contributor.md) </br>
[`【未完】linux杂项: 修改控制台字体`](linux/misc/console_font.md) </br>
[`【未完】linux杂项: 修改控制台光标行为`](linux/misc/console_cursor.md) </br>

## <h2>rootfs</h2>
### <h3>Raw</h3>
[`【待添加】rootfs: jffs2 的使用`](../default.md) </br>
[`【未完】rootfs: UBIFS 的使用`](rootfs/raw/ubifs_usage.md) </br>
[`【未完】rootfs: SquashFS 的使用`](rootfs/raw/squashfs_usage.md) </br>
[`【未完】rootfs: overlayfs 的使用`](../default.md) </br>

### <h3>Buildroot</h3>
[`【待添加】Buildroot: 常用开发操作`](../default.md) </br>

### <h3>Yocto</h3>
[`【待添加】Yocto: 什么是Yocto？`](../default.md) </br>
[`【待添加】Yocto: 新建一个Yocto工程`](../default.md) </br>
[`【待添加】Yocto: 如何在现有的工程上开发`](../default.md) </br>
[`【未完】Yocto: 如何添加一个layer`](rootfs/yocto/yocto_new_layer.md) </br>
[`【待添加】Yocto: 如何编写一个Recipe`](../default.md) </br>
[`【未完】Yocto: 修改SDK环境加载脚本中的flags`](rootfs/yocto/yocto_sdk_flags.md)

### <h3>debian</h3>
[`【未完】使用debootstrap构建debian根文件系统`](rootfs/debian/debootstrap_usage.md)</br>
[`【待添加】使用multistrap构建emdebian文件系统`](../default.md) </br>

### <h3>打包</h3>
[`【待添加】手动: 将固件打包成SPI-Nor格式`](../default.md) </br>
[`【待添加】手动: 将固件打包成SPI-Nand格式`](../default.md) </br>
[`【待添加】手动: 将固件打包成sdcard/eMMC格式`](../default.md) </br>

## <h2>调试</h2>
[`qemu`]() </br>

## <h2>烧录</h2>

[`【待添加】烧录: 通过USB模拟板上Flash为大容量存储设备进行固件烧录`](../default.md) </br>