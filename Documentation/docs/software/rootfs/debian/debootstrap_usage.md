# 使用debootstrap构建debian根文件系统

## 构建根文件系统

#### 环境

cpu架构： armel 32bit
kernel版本： 5.18.0

#### 镜像的制作

#### Debian根文件系统的安装
第一阶段
```bash
# assume at ~/source/debian_work
# run as root

mkdir rootfs

# for armel
/usr/sbin/debootstrap --foreign --arch armel bullseye \
     rootfs https://mirrors.tuna.tsinghua.edu.cn/debian/

# for armhf
/usr/sbin/debootstrap --foreign --arch armhf bullseye \
     rootfs https://mirrors.tuna.tsinghua.edu.cn/debian/

cp /usr/bin/qemu-arm-static rootfs/usr/bin

LANG=en_US.UTF-8 chroot rootfs qemu-arm-static /bin/bash
```
第二阶段
如果目标架构与主机不同，则需要完成多阶段自举：
```bash
/debootstrap/debootstrap --second-stage
```

#### 安装必要的软件
先安装locales生成一下本地化信息
```
apt install locales
locale-gen en_US.UTF-8
```

```shell
apt install vim net-tools alsa-utils iw wireless-tools gcc g++ udhcpc ssh make git python3 python3-pip bash-completion
```

### 安装可选的软件
```shell
apt install build-essential
```

#### 安装显示管理器

#### 安装 xfce4 桌面
```bash

sudo apt install xorg --no-install-recommends
sudo apt install xfce4 --no-install-recommends


xinit -- :0 -nolisten tcp vt7 -noreset -f /dev/null &
```

#### 安装 lxde 桌面

#### 安装 lxqt 桌面

#### 安装 wmaker 桌面

#### 安装 fvwm-crystal 桌面

#### 安装 mate 桌面

## 打包UBI镜像

```shell
sudo mkfs.ubifs -d targetdir -e 0x1f000 -c 1023 -m 0x800 -x none -F -o ./rootfs.ubifs
ubinize -o rootfs.ubi -m 0x800 -p 0x20000 ./ubinize.cfg

> cat ubinize.cfg 
[ubifs]
mode=ubi
vol_id=0
vol_type=dynamic
vol_size=112MiB
vol_name=rootfs
vol_alignment=1
vol_flags=autoresize
image=./rootfs.ubifs

```

## 更多

### T113 buildroot sdk 移植debian

#### 修改内核配置
```bash
make linux-menuconfig
```

开启如下选项

`General Setup ----> Control Group support`

#### 修改设备树
dtc -I dtb -O dts ./sun8iw20p1-t113-100ask-t113-pro.dtb > dump.dts
```dts
        spi1_pins_a: spi1@0 {
                pins = "PD11", "PD12", "PD13", "PD14", "PD15"; /*clk mosi miso hold wp*/
                function = "spi1";
                drive-strength = <10>;
        };

        spi1_pins_b: spi1@1 {
                pins = "PD10";
                function = "spi1";
                drive-strength = <10>;
                bias-pull-up;   // only CS should be pulled up
        };

        spi1_pins_c: spi1@2 {
                allwinner,pins = "PD11", "PD12", "PD13", "PD14","PD15", "PD10";
                allwinner,function = "gpio_in";
                allwinner,muxsel = <0>;
                drive-strength = <10>;
        };

        st7789v: st7789v@0 {
                #address-cells = <1>;
                #size-cells = <1>;
                compatible = "sitronix,st7789v";
                status = "okay";
                reg = <0>;
                spi-max-frequency = <50000000>;
                rgb;
                buswidth = <8>;
                rotate = <90>;
                fps = <60>;
                spi-cpol;
                spi-cpha;
                reset-gpios = <&pio PB 3 GPIO_ACTIVE_HIGH>;
                dc-gpios = <&pio PD 22 GPIO_ACTIVE_LOW>;
                //debug = <1>;
        }; 
```

