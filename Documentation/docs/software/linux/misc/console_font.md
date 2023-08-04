<h1 align="center">Linux --- 修改控制台字体consolefont</h1>

## 用户层面

对于debian like 的系统，可到如下路径查找console font
```shell
$ cd /usr/share/consolefonts/
$ ls
Arabic-Fixed15.psf.gz             CyrSlav-VGA32x16.psf.gz               FullGreek-VGA8.psf.gz            Lat15-VGA16.psf.gz              Thai-Fixed14.psf.gz
Arabic-Fixed16.psf.gz             CyrSlav-VGA8.psf.gz                   Georgian-Fixed13.psf.gz          Lat15-VGA28x16.psf.gz           Thai-Fixed15.psf.gz
	....
```
### 尝试切换终端字体

`CTRL +  ALT + F3` 切换至 tty3
```shell
$ setfont /usr/share/consolefonts/Arabic-Fixed15.psf.gz 
```

### 相关文件及指令
`/etc/`
```shell
$ setupcon --print-commands-only
setfont '-C' '/dev/tty1' '/usr/share/consolefonts/Uni2-Fixed16.psf.gz' 
setfont '-C' '/dev/tty2' '/usr/share/consolefonts/Uni2-Fixed16.psf.gz' 
setfont '-C' '/dev/tty3' '/usr/share/consolefonts/Uni2-Fixed16.psf.gz' 
setfont '-C' '/dev/tty4' '/usr/share/consolefonts/Uni2-Fixed16.psf.gz' 
setfont '-C' '/dev/tty5' '/usr/share/consolefonts/Uni2-Fixed16.psf.gz' 
setfont '-C' '/dev/tty6' '/usr/share/consolefonts/Uni2-Fixed16.psf.gz' 
	......
```

## 内核层面

### config

make menuconfig

定位到 > Library routines， 选择 CONFIG_FONTS=y

还可继续选择编译进内核的字体

![image](https://img2023.cnblogs.com/blog/2605173/202307/2605173-20230727180930202-1889010363.png)

### 字体匹配逻辑

找到 `lib/fonts/fonts.c` 

存放所有字体的数组
```c
static const struct font_desc *fonts[] = {
	&font_vga_8x8,
	...
}
```

导出了两个函数
```c
EXPORT_SYMBOL(find_font);
EXPORT_SYMBOL(get_default_font);
```
对于函数`get_default_font` ，根据屏幕以及字体尺寸，使用得分方式选择合适字体。

这两个函数都会在 `fbcon.c` 中使用，`find_font`是用来指定选择字体的接口，也在`fbcon.c`中被调用