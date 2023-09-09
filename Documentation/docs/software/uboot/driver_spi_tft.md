<h1 align="center">uboot驱动实战 为SPI TFT添加显示支持</h1>

Table of contents
----------------------

- [名词解释](#名词解释)
- [写在前面](#写在前面)
- [实现](#实现)
- [优化思路](#优化思路)
- [更多](#更多)
- [参考资料](#参考资料)

## 名词解释
| 名词 | 解释 |
| --- | --- |
| TFT | TFT（Thin Film Transistor），是薄膜晶体管的缩写。 |
| SPI | Serial Peripheral Interface，是串行外围设备接口，是一种高速，全双工，同步的通信总线 |

## 写在前面
本次实验平台基于如下环境

| 名称 | 版本 |
| --- | --- |
| SoC | Allwinner Suniv F1C100s |
| u-boot | v2023.07 |
| 屏幕 | 基于 SSD1327 的16灰阶 OLED |

## 思路

简单说下实现思路

还是参考现有驱动的方法，经过一番查阅后，`drivers/video/seps525.c`这个驱动是最符合我们现在的场景的。

都是通过SPI通讯，类似SPI-TFT，有gpio操作。

所以在此驱动基础上，实现到我们器件的操作、转换、传输接口

### uboot Video 框架
[在这里放驱动框图]

## 实现
### 1. 定义数据结构

底层显示接口
```c
struct ssd1327_operations {
    int (*init_display)(struct ssd1327_priv *priv);		// 初始化
    int (*reset)(struct ssd1327_priv *priv);			// 复位
    int (*clear)(struct ssd1327_priv *priv, int clear);	// 清屏
    int (*blank)(struct ssd1327_priv *priv, bool on);	// 整屏操作
    int (*sleep)(struct ssd1327_priv *priv, bool on);	// 休眠
    int (*set_var)(struct ssd1327_priv *priv);		// 控制旋转等额外设置
    int (*set_contrast)(struct ssd1327_priv *priv, u8 contrast);	// 设置对比度
    int (*set_addr_win)(struct ssd1327_priv *priv, int xs, int ys, int xe, int ye);	// 设置绘制窗口
    int (*set_cursor)(struct ssd1327_priv *priv, int x, int y);	//设置起点
};
```

有关屏幕面板参数的
```c
struct ssd1327_display {
    u32                     xres;	// 水平分辨率
    u32                     yres;	// 垂直分辨率
    u32                     bpp;	// 像素深度
    u32                     rotate;	// 旋转角度
};
```

显示驱动主体
```c
struct ssd1327_priv {
    struct udevice          *dev;
    struct spi_slave        *spi;
    u8                      *buf;		// 用于框架本身的buffer
    struct {
        void *buf;
        size_t len;
    } txbuf;	// 用来发送到spi的buffer
    struct {
        struct gpio_desc *reset;
        struct gpio_desc *dc;
        struct gpio_desc *cs;
        struct gpio_desc *blk;
    } gpio;		// 需要请求的gpio
    
    /* device specific */
    const struct ssd1327_operations  *tftops;
    struct ssd1327_display           *display;
    
    u8              contrast;
};
```

### 2. 编写 uboot driver 框架
```c
U_BOOT_DRIVER(ssd1327) = {
    .name       = DRV_NAME "_video",
    .id         = UCLASS_VIDEO,
    .ops        = &ssd1327_ops,
    .bind       = ssd1327_bind,
    .probe      = ssd1327_probe,
    .of_match   = ssd1327_ids,
    .of_to_plat = ssd1327_ofdata_to_platdata,
    .plat_auto  = sizeof(struct video_uc_plat),
    .priv_auto  = sizeof(struct ssd1327_priv),
};
```

## 基准测试

### 1. SPI 速率
SPI速率不能太高，否则会超过驱动IC的时序范围

| 速率 | 耗时 |
| --- | --- |
| 8 MHz | |
| 12 MHz | |
| 16 MHz | |

### 2. DMA 传输
| 状态 | 耗时 |
| --- | --- |
| 关闭 | |
| 开启 | |

### 3. 颜色转换算法
| 方式 | 耗时 |
| --- | --- |
| 移位法 | |

### 4. video_sync 传输优化
| 方式 | 耗时 |
| --- | --- |
| Buffered |  |
| Legacy |  |

### 5. 去除转换算法，video层直接拿灰阶数据

## 更多
### 器件细节

## 参考资料
