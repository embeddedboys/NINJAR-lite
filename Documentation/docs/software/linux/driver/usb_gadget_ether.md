HamsterBear Linux 启用USB Gadget RNDIS
--------------------------------------

环境
----
- `Soc` - F1C200s
- `Kernel版本` - 主线 5.17.0


适配过程供参考

## kernel
修改 `arch/arm/boot/dts/suniv-f1c100s.dtsi`
在soc节点添加如下节点
```c
                usb_otg: usb@1c13000 {
                        compatible = "allwinner,suniv-musb"; // compatible reg等因soc而异
                        reg = <0x01c13000 0x0400>;
                        clocks = <&ccu CLK_BUS_OTG>;
                        resets = <&ccu RST_BUS_OTG>;
                        interrupts = <26>;
                        interrupt-names = "mc";
                        phys = <&usbphy 0>;
                        phy-names = "usb";
                        extcon = <&usbphy 0>;
                        allwinner,sram = <&otg_sram 1>;
                        status = "disabled";
                };

                usbphy: phy@1c13400 {
                        compatible = "allwinner,suniv-usb-phy";  // 同上
                        reg = <0x01c13400 0x10>;
                        reg-names = "phy_ctrl";
                        clocks = <&ccu CLK_USB_PHY0>;
                        clock-names = "usb0_phy";
                        resets = <&ccu RST_USB_PHY0>;
                        reset-names = "usb0_reset";
                        #phy-cells = <1>;
                        status = "disabled";
                };

```
对应如下两个驱动文件：
`drivers/phy/allwinner/phy-sun4i-usb.c`
`drivers/usb/musb/sunxi.c`

在 `arch/arm/boot/dts/suniv-f1c100s-licheepi-nano.dts` 中引用结点如下
```c
&otg_sram {
        status = "okay";
};

&usb_otg {
        dr_mode = "otg";
        status = "okay";
};

&usbphy {
        usb0_id_det-gpio = <&pio 4 2 GPIO_ACTIVE_HIGH>; /* PE2 */  // USB ID脚 需要根据查看板子原理图确定
        status = "okay";
};
```
--------
#### 配置kernel config
定位到 `Device drivers` --> `USB support`
![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326164339016-2023059160.png)

配置如上

然后选择 `USB Gadget` 编译进内核
![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326164535954-762361538.png)

`USB Gadget` 的配置
![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326164609659-412865758.png)

进入 `USB Gadget precomposed configurations`，配置如下
![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326164923291-1418100850.png)


选择了`USB Ethernet Gadget USB以太网`、
`USB Mass Storage Gadget USB大容量存储设备`、
`USB Serial Gadget USB虚拟串口`编译为模块

ok， 配置完之后重新编译内核+模块

- 拷贝内核到sd卡boot分区
- 拷贝驱动文件到文件系统`/lib/modules`下

驱动文件打包参考：
```shell
INSTALL_MOD_PATH=./tmp make modules_install
```
## rootfs

拷贝完成后连接到板子串口，执行modprobe加载驱动文件
```shell
modprobe g_ether
```

此时，不出意外的情况下（虚拟机会询问新设备连接位置），windows pc机这边会识别到一个`端口(COM 和 LPT)`

![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326171026667-1369829994.png)

为什么会是一个端口？，可参考如下回答
> 最初由MS论坛上的dlech发表
Linux rndis小工具函数的USB类为2，子类为2，与usbser.inf文件中的“USB \ Class_02＆SubClass_02”相匹配。这就是为什么对于某些人来说，他们的设备最初被检测为COM端口而不是RNDIS。
通过一些快速的Google搜索，似乎有些VM用户没有遇到此问题，可能是因为类/子类对在呈现给访客时被篡改了？

