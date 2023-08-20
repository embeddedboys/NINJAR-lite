# <h1 align="center">开发记录</h1>

## 结构设计篇

## 软件篇

### ninjar-lite-daemon 业务逻辑

检测USB热插拔

切换OTG功能

向外暴露WEB配置页面

设置系统、内核层配置

### ninjar-lite-app 业务逻辑

作为前台应用运行，只处理业务，软件形态暂未定义
### ninjar-lite-app准备工作
移植lvgl、输入

移植initcall

移植lwan, 失败了，完全不知道应该怎么单独拿出来编译

选择并移植一个优秀log库

做到这部分的时候，再考虑换Rust写app，因为期间了解了另外一个
非常优秀的gui框架 slint，同时这也是个学习Rust的极好机会

### 移植debian
尝试了最新的3个版本debian，即使最老的现行版本，在初始化安装后，文件系统的大小也有将近200MB
所以需要至少256MB Nand才能放下，感觉有点得不偿失了

### 移植emdebian
尝试移植了下，大小在100MB左右，这个作为一个备选方案


### 移植slint

## PCB设计篇

### 第三版PCB需求
核心板固定螺母位置调整

用户按键默认上拉

WIFI模组默认关闭

线路、铺铜优化

### 设计自己的屏幕板
### 外围模块设计——流水灯

## 服务层适配篇

### 加入gpsd支持

### 通过内核pps驱动 和 chrony 服务实现时间同步

## 底层适配篇

### 编码器功能测试 2023/7/12
在等新板子 2023/6/23

用的内核编码器驱动，正反转都能识别 2023/7/8

### 用户按键的测试

2023/7/10 用户按键应该给个默认状态，比如上拉，下一版pcb修改
### 验证NFS文件系统可行性 2023/7/5

### 解决开发板上gps模块的问题 2023/7/3
焊接问题，另外确定GPS_EN是否需要外部上拉

### usb主机模式，vbus主动供电，内核regulator驱动

### suniv-epd驱动在6.3.8内核上的编译 2023/7/1
需要保证内核fb相关内容开启

另外，6.x版本的内核，对于gpio flag的获取有了变化，对接口进行了更新。

### 测试gps模组功能
可通过标准tty设备读取gps模块上报内容

验证GPS_EN脚功能，运行时状态切换。

GPS_PPS引脚脉冲输出验证

```
2023/6/22 验证了从ttyS2读取报文
# cat /dev/ttyS2 
$GNGGA,,,,,,0,00,25.5,,,,,,*64

$GNGLL,,,,,,V,N*7A

$GNGSA,A,1,,,,,,,,,,,,,25.5,25.5,25.5,1*01

$GNGSA,A,1,,,,,,,,,,,,,25.5,25.5,25.5,4*04

$GPGSV,1,1,00,0*65

$BDGSV,1,1,00,0*74

$GNRMC,,V,,,,,,,,,,N,V*37

$GNVTG,,,,,,,,,N*2E

$GNZDA,,,,,,*56

$GPTXT,01,01,01,ANTENNA OK*35
```

### 6.4内核适配yaffs2文件系统
头大啊，内核打了yaffs2的补丁，编译一堆报错，看起来是新版内核vfs接口改了，应该不会很难搞吧。

放弃用yaffs2了，用ubifs应该更合适

### 测试USB网络稳定性
```
2023/6/22 对usb0配置ip时，注意网关不能与现有网络内冲突
64 bytes from 192.168.25.101: seq=485 ttl=64 time=0.504 ms
64 bytes from 192.168.25.101: seq=486 ttl=64 time=0.496 ms
64 bytes from 192.168.25.101: seq=487 ttl=64 time=0.513 ms
64 bytes from 192.168.25.101: seq=488 ttl=64 time=0.509 ms
64 bytes from 192.168.25.101: seq=489 ttl=64 time=0.527 ms
64 bytes from 192.168.25.101: seq=490 ttl=64 time=0.511 ms
64 bytes from 192.168.25.101: seq=491 ttl=64 time=0.512 ms
```

### 测试A31 I2C驱动可行性

### 显示屏相关的步骤
通过调节对比度的方式，模拟亮度

适配新版内核 fb_deferred_io

```
2023/6/22 可通过向 /sys/devices/platform/soc/1c06000.spi/spi_master/spi1/spi1.0/gamma 节点中写入0~255来调节亮度，值越大越亮

2023/7/1 显示屏ok，新版内核fb改了些东西，我暂时用全刷方式
```

### 测试其他屏幕的可行性
- 1.3 128x160 st7789v
- 1.54 200x200 ssd1681 e-ink
- 1.44 128x128 st7735s
- 1.54 240x240 st7789v

