HamsterBear Linux ST7789V FBTFT驱动适配
------------------------------

* `平台` - F1C200s
* `Linux版本` - 5.18
* `TFT屏` - 1.69寸IPS高清ST7789V

#### 修改设备树，在spi0节点下添加
```c
&spi0 {
        pinctrl-names = "default";
        pinctrl-0 = <&spi0_pd_pins>;
        status = "okay";
        spi-max-frequency = <50000000>;

        st7789v: st7789v@0 {
                #address-cells = <1>;
                #size-cells = <1>;
                compatible = "sitronix,st7789v";
                reg = <0>;
                spi-max-frequency = <50000000>;
                rgb;
                buswidth = <8>;
                rotate = <90>;
                fps = <60>;
                spi-cpol;
                spi-cpha;
                reset-gpios = <&pio 4 10 GPIO_ACTIVE_HIGH>;     /* PE10 */
                dc-gpios = <&pio 2 3 GPIO_ACTIVE_LOW>;          /* PC3 */
                debug = <1>;
        };
};
```

#### 确定内核config打开
需要注意的是，fbtft的驱动只有在内核framebuffer相关驱动开启后才会出现，所以要先开启fb驱动支持
Device Drivers -> Graphics support -> Frame buffer Devices  --->，编译进内核
![image](https://img2022.cnblogs.com/blog/2605173/202205/2605173-20220510014807534-1679025608.png)

可根据需要将控制台映射到fb功能开启
![image](https://img2022.cnblogs.com/blog/2605173/202205/2605173-20220510014927313-1363991664.png)

这里我同时选择了16色的Tux 启动logo
![image](https://img2022.cnblogs.com/blog/2605173/202205/2605173-20220510014958052-1354061996.png)


然后寻找fbtft驱动，路径如下：
Device Drivers -> Staging drivers -> Support for small TFT LCD display modules  --->，编译进内核
![image](https://img2022.cnblogs.com/blog/2605173/202205/2605173-20220510014548262-1638003774.png)


#### 修改`drivers/staging/fbtft/fb_st7789v.c`，如下几处
1. 修改init函数，该初始化序列参考自厂家
```c
static int init_display(struct fbtft_par *par)
{
        int rc;

        par->fbtftops.reset(par);
		mdelay(100);
        rc = init_tearing_effect_line(par);
        if (rc)
                return rc;

        write_reg(par, 0x11); //Sleep out
        mdelay(120);     //Delay 120ms

        //************* Start Initial Sequence **********//
        write_reg(par, 0x36, 0x00);

        write_reg(par, 0x3A, 0x05);

        write_reg(par, 0xB2,0x0C,0x0C,0x00,0x33,0x33);

        write_reg(par, 0xB7,0x35);

        /* VCOM  = 1.35V */
        write_reg(par, 0xBB, 0x32);

        write_reg(par, 0xC2, 0x01);

        /* GVDD = 4.8V */
        write_reg(par, 0xC3,0x15);

        /* VDX = 0V */
        write_reg(par, 0xC4, 0x20);

        /* FPS = 60Hz */
        write_reg(par, 0xC6,0x0F);

        write_reg(par, 0xD0, 0xA4, 0xA1);

        write_reg(par, 0xE0,0xD0,0x08,0x0E,0x09,0x09,0x05,0x31,0x33,0x48,0x17,0x14,0x15,0x31,0x34);

        write_reg(par, 0xE1,0xD0,0x08,0x0E,0x09,0x09,0x15,0x31,0x33,0x48,0x17,0x14,0x15,0x31,0x34);

        write_reg(par, MIPI_DCS_ENTER_INVERT_MODE);

        write_reg(par, MIPI_DCS_SET_DISPLAY_ON);

        return 0;
}
```
2， 修改display参数
```c
static struct fbtft_display display = {
        .regwidth = 8,
        .width = 240,
        .height = 280,	/* 根据你的屏幕分辨率确定 */
        .gamma_num = 2,
        .gamma_len = 14,
        .gamma = HSD20_IPS_GAMMA,
        .fbtftops = {
                .init_display = init_display,
                .write_vmem = write_vmem,
                .set_var = set_var,
                .set_gamma = set_gamma,
                .blank = blank,
        },
};
```

#### 修改`drivers/staging/fbtft/fbtft-core.c`，如下几处
1. 添加头文件
```c
#include <linux/of.h>
#include <linux/of_gpio.h>
```

2. 修改`fbtft_request_one_gpio`函数
```c
static int fbtft_request_one_gpio(struct fbtft_par *par,
                                  const char *name, int index,
                                  struct gpio_desc **gpiop)
{
        int ret, gpio;
        struct device *dev = par->info->device;
        struct device_node *np = dev->of_node;
        enum of_gpio_flags flags;

        /* Get GPIO from device tree */
        gpio = of_get_named_gpio_flags(np, name, index, &flags);
        if (gpio < 0) {
                return 0;
        }


        ret = devm_gpio_request_one(dev, gpio,
                        (flags & OF_GPIO_ACTIVE_LOW)?GPIOF_OUT_INIT_LOW:GPIOF_OUT_INIT_HIGH,
                        dev->driver->name);
        if (ret) {
                dev_err(dev, "Failed to request %s GPIO%d\n", name, gpio);
                return -ENODEV;
        }

        *gpiop = gpio_to_desc(gpio);

        fbtft_par_dbg(DEBUG_REQUEST_GPIOS, par, "%s: '%s' GPIO%d\n",
                      __func__, name, gpio);

        return ret;
}
```

3. 修改`fbtft_request_gpios`函数
```c
static int fbtft_request_gpios(struct fbtft_par *par)
{
        int i;
        int ret;

        ret = fbtft_request_one_gpio(par, "reset-gpios", 0, &par->gpio.reset);
        if (ret)
                return ret;
        ret = fbtft_request_one_gpio(par, "dc-gpios", 0, &par->gpio.dc);
        if (ret)
                return ret;
        ret = fbtft_request_one_gpio(par, "rd-gpios", 0, &par->gpio.rd);
        if (ret)
                return ret;
        ret = fbtft_request_one_gpio(par, "wr-gpios", 0, &par->gpio.wr);
        if (ret)
                return ret;
        ret = fbtft_request_one_gpio(par, "cs-gpios", 0, &par->gpio.cs);
        if (ret)
                return ret;
        ret = fbtft_request_one_gpio(par, "latch-gpios", 0, &par->gpio.latch);
        if (ret)
                return ret;
        for (i = 0; i < 16; i++) {
                ret = fbtft_request_one_gpio(par, "db-gpios", i,
                                             &par->gpio.db[i]);
                if (ret)
                        return ret;
                ret = fbtft_request_one_gpio(par, "led-gpios", i,
                                             &par->gpio.led[i]);
                if (ret)
                        return ret;
                ret = fbtft_request_one_gpio(par, "aux-gpios", i,
                                             &par->gpio.aux[i]);
                if (ret)
                        return ret;
        }

        return 0;
}
```

4. 修改`fbtft_set_win_addr`函数
因为st7789v支持到240x320的分辨率，但是我这块屏是240*280，所以x start和x end都有偏移,
这个偏移值也是我从厂家源码中参考的。
```c
static void fbtft_set_addr_win(struct fbtft_par *par, int xs, int ys, int xe,
                               int ye)
{
        xs = xs + 20;
        xe = xe + 20;
        write_reg(par, MIPI_DCS_SET_COLUMN_ADDRESS,
                  (xs >> 8) & 0xFF, xs & 0xFF, (xe >> 8) & 0xFF, xe & 0xFF);

        write_reg(par, MIPI_DCS_SET_PAGE_ADDRESS,
                  (ys >> 8) & 0xFF, ys & 0xFF, (ye >> 8) & 0xFF, ye & 0xFF);

        write_reg(par, MIPI_DCS_WRITE_MEMORY_START);
}
```

5. 修改`fbtft_reset`函数
st7789v是low level reset
```c
static void fbtft_reset(struct fbtft_par *par)
{
        if (!par->gpio.reset)
                return;

        fbtft_par_dbg(DEBUG_RESET, par, "%s()\n", __func__);

        gpiod_set_value_cansleep(par->gpio.reset, 1);
        usleep_range(20, 40);
        gpiod_set_value_cansleep(par->gpio.reset, 0);   /* Low level reset */
        msleep(120);

        gpiod_set_value_cansleep(par->gpio.reset, 1);  /* Activate chip */
}
```

#### 最后附上两个文件的直观patch
```diff
```

```diff
```