#### 打包debian文件系统为ext4格式
```bash
# generate a empty file, size :1024MB
dd if=/dev/zero of=rootfs.ext2 bs=1M count=2048

# pack rootfs into the ext2 file
sudo mke2fs -t ext2 -d ./rootfs -r 0 -L rootfs -O ^64bit -m 5 rootfs.ext2

# ext2 to ext4
sudo tune2fs -O extents,has_journal,large_file,huge_file ./rootfs.ext2
```

#### 使用buildroot的genimage工具，重新打包sd卡镜像
```bash
# 在 out/images 目录下
../host/bin/genimage --config ./genimage.cfg --rootpath=$PWD/../target --inputpath=$PWD --mcopy=../host/bin/mcopy --outputpath=$PWD
```
> 注意： 此环境基于百问网 t113 buildroot工程

```bash
# <file system> <mount pt>      <type>  <options>       <dump>  <pass>                                                                                                                         
/dev/root       /               ext2    rw,noauto       0       1
/dev/mmcblk0p4  /boot           vfat	rw,defaults		0		0
proc            /proc           proc    defaults        0       0
devpts          /dev/pts        devpts  defaults,gid=5,mode=620,ptmxmode=0666   0       0
tmpfs           /dev/shm        tmpfs   mode=0777       0       0
tmpfs           /tmp            tmpfs   mode=1777       0       0
tmpfs           /run            tmpfs   mode=0755,nosuid,nodev  0       0
sysfs           /sys            sysfs   defaults        0       0
/swap			swap			swap	defaults		0		0
```

#### 重新刷入sd卡
```bash
sudo dd if=./100ask-t113-pro_sdcard.img of=/dev/sda bs=4M
```

```
Disk /dev/sda：29.12 GiB，31266439168 字节，61067264 个扇区
Disk model: MassStorageClass
单元：扇区 / 1 * 512 = 512 字节
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：gpt
磁盘标识符：6B54CC89-E5AB-40E4-8552-38ED1A926CCD

设备          起点    末尾    扇区  大小 类型
/dev/sda1    35392   39487    4096    2M Linux 文件系统
/dev/sda2    39488   39743     256  128K Linux 文件系统
/dev/sda3    39744   39999     256  128K Linux 文件系统
/dev/sda4    40000  105535   65536   32M Linux 文件系统
/dev/sda5   105536 4299839 4194304    2G Linux 文件系统
/dev/sda6  4299840 4824127  524288  256M Linux 文件系统
	

# fdisk -l
Disk /dev/mmcblk0: 29 GB, 31266439168 bytes, 61067264 sectors
954176 cylinders, 4 heads, 16 sectors/track
Units: sectors of 1 * 512 = 512 bytes

Device       Boot StartCHS    EndCHS        StartLBA     EndLBA    Sectors  Size Id Type
/dev/mmcblk0p1 *  2,124,59    6,145,11         40000     105535      65536 32.0M  c Win95 FAT32 (LBA)
/dev/mmcblk0p2    6,145,12    267,166,27      105536    4299839    4194304 2048M 83 Linux
/dev/mmcblk0p3    267,166,28  300,73,29      4299840    4824127     524288  256M  c Win95 FAT32 (LBA)
/dev/mmcblk0p4    0,0,2       0,33,1               1       2079       2079 1039K ee EFI GPT

```