需要将驱动替换为 `Windows 10 signed RNDIS driver for USBNetwork`，可参考如下链接`1楼`的解决方案
[https://www.mobileread.com/forums/showthread.php?p=3283986](https://www.mobileread.com/forums/showthread.php?p=3283986)

如果你访问不了上面的链接，这里提供一份备用驱动文件链接
[kindle_rndis.inf_amd64-v1.0.0.1.zip](http://embeddedboys.space/uploads/kindle_rndis.inf_amd64-v1.0.0.1.zip)

顺便贴下怎么用吧
> 1. Download & Unzip attachment kindle_rndis.inf_amd64-v1.0.0.1.zip
> 2. R-click "5-runasadmin_register-CA-cer.cmd" and "Run as administrator"*
> 3. In Device Manager, expand "Ports (COM & LPT)", R-click "Serial USB device (COM3)" > Update Driver Software...
> 4. Browse for my computer for driver software > Select extract folder

替换完成之后会变成这个设备 `Kindle USB RNDIS Device`

![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326171237518-676065611.png)

然后回到板子这边
```shell
ifconfig usb0 up
ifconfig usb0 192.168.1.101
```

然后pc这边来到 控制面板 --> 网络和Internet --> 网络连接，亦可直接搜索`查看网络连接`
![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326174700565-940059786.png)

手动分配下ip地址为 `192.168.1.100`
>（这里因为我给usb共享了网络，所以此处ip为192.168.137.1，为网关地址）

![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326174927342-613451268.png)

然后在板子这边 ping 192.168.1.100 已经可以ping通了

#### 更多的玩法

##### 1. 共享pc网络给板子
![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326175145433-752413590.png)

pc中选中一个有效的网络连接， 然后右键属性 - 共享，勾选`允许其他网络用户通过此计算机的Internet连接来连接`，选择usb网卡，
（如果手动配置过ip，会提示ip信息覆盖）
然后回到板子，执行
```shell
udhcpc -i usb0
```
此时已经得到了动态分配的ip地址，`/etc/resolv.conf`中添加dns服务器`nameserver 114.114.114.114`，并尝试访问外网
![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220326175546091-561990888.png)

我想只用一根usb线就能调试板子，就像[`beaglebone black`](https://beagleboard.org/black)那样，真酷，我应该怎么做？

添加`/etc/init.d/S51usbifup`，增加`755`权限，脚本如下，
> (udhcpc可能会导致脚本阻塞，视情况可在启动后执行)

```shell
#!/bin/sh
#
# usbifup        Starts usb gadget rndis.
#

umask 077

start() {
        printf "Starting USB Gadget RNDIS: "
        modprobe g_ether
        ifconfig usb0 up
        udhcpc -i usb0
        echo "OK"
}
stop() {
        printf "Stopping USB Gadget RNDIS: "
        rmmod g_ether
        echo "OK"
}
restart() {
        stop
        start
}

case "$1" in
  start)
        start
        ;;
  stop)
        stop
        ;;
  restart|reload)
        restart
        ;;
  *)
        echo "Usage: $0 {start|stop|restart}"
        exit 1
esac

exit $?
```

这样，启动后便会自动加载驱动，pc机这边控制台执行
```shell
arp -a | findstr "137"
接口: 192.168.137.1 --- 0x5c
  192.168.137.115       36-bd-3d-9b-31-e0     静态
  192.168.137.255       ff-ff-ff-ff-ff-ff     静态
```

`192.168.137.115`就是自动分配给板子的ip

使用ssh登录到板子`ssh root@192.168.137.115`
> root用户登陆需要修改/etc/ssh/sshd_config，添加PermitRootLogin yes

![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220327145822217-592671812.png)


##### 2. 使用 `VNC` 连接到 `X` 桌面环境

#### 针对 `F1c200s` suniv系列的修改
需要修改适配 phy 和 musb 的代码，其实全志官方4.15版本的内核早就修改过了，但不知道为什么没有合并到主线里。
```diff
diff --git a/drivers/phy/allwinner/phy-sun4i-usb.c b/drivers/phy/allwinner/phy-sun4i-usb.c
index 788dd5c..44dcffa 100644
--- a/drivers/phy/allwinner/phy-sun4i-usb.c
+++ b/drivers/phy/allwinner/phy-sun4i-usb.c
@@ -99,6 +99,7 @@
 #define POLL_TIME                      msecs_to_jiffies(250)

 enum sun4i_usb_phy_type {
+       suniv_phy,
        sun4i_a10_phy,
        sun6i_a31_phy,
        sun8i_a33_phy,
@@ -857,6 +858,14 @@ static int sun4i_usb_phy_probe(struct platform_device *pdev)
        return 0;
 }

+static const struct sun4i_usb_phy_cfg suniv_cfg = {
+       .num_phys = 1,
+       .type = suniv_phy,
+       .disc_thresh = 3,
+       .phyctl_offset = REG_PHYCTL_A10,
+       .dedicated_clocks = true,
+};
+
 static const struct sun4i_usb_phy_cfg sun4i_a10_cfg = {
        .num_phys = 3,
        .type = sun4i_a10_phy,
@@ -970,6 +979,7 @@ static const struct sun4i_usb_phy_cfg sun50i_h6_cfg = {
 };

 static const struct of_device_id sun4i_usb_phy_of_match[] = {
+       { .compatible = "allwinner,suniv-usb-phy", .data = &suniv_cfg },
        { .compatible = "allwinner,sun4i-a10-usb-phy", .data = &sun4i_a10_cfg },
        { .compatible = "allwinner,sun5i-a13-usb-phy", .data = &sun5i_a13_cfg },
        { .compatible = "allwinner,sun6i-a31-usb-phy", .data = &sun6i_a31_cfg },
diff --git a/drivers/usb/musb/sunxi.c b/drivers/usb/musb/sunxi.c
index 961c858..85542d1 100644
--- a/drivers/usb/musb/sunxi.c
+++ b/drivers/usb/musb/sunxi.c
@@ -722,14 +722,16 @@ static int sunxi_musb_probe(struct platform_device *pdev)
        INIT_WORK(&glue->work, sunxi_musb_work);
        glue->host_nb.notifier_call = sunxi_musb_host_notifier;

-       if (of_device_is_compatible(np, "allwinner,sun4i-a10-musb"))
+       if (of_device_is_compatible(np, "allwinner,sun4i-a10-musb") ||
+           of_device_is_compatible(np, "allwinner,suniv-musb"))
                set_bit(SUNXI_MUSB_FL_HAS_SRAM, &glue->flags);

        if (of_device_is_compatible(np, "allwinner,sun6i-a31-musb"))
                set_bit(SUNXI_MUSB_FL_HAS_RESET, &glue->flags);

        if (of_device_is_compatible(np, "allwinner,sun8i-a33-musb") ||
-           of_device_is_compatible(np, "allwinner,sun8i-h3-musb")) {
+           of_device_is_compatible(np, "allwinner,sun8i-h3-musb") ||
+           of_device_is_compatible(np, "allwinner,suniv-musb")) {
                set_bit(SUNXI_MUSB_FL_HAS_RESET, &glue->flags);
                set_bit(SUNXI_MUSB_FL_NO_CONFIGDATA, &glue->flags);
        }
@@ -822,6 +824,7 @@ static int sunxi_musb_remove(struct platform_device *pdev)
 }

 static const struct of_device_id sunxi_musb_match[] = {
+       { .compatible = "allwinner,suniv-musb", },
        { .compatible = "allwinner,sun4i-a10-musb", },
        { .compatible = "allwinner,sun6i-a31-musb", },
        { .compatible = "allwinner,sun8i-a33-musb", },


```

#### 参考资料
[Windows 10 signed RNDIS driver for USBNetwork](https://www.mobileread.com/forums/showthread.php?p=3283986)
[测试测试 g_serial / g_ether USB Gadget (RNDIS)](https://whycan.com/viewtopic.php?id=2401)
[小白自制Linux开发板 七. USB驱动配置](https://www.cnblogs.com/twzy/p/15243838.html)