只是为了确保替代料的多样性

### 用户led的测试 2023/6/28
目前通过内核led子系统的trigger功能，将led配置为heartbeat，也可配置为其他trigger，例如与MTD活动联系起来。

### 测试nand flash稳定性及性能 2023/6/27
```
[ 7416.602565] =================================================
[ 7416.608455] mtd_speedtest: MTD device: 0
[ 7416.612407] mtd_speedtest: MTD device size 134217728, eraseblock size 131072, page size 2048, 4
[ 7416.634567] mtd_test: scanning for bad eraseblocks
[ 7416.900006] mtd_test: scanned 1024 eraseblocks, 0 are bad
[ 7417.664480] mtd_speedtest: testing eraseblock write speed
[ 7483.805228] mtd_speedtest: eraseblock write speed is 1981 KiB/s
[ 7483.811206] mtd_speedtest: testing eraseblock read speed
[ 7524.260132] mtd_speedtest: eraseblock read speed is 3240 KiB/s
[ 7525.113086] mtd_speedtest: testing page write speed
[ 7592.136201] mtd_speedtest: page write speed is 1955 KiB/s
[ 7592.141661] mtd_speedtest: testing page read speed
[ 7633.098185] mtd_speedtest: page read speed is 3200 KiB/s
[ 7633.953253] mtd_speedtest: testing 2 page write speed

 [ 7700.336261] mtd_speedtest: 2 page write speed is 1974 KiB/s
[ 7700.341892] mtd_speedtest: testing 2 page read speed
[ 7741.086483] mtd_speedtest: 2 page read speed is 3217 KiB/s
[ 7741.092029] mtd_speedtest: Testing erase speed
[ 7741.950513] mtd_speedtest: erase speed is 153493 KiB/s
[ 7741.955782] mtd_speedtest: Testing 2x multi-block erase speed
[ 7742.703943] mtd_speedtest: 2x multi-block erase speed is 176553 KiB/s
[ 7742.710515] mtd_speedtest: Testing 4x multi-block erase speed
[ 7743.455312] mtd_speedtest: 4x multi-block erase speed is 177382 KiB/s
[ 7743.461816] mtd_speedtest: Testing 8x multi-block erase speed
[ 7744.210430] mtd_speedtest: 8x multi-block erase speed is 176466 KiB/s
[ 7744.216999] mtd_speedtest: Testing 16x multi-block erase speed
[ 7744.970171] mtd_speedtest: 16x multi-block erase speed is 175389 KiB/s
[ 7744.976827] mtd_speedtest: Testing 32x multi-block erase speed
[ 7745.725682] mtd_speedtest: 32x multi-block erase speed is 176408 KiB/s
[ 7745.732270] mtd_speedtest: Testing 64x multi-block erase speed
[ 7746.482609] mtd_speedtest: 64x multi-block erase speed is 176083 KiB/s
[ 7746.489282] mtd_speedtest: finished
[ 7746.492824] =================================================
```

### 解决 spi nand 文件系统问题 2023/6/27

### 给uboot spl添加从spi nand启动的功能 2023/6/24

### 重做一份从spi nand启动的uboot、内核、文件系统 2023/6/22

uboot可以先用内存启动那个，可以快速测试内核和文件系统，建议在f1c200s上测试，不容易爆内存。

2023/6/21 uboot已经能够识别spi nand，并且能通过mtd命令操作nand

2023/6/22 对核心、内存频率进行了测试，核心频率最高只能跑到700MHz左右，内存频率暂时运行在256MHz

2023/6/22 重新配置内核，加入网络等支持

2023/6/26 最新修改的uboot，各项参数看起来都正常了，接下来就是要启动完整uboot
```
U-Boot SPL 2023.07-rc4-ninjar-lite-gcd863354e5-dirty (Jun 25 2023 - 11:49:37 -0400)
DRAM: 64 MiB
size=18, ptr=18, limit=100000: 81d00000
==== boot source from BROM: 0xe1a00000
Trying to boot from sunxi SPI
image magic : 0x56190527
image hcrc : 0x3e801d26
image name : U-Boot 2023.07-rc4-ninjar-lite-g 
spl_parse_legacy_header
payload image: U-Boot 2023.07-rc4-ninjar-lite-g  load addr: 0x81700000 size: 415592
u-boot name : U-Boot 2023.07-rc4-ninjar-lite-g 
         load addr : 0x81700000
             entry : 0x81700000
         load offt : 0x80000
              size : 0x65768
>>> spl_image->boot_device = 0x8
```

### 电源管理

### OTA升级

### recovery模式

给recovery内核单独搞一个分区，开机过程检测到用户按键处于按下状态，进入recovery模式

### Yocto工程搭建