[PATCH]
```
From 5a2693e6c4d189d802e1e72d5dfb7c0631815361 Mon Sep 17 00:00:00 2001
From: huazheng <huazheng@keyirobot.com>
Date: Fri, 14 Apr 2023 18:15:33 +0800
Subject: [PATCH] patch fbtft to support 240x280 st7789v tft lcd

Signed-off-by: huazheng <huazheng@keyirobot.com>
---
 drivers/staging/fbtft/fb_st7789v.c | 50 +++++++++++++++++++++++++-
 drivers/staging/fbtft/fbtft-core.c | 58 ++++++++++++++++++++++++++++--
 2 files changed, 105 insertions(+), 3 deletions(-)

diff --git a/drivers/staging/fbtft/fb_st7789v.c b/drivers/staging/fbtft/fb_st7789v.c
index 3c3f38793..939cc508b 100644
--- a/drivers/staging/fbtft/fb_st7789v.c
+++ b/drivers/staging/fbtft/fb_st7789v.c
@@ -74,6 +74,7 @@ enum st7789v_command {
  *
  * Return: 0 on success, < 0 if error occurred.
  */
+#if 0
 static int init_display(struct fbtft_par *par)
 {
 	/* turn off sleep mode */
@@ -122,6 +123,53 @@ static int init_display(struct fbtft_par *par)
 	write_reg(par, MIPI_DCS_SET_DISPLAY_ON);
 	return 0;
 }
+#else
+static int init_display(struct fbtft_par *par)
+{
+        int rc;
+
+        par->fbtftops.reset(par);
+		mdelay(100);
+
+        write_reg(par, 0x11); //Sleep out
+        mdelay(120);     //Delay 120ms
+
+        //************* Start Initial Sequence **********//
+        write_reg(par, 0x36, 0x00);
+
+        write_reg(par, 0x3A, 0x05);
+
+        write_reg(par, 0xB2,0x0C,0x0C,0x00,0x33,0x33);
+
+        write_reg(par, 0xB7,0x35);
+
+        /* VCOM  = 1.35V */
+        write_reg(par, 0xBB, 0x32);
+
+        write_reg(par, 0xC2, 0x01);
+
+        /* GVDD = 4.8V */
+        write_reg(par, 0xC3,0x15);
+
+        /* VDX = 0V */
+        write_reg(par, 0xC4, 0x20);
+
+        /* FPS = 60Hz */
+        write_reg(par, 0xC6,0x0F);
+
+        write_reg(par, 0xD0, 0xA4, 0xA1);
+
+        write_reg(par, 0xE0,0xD0,0x08,0x0E,0x09,0x09,0x05,0x31,0x33,0x48,0x17,0x14,0x15,0x31,0x34);
+
+        write_reg(par, 0xE1,0xD0,0x08,0x0E,0x09,0x09,0x15,0x31,0x33,0x48,0x17,0x14,0x15,0x31,0x34);
+
+        write_reg(par, MIPI_DCS_ENTER_INVERT_MODE);
+
+        write_reg(par, MIPI_DCS_SET_DISPLAY_ON);
+
+        return 0;
+}
+#endif
 
 /**
  * set_var() - apply LCD properties like rotation and BGR mode
@@ -231,7 +279,7 @@ static int blank(struct fbtft_par *par, bool on)
 static struct fbtft_display display = {
 	.regwidth = 8,
 	.width = 240,
-	.height = 320,
+	.height = 280,
 	.gamma_num = 2,
 	.gamma_len = 14,
 	.gamma = DEFAULT_GAMMA,
diff --git a/drivers/staging/fbtft/fbtft-core.c b/drivers/staging/fbtft/fbtft-core.c
index 61f0286fb..27b9bc7b3 100644
--- a/drivers/staging/fbtft/fbtft-core.c
+++ b/drivers/staging/fbtft/fbtft-core.c
@@ -24,6 +24,7 @@
 #include <linux/platform_device.h>
 #include <linux/spinlock.h>
 #include <linux/of.h>
+#include <linux/of_gpio.h>
 #include <video/mipi_display.h>
 
 #include "fbtft.h"
@@ -71,6 +72,7 @@ void fbtft_dbg_hex(const struct device *dev, int groupsize,
 EXPORT_SYMBOL(fbtft_dbg_hex);
 
 #ifdef CONFIG_OF
+#if 0
 static int fbtft_request_one_gpio(struct fbtft_par *par,
 				  const char *name, int index,
 				  struct gpio_desc **gpiop)
@@ -91,6 +93,39 @@ static int fbtft_request_one_gpio(struct fbtft_par *par,
 
 	return ret;
 }
+#else
+static int fbtft_request_one_gpio(struct fbtft_par *par,
+                                  const char *name, int index,
+                                  struct gpio_desc **gpiop)
+{
+        int ret, gpio;
+        struct device *dev = par->info->device;
+        struct device_node *np = dev->of_node;
+        enum of_gpio_flags flags;
+
+        /* Get GPIO from device tree */
+        gpio = of_get_named_gpio_flags(np, name, index, &flags);
+        if (gpio < 0) {
+                return 0;
+        }
+
+
+        ret = devm_gpio_request_one(dev, gpio,
+                        (flags & OF_GPIO_ACTIVE_LOW)?GPIOF_OUT_INIT_LOW:GPIOF_OUT_INIT_HIGH,
+                        dev->driver->name);
+        if (ret) {
+                dev_err(dev, "Failed to request %s GPIO%d\n", name, gpio);
+                return -ENODEV;
+        }
+
+        *gpiop = gpio_to_desc(gpio);
+
+        fbtft_par_dbg(DEBUG_REQUEST_GPIOS, par, "%s: '%s' GPIO%d\n",
+                      __func__, name, gpio);
+
+        return ret;
+}
+#endif
 
 static int fbtft_request_gpios_dt(struct fbtft_par *par)
 {
@@ -100,10 +135,10 @@ static int fbtft_request_gpios_dt(struct fbtft_par *par)
 	if (!par->info->device->of_node)
 		return -EINVAL;
 
-	ret = fbtft_request_one_gpio(par, "reset", 0, &par->gpio.reset);
+	ret = fbtft_request_one_gpio(par, "reset-gpios", 0, &par->gpio.reset);
 	if (ret)
 		return ret;
-	ret = fbtft_request_one_gpio(par, "dc", 0, &par->gpio.dc);
+	ret = fbtft_request_one_gpio(par, "dc-gpios", 0, &par->gpio.dc);
 	if (ret)
 		return ret;
 	ret = fbtft_request_one_gpio(par, "rd", 0, &par->gpio.rd);
@@ -217,6 +252,8 @@ EXPORT_SYMBOL(fbtft_unregister_backlight);
 static void fbtft_set_addr_win(struct fbtft_par *par, int xs, int ys, int xe,
 			       int ye)
 {
+	xs = xs + 20;
+	xe = xe + 20;
 	write_reg(par, MIPI_DCS_SET_COLUMN_ADDRESS,
 		  (xs >> 8) & 0xFF, xs & 0xFF, (xe >> 8) & 0xFF, xe & 0xFF);
 
@@ -226,6 +263,7 @@ static void fbtft_set_addr_win(struct fbtft_par *par, int xs, int ys, int xe,
 	write_reg(par, MIPI_DCS_WRITE_MEMORY_START);
 }
 
+#if 0
 static void fbtft_reset(struct fbtft_par *par)
 {
 	if (!par->gpio.reset)
@@ -236,6 +274,22 @@ static void fbtft_reset(struct fbtft_par *par)
 	gpiod_set_value_cansleep(par->gpio.reset, 0);
 	msleep(120);
 }
+#else
+static void fbtft_reset(struct fbtft_par *par)
+{
+        if (!par->gpio.reset)
+                return;
+
+        fbtft_par_dbg(DEBUG_RESET, par, "%s()\n", __func__);
+
+        gpiod_set_value_cansleep(par->gpio.reset, 1);
+        usleep_range(20, 40);
+        gpiod_set_value_cansleep(par->gpio.reset, 0);   /* Low level reset */
+        msleep(120);
+
+        gpiod_set_value_cansleep(par->gpio.reset, 1);  /* Activate chip */
+}
+#endif
 
 static void fbtft_update_display(struct fbtft_par *par, unsigned int start_line,
 				 unsigned int end_line)
-- 
2.40.0

```

参考资料
-----------------------------
[Debian GNU/Linux 安装手册](https://www.debian.org/releases/stable/armhf/)
[通过 Unix/Linux 系统来安装 Debian GNU/Linux](https://www.debian.org/releases/stable/armhf/apds03.zh-cn.html)