主线linux f1c200s fbtft ssd1306 适配记录

menuconfig中开启staging drivers下small tft菜单中 fb ssd1306, 选择*编译进内核。

![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220309041225866-1932444512.png)


修改pio节点如下，添加 spi1 复用引脚。
```c
pio: pinctrl@1c20800 {
                        compatible = "allwinner,suniv-pinctrl";
                        reg = <0x01c20800 0x400>;
                        interrupts = <38>, <39>, <40>;
                        clocks = <&ccu CLK_BUS_PIO>, <&osc24M>, <&osc32k>;
                        clock-names = "apb", "hosc", "losc";
                        gpio-controller;
                        interrupt-controller;
                        #interrupt-cells = <3>;
                        #gpio-cells = <3>;

                        spi0_pins_a: spi0-pins-pc {
                                pins = "PC0", "PC1", "PC2", "PC3";
                                function = "spi0";
                        };

                        spi1_pins_a: spi1-pins-pa {
                                pins = "PA0", "PA1", "PA2", "PA3";
                                function = "spi1";
                        };

                        lcd_rgb666_pins: lcd-rgb666-pins {
                                pins = "PD0", "PD1", "PD2", "PD3", "PD4",
                                       "PD5", "PD6", "PD7", "PD8", "PD9",
                                       "PD10", "PD11", "PD12", "PD13", "PD14",
                                       "PD15", "PD16", "PD17", "PD18", "PD19",
                                       "PD20", "PD21";
                                function = "lcd";
                        };

                        uart0_pins_a: uart-pins-pe {
                                pins = "PE0", "PE1";
                                function = "uart0";
                        };

                        mmc0_pins: mmc0-pins {
                                pins = "PF0", "PF1", "PF2", "PF3", "PF4", "PF5";
                                function = "mmc0";
                        };
                };
```

修改spi1节点如下
```c
&spi1 {
        pinctrl-names = "default";
        pinctrl-0 = <&spi1_pins_a>;
        status = "okay";
        spi-max-frequency = <50000000>;
        ssd1306: ssd1306@0 {
                #address-cells = <1>;
                #size-cells = <1>;
                compatible = "solomon,ssd1306";
                reg = <0>;
                spi-max-frequency = <50000000>;
                buswidth = <8>;
                rotate = <0>;
                fps = <30>;
                spi-cpol;
                spi-cpha;
                reset-gpios = <&pio 4 11 GPIO_ACTIVE_HIGH>;
                dc-gpios = <&pio 4 12 GPIO_ACTIVE_LOW>;
                debug = <1>;
        };
};
```

```shell
[    1.028749] fbtft_of_value: buswidth = 8
[    1.032829] fbtft_of_value: debug = 1
[    1.036525] fbtft_of_value: rotate = 0
[    1.040289] fbtft_of_value: fps = 30
[    1.044381] fb_ssd1306 spi1.0: fbtft_request_one_gpio: 'reset-gpios' = GPIO139
[    1.051982] fb_ssd1306 spi1.0: fbtft_request_one_gpio: 'dc-gpios' = GPIO140
[    1.194658] Console: switching to colour frame buffer device 16x8
[    1.202163] graphics fb0: fb_ssd1306 frame buffer, 128x64, 16 KiB video memory, 4 KiB buffer memory, fps=33, spi1.0 at 50 MHz
```

最后，上张效果图
---------------
![image](https://img2022.cnblogs.com/blog/2605173/202203/2605173-20220316075805258-1643076819.jpg)