### linux用户空间控制lcd framebuffer控制台光标闪烁行为

定位到`/sys/class/graphics/fbcon`
有如下文件
```shell
#/sys/class/graphics/fbcon
cursor_blink  rotate        subsystem/
power/         rotate_all    uevent
```

默认情况下光标闪烁，文件cursor_blink值为1，向其中写0关闭光标闪烁
```shell
# 开启
echo 1 > /sys/class/graphics/fbcon/cursor_blink
# 关闭
echo 0 > /sys/class/graphics/fbcon/cursor_blink
```
