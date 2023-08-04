<h1 align="center">Linux UBIFS 文件系统的使用</h1>

平台环境如下

| 设施 | 版本 |
| --- | --- | 
| CPU | Allwinner F1C100s |
| linux | 6.4.0-rc4|
| uboot | v2023.07-rc4 |
| buildroot | v2023.02 |
| 闪存 | Winbond SPI-Nand 128MB W25N01G|

## 从 Buildroot 生成 UBIFS

![image](https://img2023.cnblogs.com/blog/2605173/202307/2605173-20230725164323515-388964025.jpg)


## 手动创建 UBIFS
-----------------------------------

参考如下内容
```c
3．制作 ubifs
Ubifs 的制作需要以下两个命令
mkfs.ubifs： 制作 UBIFS image
ubinize：根据 UBIFS image 制作 ubi.img，这个 ubi.img 是通过 u-boot 直接烧写在 nand flash 分区上的。

AM335x Linux SDK 里面带有制作好的文件系统，是.tar.gz 的压缩文件，可以解压在
一个目录下做为 UBI 文件系统内容，如/home/usr/fs。
GPEVM 板上的 NAND 型号为 MT29F2G08，page size 为 2048B，block size 为
64x2048B=131072B，block count 为 2048。如果制作针对 GPEVM 板的 ubifs，执行
如下两条命令:

1. $ mkfs.ubifs –F -q -r /home/usr/fs -m 2048 -e 126976 -c 2047 -o ubifs.img

参数简介：
-F：使能"white-space-fixup"，如果是通过 u-boot 烧写需要使能此功能。
-r：待制作的文件系统目录
-m：NAND FLASH 的最小读写单元，一般为 page size
-e：LEB size，对于 AM335x 的 NAND driver，为 block size-2x(page size)
-c：文件系统所占用的最大 block 数，一般小于等于 block count -1
-o：输出的 ubifs.img 文件

2. $ ubinize -o ubi.img -m 2048 -p 128KiB ubinize.cfg
参数简介：
-p：block size。
-m：NAND FLASH 的最小读写单元，一般为 page size
-o：输出的 ubi.img 文件
ubinize.cfg 为 ubinize 所需要的配置文件，内容如下：

[ubifs]
mode=ubi
image=ubifs.img
vol_id=0
vol_size=200MiB
vol_type=dynamic
vol_name=rootfs
vol_flags=autoresize

4.烧写 ubifs
可通过 u-boot 命令将生成的 ubi.img（25M）烧写到 NAND FLASH 分区上，如下示
例是将 ubi.img 先存储到 SD 卡上，然后通过 u-boot 的 fatload 命令将其拷贝至内存
中。

u-boot# mw.b 0x82000000 0xFF 
u-boot# mmc rescan
u-boot# fatload mmc 0 0x82000000 ubi.img
u-boot# nand erase 0x00780000 0xF880000
u-boot# nand write 0x82000000 0x00780000 0x1E00000
5.Linux 启动设置
在 U-boot 下设置启动信息如下：

 #setenv bootargs 'console=ttyO0,115200n8 noinitrd ip=off mem=256M 
 rootwait=1 rw ubi.mtd=7,2048 rootfstype=ubifs root=ubi0:rootfs 
init=/init'
```
以上是转载

-------------------

## uboot 传递的启动参数
```c
console=tty0 console=ttyS0,115200 panic=5 rootwait mtdparts=spi0.0:512K@0x20000(u-boot),128K@0x100000(dtb),5M@0x120000(kernel),24M@0x620000(rootfs),-(reserve) rw ubi.mtd=3,2048 rootfstype=ubifs root=ubi0:rootfs init=/linuxrc
```
注意，这里的 rootfs 的大小 必须要比ubinize.cfg里面设置的vol_size大一些，因为ubifs要预留一部分PEB来标记坏块等，具体预留多少，还没实验。

## linux 相关支持

### config
- 定位到 Device Drivers > Memory Technology Device (MTD) support

		开启如下选项：
		CONFIG_MTD_UBI

- 定位到 File systems > Miscellaneous filesystems 

		开启如下选项：
		CONFIG_UBIFS_FS
		可根据需求开启特性支持


### 启动日志
```c
[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 6.4.0-rc7-ninjar-lite+ (developer@lunarpc) (arm-linux-gnueabi-gcc (Ubuntu 12.2.0-14ubuntu2) 12.2.0, GNU ld (GNU Binutils for Ubuntu) 2.40) #13 3
[    0.000000] CPU: ARM926EJ-S [41069265] revision 5 (ARMv5TEJ), cr=0005317f
[    0.000000] CPU: VIVT data cache, VIVT instruction cache
[    0.000000] OF: fdt: Machine model: Lichee Pi Nano
[    0.000000] Memory policy: Data cache writeback
[    0.000000] Zone ranges:
[    0.000000]   Normal   [mem 0x0000000080000000-0x0000000081ffffff]
[    0.000000]   HighMem  empty
[    0.000000] Movable zone start for each node
[    0.000000] Early memory node ranges
[    0.000000]   node   0: [mem 0x0000000080000000-0x0000000081ffffff]
[    0.000000] Initmem setup node 0 [mem 0x0000000080000000-0x0000000081ffffff]
[    0.000000] Kernel command line: console=tty0 console=ttyS0,115200 panic=5 rootwait mtdparts=spi0.0:512K@0x20000(u-boot),128K@0x100000(dtb),5M@0x120000(kernel),24M@0x620c
[    0.000000] Dentry cache hash table entries: 4096 (order: 2, 16384 bytes, linear)
[    0.000000] Inode-cache hash table entries: 2048 (order: 1, 8192 bytes, linear)
[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 8128
[    0.000000] mem auto-init: stack:all(zero), heap alloc:off, heap free:off
[    0.000000] Memory: 21280K/32768K available (7168K kernel code, 588K rwdata, 1468K rodata, 1024K init, 267K bss, 11488K reserved, 0K cma-reserved, 0K highmem)
[    0.000000] SLUB: HWalign=32, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
[    0.000008] sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 89478484971ns
[    0.000096] clocksource: timer: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
[    0.000636] Console: colour dummy device 80x30
[    0.000678] printk: console [tty0] enabled
[    0.001353] Calibrating delay loop... 346.52 BogoMIPS (lpj=1732608)
[    0.060223] pid_max: default: 32768 minimum: 301
[    0.060690] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.060829] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.062183] CPU: Testing write buffer coherency: ok
[    0.065943] Setting up static identity map for 0x80100000 - 0x8010003c
[    0.067636] devtmpfs: initialized
[    0.072553] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 19112604462750000 ns
[    0.072733] futex hash table entries: 256 (order: -1, 3072 bytes, linear)
[    0.072998] pinctrl core: initialized pinctrl subsystem
[    0.075100] NET: Registered PF_NETLINK/PF_ROUTE protocol family
[    0.076740] DMA: preallocated 256 KiB pool for atomic coherent allocations
[    0.095107] SCSI subsystem initialized
[    0.095477] usbcore: registered new interface driver usbfs
[    0.095660] usbcore: registered new interface driver hub
[    0.095841] usbcore: registered new device driver usb
[    0.098603] clocksource: Switched to clocksource timer
[    0.130604] NET: Registered PF_INET protocol family
[    0.131191] IP idents hash table entries: 2048 (order: 2, 16384 bytes, linear)
[    0.132537] tcp_listen_portaddr_hash hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.132711] Table-perturb hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.132797] TCP established hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.132881] TCP bind hash table entries: 1024 (order: 1, 8192 bytes, linear)
[    0.133092] TCP: Hash tables configured (established 1024 bind 1024)
[    0.133343] UDP hash table entries: 256 (order: 0, 4096 bytes, linear)
[    0.133461] UDP-Lite hash table entries: 256 (order: 0, 4096 bytes, linear)
[    0.133895] NET: Registered PF_UNIX/PF_LOCAL protocol family
[    0.135424] RPC: Registered named UNIX socket transport module.
[    0.135556] RPC: Registered udp transport module.
[    0.135600] RPC: Registered tcp transport module.
[    0.135632] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.139709] NetWinder Floating Point Emulator V0.97 (double precision)
[    0.141230] Initialise system trusted keyrings
[    0.142052] workingset: timestamp_bits=30 max_order=13 bucket_order=0
[    0.143644] jffs2: version 2.2. (NAND) (SUMMARY)  © 2001-2006 Red Hat, Inc.
[    0.315146] Key type asymmetric registered
[    0.315277] Asymmetric key parser 'x509' registered
[    0.315564] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 253)
[    0.315662] io scheduler mq-deadline registered
[    0.315707] io scheduler kyber registered
[    0.509645] Serial: 8250/16550 driver, 8 ports, IRQ sharing disabled
[    0.516634] usbcore: registered new interface driver usb-storage
[    0.519379] usbcore: registered new interface driver usbhid
[    0.519496] usbhid: USB HID core driver
[    0.520325] NET: Registered PF_INET6 protocol family
[    0.523436] Segment Routing with IPv6
[    0.523723] In-situ OAM (IOAM) with IPv6
[    0.524017] sit: IPv6, IPv4 and MPLS over IPv4 tunneling driver
[    0.525654] NET: Registered PF_PACKET protocol family
[    0.525982] Key type dns_resolver registered
[    0.534001] Loading compiled-in X.509 certificates
[    0.553271] gpio gpiochip0: Static allocation of GPIO base is deprecated, use dynamic allocation.
[    0.560191] suniv-f1c100s-pinctrl 1c20800.pinctrl: initialized sunXi PIO driver
[    0.561635] suniv-f1c100s-pinctrl 1c20800.pinctrl: supply vcc-pe not found, using dummy regulator
[    0.564084] printk: console [ttyS0] disabled
[    0.584470] 1c25000.serial: ttyS0 at MMIO 0x1c25000 (irq = 116, base_baud = 6250000) is a 16550A
[    0.584690] printk: console [ttyS0] enabled
[    1.122530] 1c25800.serial: ttyS2 at MMIO 0x1c25800 (irq = 117, base_baud = 6250000) is a 16550A
[    1.132740] suniv-f1c100s-pinctrl 1c20800.pinctrl: supply vcc-pc not found, using dummy regulator
[    1.142538] sun6i-spi 1c05000.spi: Failed to request TX DMA channel
[    1.149026] sun6i-spi 1c05000.spi: Failed to request RX DMA channel
[    1.157449] spi-nand spi0.0: Winbond SPI NAND was found.
[    1.162959] spi-nand spi0.0: 128 MiB, block size: 128 KiB, page size: 2048, OOB size: 64
[    1.171473] 5 cmdlinepart partitions found on MTD device spi0.0
[    1.177465] Creating 5 MTD partitions on "spi0.0":
[    1.182365] 0x000000020000-0x0000000a0000 : "u-boot"
[    1.191517] 0x000000100000-0x000000120000 : "dtb"
[    1.199565] 0x000000120000-0x000000620000 : "kernel"
[    1.216775] 0x000000620000-0x000001e20000 : "rootfs"
[    1.267810] 0x000001e20000-0x000008000000 : "reserve"
[    1.450644] suniv-f1c100s-pinctrl 1c20800.pinctrl: supply vcc-pa not found, using dummy regulator
[    1.460485] sun6i-spi 1c06000.spi: Failed to request TX DMA channel
[    1.466865] sun6i-spi 1c06000.spi: Failed to request RX DMA channel
[    1.475672] rotary-encoder rotary@0: gray
[    1.480072] suniv-f1c100s-pinctrl 1c20800.pinctrl: supply vcc-pd not found, using dummy regulator
[    1.490985] input: rotary@0 as /devices/platform/rotary@0/input/input0
[    1.502130] usb_phy_generic usb_phy_generic.0.auto: dummy supplies not allowed for exclusive requests
[    1.512559] musb-hdrc musb-hdrc.1.auto: MUSB HDRC host driver
[    1.518485] musb-hdrc musb-hdrc.1.auto: new USB bus registered, assigned bus number 1
[    1.527316] suniv-f1c100s-pinctrl 1c20800.pinctrl: supply vcc-pf not found, using dummy regulator
[    1.541154] hub 1-0:1.0: USB hub found
[    1.545137] hub 1-0:1.0: 1 port detected
[    1.551775] ubi0: attaching mtd3
[    1.565601] sunxi-mmc 1c0f000.mmc: initialized, max. request size: 16384 KB
[    1.598898] random: crng init done
[    1.797859] ubi0: scanning is finished
[    1.818248] ubi0: attached mtd3 (name "rootfs", size 24 MiB)
[    1.824145] ubi0: PEB size: 131072 bytes (128 KiB), LEB size: 126976 bytes
[    1.831129] ubi0: min./max. I/O unit sizes: 2048/2048, sub-page size 2048
[    1.837943] ubi0: VID header offset: 2048 (aligned 2048), data offset: 4096
[    1.844970] ubi0: good PEBs: 192, bad PEBs: 0, corrupted PEBs: 0
[    1.851040] ubi0: user volume: 1, internal volumes: 1, max. volumes count: 128
[    1.858288] ubi0: max/mean erase counter: 2/0, WL threshold: 256, image sequence number: 1474628676
[    1.867396] ubi0: available PEBs: 0, total reserved PEBs: 192, PEBs reserved for bad PEB handling: 20
[    1.877734] input: gpio-keys as /devices/platform/gpio-keys/input/input1
[    1.884959] ubi0: background thread "ubi_bgt0d" started, PID 33
[    1.892277] cfg80211: Loading compiled-in X.509 certificates for regulatory database
[    1.907350] Loaded X.509 cert 'sforshee: 00b28ddf47aef9cea7'
[    1.913265] clk: Disabling unused clocks
[    1.918229] platform regulatory.0: Direct firmware load for regulatory.db failed with error -2
[    1.927061] cfg80211: failed to load regulatory.db
[    1.932659] UBIFS: parse sync
[    1.937945] UBIFS (ubi0:0): Mounting in unauthenticated mode
[    1.946638] UBIFS (ubi0:0): background thread "ubifs_bgt0_0" started, PID 37
[    2.098914] UBIFS (ubi0:0): UBIFS: mounted UBI device 0, volume 0, name "rootfs"
[    2.106435] UBIFS (ubi0:0): LEB size: 126976 bytes (124 KiB), min./max. I/O unit sizes: 2048 bytes/2048 bytes
[    2.116478] UBIFS (ubi0:0): FS size: 19935232 bytes (19 MiB, 157 LEBs), max 1023 LEBs, journal size 9023488 bytes (8 MiB, 72 LEBs)
[    2.128347] UBIFS (ubi0:0): reserved for root: 0 bytes (0 KiB)
[    2.134273] UBIFS (ubi0:0): media format: w4/r0 (latest is w5/r0), UUID CA1F2BD6-4AE3-4BCC-A9B1-7CEB78F71E82, small LPT model
[    2.150555] VFS: Mounted root (ubifs filesystem) on device 0:13.
[    2.158934] devtmpfs: mounted
[    2.165361] Freeing unused kernel image (initmem) memory: 1024K
[    2.171580] Run /linuxrc as init process

```

## 未解决的问题

USB Gadget ether 影响了 UBIFS 的初始化

我在内核中启用了CONFIG_USB_ETH=y，会导致如下情况发生：

第一次正常启动，没有问题 UBIFS 和 USB gadget ether 都正常工作

第二次启动，UBIFS会报错，无法正常启动，日志如下：
```c
[    1.581297] ubi0: scanning is finished
[    1.600434] ubi0 warning: ubi_eba_init: cannot reserve enough PEBs for bad PEB handling, reserved 4, need 20
[    1.612062] ubi0: attached mtd3 (name "rootfs", size 18 MiB)
[    1.617844] ubi0: PEB size: 131072 bytes (128 KiB), LEB size: 126976 bytes
[    1.624858] ubi0: min./max. I/O unit sizes: 2048/2048, sub-page size 2048
[    1.631727] ubi0: VID header offset: 2048 (aligned 2048), data offset: 4096
[    1.638712] ubi0: good PEBs: 144, bad PEBs: 0, corrupted PEBs: 0
[    1.644787] ubi0: user volume: 1, internal volumes: 1, max. volumes count: 128
[    1.652073] ubi0: max/mean erase counter: 2/0, WL threshold: 128, image sequence number: 1254135368
[    1.661184] ubi0: available PEBs: 0, total reserved PEBs: 144, PEBs reserved for bad PEB handling: 4
[    1.671391] input: gpio-keys as /devices/platform/gpio-keys/input/input1
[    1.678532] ubi0: background thread "ubi_bgt0d" started, PID 33
[    1.685842] cfg80211: Loading compiled-in X.509 certificates for regulatory database
[    1.701038] Loaded X.509 cert 'sforshee: 00b28ddf47aef9cea7'
[    1.706853] clk: Disabling unused clocks
[    1.711935] platform regulatory.0: Direct firmware load for regulatory.db failed with error -2
[    1.720770] cfg80211: failed to load regulatory.db
[    1.728279] UBIFS (ubi0:0): Mounting in unauthenticated mode
[    1.734752] UBIFS (ubi0:0): background thread "ubifs_bgt0_0" started, PID 37
[    1.777844] ubi0 warning: ubi_io_read: error -74 (ECC error) while reading 126976 bytes from PEB 3:4096, read only 126976 bytes, retry
[    1.825995] ubi0 warning: ubi_io_read: error -74 (ECC error) while reading 126976 bytes from PEB 3:4096, read only 126976 bytes, retry
[    1.874212] ubi0 warning: ubi_io_read: error -74 (ECC error) while reading 126976 bytes from PEB 3:4096, read only 126976 bytes, retry
[    1.922426] ubi0 error: ubi_io_read: error -74 (ECC error) while reading 126976 bytes from PEB 3:4096, read 126976 bytes
[    1.933608] CPU: 0 PID: 1 Comm: swapper Not tainted 6.4.0-rc7-ninjar-lite+ #11
[    1.940875] Hardware name: Allwinner suniv Family
[    1.945612]  unwind_backtrace from show_stack+0x10/0x14
[    1.950944]  show_stack from dump_stack_lvl+0x28/0x30
[    1.956087]  dump_stack_lvl from ubi_io_read+0x134/0x334
[    1.961500]  ubi_io_read from ubi_eba_read_leb+0x9c/0x464
```

我在看了一个网友的帖子说 USB Gadget Ether 影响了 UBIFS，我在关掉这个功能后可以正常使用UBIFS了，但是他也不知道具体原因。

有人知道的话，可以帮忙解释一下，感谢。

iotah 2023/7/25