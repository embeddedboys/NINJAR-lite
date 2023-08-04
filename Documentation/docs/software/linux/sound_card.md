

#### 内核config
确定已经打开对应驱动，路径如下
```c
Device Drivers -> 
<*>Sound Card Support -> 
<*>   Advanced Linux Sound Architecture ->
[*]   USB sound devices -> 
<*>   USB Audio/MIDI driver
```

或直接搜索`CONFIG_SND_USB_AUDIO`
![image](https://img2022.cnblogs.com/blog/2605173/202205/2605173-20220510015630183-1134674565.png)

#### 安装如下软件包
```shell
apt install alsa-utils mplayer
```

#### 接入USB声卡时log
```shell
root@main:~# [ 1379.028364] usb 1-1.2: new full-speed USB device number 6 using musb-hdrc
[ 1379.304380] input: C-Media Electronics Inc. USB Audio Device as /devices/platform/soc/1c13000.usb/musb-hdrc.1.auto/usb1/1-1/1-1.2/1-1.2:1.3/0003:0D8C:0014.0002/input/input2
[ 1379.389156] hid-generic 0003:0D8C:0014.0002: input: USB HID v1.00 Device [C-Media Electronics Inc. USB Audio Device] on usb-musb-hdrc.1.auto-1.2/input3
```

#### 查看系统内声卡设备
```shell
root@main:~# cat /proc/asound/cards
 0 [sun4icodec     ]: sun4i-codec - sun4i-codec
                      sun4i-codec
 1 [Device         ]: USB-Audio - USB Audio Device
                      C-Media Electronics Inc. USB Audio Device at usb-musb-hdrc.1.auto-1.2, full spe
```

#### 修改缺省配置文件
`/etc/asound.conf`, 1 代表声卡id
```shell
defaults.ctl.card 1
defaults.pcm.card 1
defaults.timer.card 1
```

#### 测试音频输出
plughw:1,0   1为声卡id 0为次设备号
测试音频:
Tell It to My Heart (Instrumental)
Franck Choppin

```shell
root@main:~# aplay -D plughw:1,0 ./Tell_it_to_my_heart_fc.wav
Playing WAVE './Tell_it_to_my_heart_fc.wav' : Signed 16 bit Little Endian, Rate 44100 Hz, Stereo
```

#### 测试视频音频输出
```shell
root@main:~# mplayer -i test.mp4
```

#### 调整音量
```shell
root@main:~# alsamixer
┌────────────────────────────── AlsaMixer v1.2.4 ──────────────────────────────┐
│ Card: USB Audio Device                               F1:  Help               │
│ Chip: USB Mixer                                      F2:  System information │
│ View: F3:[Playback] F4: Capture  F5: All             F6:  Select sound card  │
│ Item: Speaker [dB gain: -36.00, -36.00]              Esc: Exit               │
│                                                                              │
│                  ┌──┐               ┌──┐                                     │
│                  │  │               │  │                                     │
│                  │  │               │  │                                     │
│                  │  │               │  │                                     │
│                  │  │               │  │                                     │
│                  │  │               │  │                                     │
│                  │  │               │▒▒│                                     │
│                  │  │               │▒▒│                                     │
│                  │  │               │▒▒│                                     │
│                  │▒▒│               │▒▒│                                     │
│                  │▒▒│               │▒▒│                                     │
│                  │▒▒│               │▒▒│                                     │
│                  ├──┤               ├──┤               ┌──┐                  │
│                  │OO│               │MM│               │OO│                  │
│                  └──┘               └──┘               └──┘                  │
│                 25<>25               56                                      │
│          <     Speaker      >       Mic         Auto Gain Control            │
└──────────────────────────────────────────────────────────────────────────────┘

```

#### X应用音频输出，暂未实验