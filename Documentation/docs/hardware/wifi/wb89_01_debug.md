
<h1 align="center">WB89-01 WIFI模块问题分析</h1>

> ## 目录

- [名词解释](#名词解释)
- [写在前面](#写在前面)
- [参考资料](#参考资料)

## 名词解释
| 名词 | 解释 |
| --- | --- |

## 写在前面

本文将从问题入手，通过飞线方式，查找板上问题。

| 设施 | 版本 |
| --- | --- |
| 开发板 | f1c100s_base |
| WIFI模组 | WB89-01 |
| linux | 6.5.6

<img src="../assets/wb89-01.jpg" width="30%"> </br>

## 问题

板子上电后，无法扫描到任何mmc设备。加载esp8089 wifi驱动，没有任何反应

## 调试记录

### 确认问题原因

#### 1. 是否是MMC接口的问题？

我准备了这样一个 SPI 接口的 TF 卡模块

<div align="center">
<img src="../assets/tf_module.jpg" width="50%">
</div>

> 这里要说明一下，TF 卡是可以工作于SDIO或者SPI模式下的

TF卡的引脚定义如下图所示

<div align="center">
<img src="../assets/image.png" width="80%">
</div>

需要将模块上的引脚与SDIO的接口连接起来，例如:

- DO --> DAT0
- VSS --> VSS
- VDD --> VDD
- DI --> CMD
- SCLK --> CLK

我们让其工作于sdio 1bit-mode下

将 WIFI 模组通过热风枪取下，接入tf卡，看到串口有如下信息打印
```c
# [   77.847103] mmc0: new high speed SDHC card at address e624
[   77.859029] mmcblk0: mmc0:e624 SU04G 3.69 GiB 
[   77.882709]  mmcblk0:
```

通过debugfs查看mmc设备
```
# mount -t debugfs none /sys/kernel/debug

# ls /sys/kernel/debug/mmc0
caps       caps2      clock      err_state  err_stats  ios        mmc0:e624
```

#### 结论

- 板子上的mmc接口工作正常

#### 2. 是否是WIFI模组的问题？

将WIFI模组飞线，如下图所示

<div align="center">
<img src="../assets/module_wiring.jpg" width="50%"> </br>
</div>

连接至板子的过程中，我发现原理图居然画错了

<div align="center">
</div>

<img src="../assets/wb89-01_wrong_symbol.png" width="80%">

> 图中标注的地方

通过查看说明书中的pinout

<div align="center">
<img src="../assets/wb89-01_pinout.png" width="50%">
</div>

修正后的原理图如下图所示

<div align="center">
<img src="../assets/wb89-01_right_symbol.png" width="80%">
</div>

来到PCB这边，现在的连接如下图所示

<div align="center">
<img src="../assets/wb89-01_wrong_pcb.png" width="50%">
</div>

可以看到8、9号焊盘和13、14画反了，还好这边的焊盘是对称的，实验还可以进行下去。

我们先继续等会再修改PCB。 将WIIF模组飞线连接好后，如下图所示

<div align="center">
<img src="../assets/module_board_wiring.jpg" width="100%">
</div>


给板子上电，发现串口出现如下打印信息，mmc卡已经识别到了
```c
[    1.627762] sunxi-mmc 1c0f000.mmc: initialized, max. request size: 16384 KB
[    1.635327] ubi0: attaching mtd3
[    1.663866] mmc0: queuing unknown CIS tuple 0x01 [d9 01 ff] (3 bytes)
[    1.685014] random: crng init done
[    1.688583] mmc0: queuing unknown CIS tuple 0x1a [01 01 00 02 07] (5 bytes)
[    1.700617] mmc0: queuing unknown CIS tuple 0x1b [c1 41 30 30 ff ff ff ff] (8 bytes)
```
#### 结论

- 原理图符号画错，导致PCB板与模组的连接出现问题，模组没有上电。



我们接着进度，尝试加载我修改适配6.x内核的esp8089驱动，不出所料，果然崩了😂，看日志是个空指针错误
```c
# insmod esp8089.ko
[ 1110.173154] esp_sdio_init
[ 1110.177417] ------------[ cut here ]------------
[ 1110.183063] WARNING: CPU: 0 PID: 106 at net/mac80211/main.c:634 ieee80211_alloc_hw_nm+0xb4/0x670
[ 1110.193900] Modules linked in: esp8089(O+) g_ether fb_ssd1327(O)
[ 1110.201599] CPU: 0 PID: 106 Comm: insmod Tainted: G           O       6.4.0-rc7-ninjar-lite+ #1
[ 1110.212248] Hardware name: Allwinner suniv Family
[ 1110.217962]  unwind_backtrace from show_stack+0x10/0x14
[ 1110.224292]  show_stack from dump_stack_lvl+0x28/0x30
[ 1110.230410]  dump_stack_lvl from __warn+0xa0/0xd8
[ 1110.236125]  __warn from warn_slowpath_fmt+0x178/0x194
[ 1110.242255]  warn_slowpath_fmt from ieee80211_alloc_hw_nm+0xb4/0x670
[ 1110.250064]  ieee80211_alloc_hw_nm from esp_pub_alloc_mac80211+0x1c/0x170 [esp8089]
[ 1110.259449]  esp_pub_alloc_mac80211 [esp8089] from esp_sdio_probe+0x270/0x40c [esp8089]
[ 1110.269250]  esp_sdio_probe [esp8089] from sdio_bus_probe+0xbc/0x17c
[ 1110.277291]  sdio_bus_probe from really_probe+0xc4/0x298
[ 1110.283646]  really_probe from __driver_probe_device+0x84/0x19c
[ 1110.291024]  __driver_probe_device from driver_probe_device+0x30/0xdc
[ 1110.298924]  driver_probe_device from __driver_attach+0x94/0x104
[ 1110.306381]  __driver_attach from bus_for_each_dev+0x6c/0xb8
[ 1110.313049]  bus_for_each_dev from bus_add_driver+0x138/0x1cc
[ 1110.319798]  bus_add_driver from driver_register+0x7c/0x114
[ 1110.326369]  driver_register from esp_sdio_init+0x1c/0x30 [esp8089]
[ 1110.334305]  esp_sdio_init [esp8089] from do_one_initcall+0x48/0x248
[ 1110.342317]  do_one_initcall from do_init_module+0x48/0x1cc
[ 1110.348876]  do_init_module from load_module+0x1720/0x18b8
[ 1110.355352]  load_module from sys_finit_module+0xb8/0x104
[ 1110.361721]  sys_finit_module from ret_fast_syscall+0x0/0x44
[ 1110.368337] Exception stack(0xc2909fa8 to 0xc2909ff0)
[ 1110.374309] 9fa0:                   193a7662 00000001 00000003 004fee30 00000000 be837f4d
[ 1110.383836] 9fc0: 193a7662 00000001 00000003 0000017b be837fef 00000000 00000001 00000000
[ 1110.393391] 9fe0: be837db4 be837d98 00434f00 b6ef709c
[ 1110.399588] ---[ end trace 0000000000000000 ]---
[ 1110.405324] ieee80211 can't alloc hw!
[ 1110.410025] 8<--- cut here ---
[ 1110.414095] Unable to handle kernel NULL pointer dereference at virtual address 00000004 when write
[ 1110.425333] [00000004] *pgd=00000000
[ 1110.430019] Internal error: Oops: 805 [#1] PREEMPT ARM
[ 1110.436244] Modules linked in: esp8089(O+) g_ether fb_ssd1327(O)
[ 1110.443851] CPU: 0 PID: 106 Comm: insmod Tainted: G        W  O       6.4.0-rc7-ninjar-lite+ #1
[ 1110.454598] Hardware name: Allwinner suniv Family
[ 1110.460321] PC is at esp_sdio_probe+0x27c/0x40c [esp8089]
[ 1110.467032] LR is at esp_pub_alloc_mac80211+0x130/0x170 [esp8089]
[ 1110.474828] pc : [<bf016eb8>]    lr : [<bf01df98>]    psr: a0000013
[ 1110.482548] sp : c2909c98  ip : ffffefff  fp : c0f82800
[ 1110.488755] r10: bf02011c  r9 : c0fa4a08  r8 : bf009a00
[ 1110.494959] r7 : bf009000  r6 : fffffff4  r5 : c0fa4a00  r4 : c1961c80
[ 1110.502944] r3 : 00000000  r2 : 9683abd4  r1 : c2909bd0  r0 : fffffff4
[ 1110.510969] Flags: NzCv  IRQs on  FIQs on  Mode SVC_32  ISA ARM  Segment none
[ 1110.519626] Control: 0005317f  Table: 81944000  DAC: 00000051
[ 1110.526407] Register r0 information: non-paged memory
[ 1110.532525] Register r1 information: 2-page vmalloc region starting at 0xc2908000 allocated at kernel_clone+0xb4/0x3b4
[ 1110.545350] Register r2 information: non-paged memory
[ 1110.551458] Register r3 information: NULL pointer
[ 1110.557204] Register r4 information: slab kmalloc-128 start c1961c80 pointer offset 0 size 128
[ 1110.567849] Register r5 information: slab kmalloc-512 start c0fa4a00 pointer offset 0 size 512
[ 1110.578465] Register r6 information: non-paged memory
[ 1110.584546] Register r7 information: 4-page vmalloc region starting at 0xbf009000 allocated at load_module+0x6b4/0x18b8
[ 1110.597386] Register r8 information: 4-page vmalloc region starting at 0xbf009000 allocated at load_module+0x6b4/0x18b8
[ 1110.610219] Register r9 information: slab kmalloc-512 start c0fa4a00 pointer offset 8 size 512
[ 1110.620846] Register r10 information: 2-page vmalloc region starting at 0xbf020000 allocated at load_module+0x6b4/0x18b8
[ 1110.633772] Register r11 information: slab kmalloc-1k start c0f82800 pointer offset 0 size 1024
[ 1110.644451] Register r12 information: non-paged memory
[ 1110.650593] Process insmod (pid: 106, stack limit = 0x262a8b3e)
[ 1110.657995] Stack: (0xc2909c98 to 0xc290a000)
[ 1110.663370] 9c80:                                                       c0bb8228 bf0097a0
[ 1110.673029] 9ca0: c2909f40 c0fa4a08 c0fa4a00 00000000 bf009010 bf02011c c0bb8228 bf0097a0
[ 1110.682662] 9cc0: c2909f40 c04f24fc c0fa4a08 00000000 bf009010 00000000 c0cf0900 c0409ce8
[ 1110.692297] 9ce0: c0fa4a08 bf009010 c0fa4a08 00000014 c0cf0900 c0409f40 00000000 c028a9ac
[ 1110.701978] 9d00: 00000000 c0bd0fb0 c0fa4a08 c0fa4a08 00000014 c0cf0900 c0bb8228 bf0097a0
[ 1110.711681] 9d20: c2909f40 c040a088 bf009010 c0fa4a08 c040a1f8 c0b0322c c0cf0900 c040a28c
[ 1110.721387] 9d40: 00000000 bf009010 c040a1f8 c0407fdc 00000000 c0cf094c c0fcb030 9683abd4
[ 1110.731105] 9d60: bf009010 c1897360 c0cf0900 00000000 c1897394 c0408ed4 bf020ae0 c18bd900
[ 1110.740871] 9d80: c0b95000 bf009010 bf0160fc 00000000 c18bd900 c0b95000 c0bb8228 c040aaf4
[ 1110.750676] 9da0: c0b0322c bf0160fc 00000000 bf016118 c0b0322c c0101da4 00000000 c0739804
[ 1110.760491] 9dc0: 00000024 00000000 c0807f90 c184b848 00000036 c01ef4c4 00000000 c01f4c44
[ 1110.770308] 9de0: c0b0322c c028a7d0 c2a0efff 00081938 c1969160 c1fec700 00000000 00081938
[ 1110.780105] 9e00: c18bd900 c1fec700 00000000 9683abd4 bf0097a0 bf0097a0 c1969160 0000000c
[ 1110.789927] 9e20: 00000010 00000000 c0bb8228 bf0097a0 c2909f40 c0757cfc 00000000 00000000
[ 1110.799769] 9e40: 00000000 00000000 0000000c c0169948 ffff8000 00007fff bf0097a0 c0167410
[ 1110.809587] 9e60: 00000001 bf0097ac 00000001 bf0097a0 bf023004 00000001 0000001b c29fb000
[ 1110.819400] 9e80: 00000007 c2a0d81c 00000000 c2909f40 00000001 004fee30 00000000 c190e8c0
[ 1110.829216] 9ea0: c0b0322c c2909f3c 00012ccc 00012ccc 00000000 00000000 00000000 00000000
[ 1110.839016] 9ec0: 6e72656b 00006c65 00000000 00000000 00000000 00000000 00000000 00000000
[ 1110.848798] 9ee0: 00000000 00000000 00000000 00000000 00000000 00000000 00000000 9683abd4
[ 1110.858547] 9f00: c2909f3c 00000000 c0b0322c 004fee30 00000003 c0100234 c18bd900 00000000
[ 1110.868278] 9f20: 00000000 c0169cec c2909f3c 7fffffff 00000000 00000002 c0213194 c29fb000
[ 1110.878016] 9f40: c2a04fd0 c2a05820 c29fb000 00012ccc c2a0d81c c2a0d718 c2a08de0 00000a88
[ 1110.887724] 9f60: 00002338 0000479c 000030f3 00000000 0000478c 0000001b 0000001c 00000011
[ 1110.897441] 9f80: 00000000 00000007 00000000 9683abd4 004fe440 193a7662 00000001 00000003
[ 1110.907153] 9fa0: 0000017b c0100040 193a7662 00000001 00000003 004fee30 00000000 be837f4d
[ 1110.916854] 9fc0: 193a7662 00000001 00000003 0000017b be837fef 00000000 00000001 00000000
[ 1110.926566] 9fe0: be837db4 be837d98 00434f00 b6ef709c 20000010 00000003 00000000 00000000
[ 1110.936280]  esp_sdio_probe [esp8089] from sdio_bus_probe+0xbc/0x17c
[ 1110.944515]  sdio_bus_probe from really_probe+0xc4/0x298
[ 1110.950957]  really_probe from __driver_probe_device+0x84/0x19c
[ 1110.958428]  __driver_probe_device from driver_probe_device+0x30/0xdc
[ 1110.966409]  driver_probe_device from __driver_attach+0x94/0x104
[ 1110.973939]  __driver_attach from bus_for_each_dev+0x6c/0xb8
[ 1110.980647]  bus_for_each_dev from bus_add_driver+0x138/0x1cc
[ 1110.987410]  bus_add_driver from driver_register+0x7c/0x114
[ 1110.993987]  driver_register from esp_sdio_init+0x1c/0x30 [esp8089]
[ 1111.001929]  esp_sdio_init [esp8089] from do_one_initcall+0x48/0x248
[ 1111.009947]  do_one_initcall from do_init_module+0x48/0x1cc
[ 1111.016508]  do_init_module from load_module+0x1720/0x18b8
[ 1111.022985]  load_module from sys_finit_module+0xb8/0x104
[ 1111.029354]  sys_finit_module from ret_fast_syscall+0x0/0x44
[ 1111.035973] Exception stack(0xc2909fa8 to 0xc2909ff0)
[ 1111.041940] 9fa0:                   193a7662 00000001 00000003 004fee30 00000000 be837f4d
[ 1111.051468] 9fc0: 193a7662 00000001 00000003 0000017b be837fef 00000000 00000001 00000000
[ 1111.061032] 9fe0: be837db4 be837d98 00434f00 b6ef709c
[ 1111.067054] Code: eb001bee e2506000 0a00000a e3a03000 (e5864010) 
[ 1111.074700] ---[ end trace 0000000000000000 ]---
[ 1111.080440] note: insmod[106] exited with preempt_count 1
Segmentation fault
```

接下来就是软件层面的问题了，我们放到`驱动修改适配`章节来细说吧

## 驱动修改适配

我fork了一份 [@al117](https://github.com/al177/esp8089) 移植适配的 esp8089 驱动作为基础 [NINJAR-lite/esp8089_al117](https://github.com/ninjar-lite/esp8089_al177)，经过如下修改，确认不会再加载阶段崩溃了，但是还有问题，我们接着往下看。
```diff
diff --git a/esp_mac80211.c b/esp_mac80211.c
index 8b56c1a..b8ac767 100755
--- a/esp_mac80211.c
+++ b/esp_mac80211.c
@@ -790,6 +790,14 @@ static int esp_op_set_tim(struct ieee80211_hw *hw, struct ieee80211_sta *sta,
 }
 #endif
 
+static void esp_op_wake_tx_queue(struct ieee80211_hw *hw,
+                              struct ieee80211_txq *txq)
+{
+        struct esp_pub *epub = (struct esp_pub *) hw->priv;
+        if (epub)
+             ieee80211_queue_work(hw, &epub->tx_work);
+}
+
 #if (LINUX_VERSION_CODE >= KERNEL_VERSION(2, 6, 30))
 static int esp_op_set_key(struct ieee80211_hw *hw, enum set_key_cmd cmd,
                           struct ieee80211_vif *vif, struct ieee80211_sta *sta,
@@ -1936,6 +1944,7 @@ static const struct ieee80211_ops esp_mac80211_ops = {
         .prepare_multicast = esp_op_prepare_multicast,
 #endif
         .configure_filter = esp_op_configure_filter,
+        .wake_tx_queue = esp_op_wake_tx_queue,
         .set_key = esp_op_set_key,
         .update_tkip_key = esp_op_update_tkip_key,
         //.sched_scan_start = esp_op_sched_scan_start,
```

刚才提到的，虽然不会再加载阶段崩溃了，但是运行了一会儿之后，驱动崩溃了，日志如下：
```log
```

推测是因为供电不足问题，网卡中途掉了，那先来排查供电问题。

我在模组的电源输入端，并联的一个大电容，