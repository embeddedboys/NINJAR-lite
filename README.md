[![(Script) Discord Activity Badge](https://badgen.net/badge/Discord%20User/Offline?color=545454&labelColor=434343&icon=discord)](https://github.com/embeddedboys/NINJAR-lite)

<h1 align="center">
    <img src="assets/048-boy-next.png" width="5%" alt="embeddedboys logo" />
    <span>项目开发中。。。</span>
    
</h1>

![badge](https://img.shields.io/github/stars/embeddedboys/NINJAR-lite)
![badge](https://img.shields.io/github/repo-size/embeddedboys/NINJAR-lite)
![badge](https://img.shields.io/github/last-commit/iotahydrae/NINJAR-lite/main)
![badge](https://img.shields.io/github/commit-activity/t/embeddedboys/NINJAR-lite)
![badge](https://img.shields.io/github/license/embeddedboys/NINJAR-lite)
[![build docs](https://github.com/embeddedboys/NINJAR-lite/actions/workflows/blank.yml/badge.svg?branch=main&event=push)](https://github.com/embeddedboys/NINJAR-lite/actions/workflows/blank.yml)
[![pages-build-deployment](https://github.com/embeddedboys/NINJAR-lite/actions/workflows/pages/pages-build-deployment/badge.svg?branch=main)](https://github.com/embeddedboys/NINJAR-lite/actions/workflows/pages/pages-build-deployment)

`项目官网` : [`https://embeddedboys.github.io/NINJAR-lite`](https://embeddedboys.github.io/NINJAR-lite)

`文档链接` : [`https://embeddedboys.github.io/NINJAR-lite`](https://embeddedboys.github.io/NINJAR-lite)

`项目仓库` : [`https://github.com/embeddedboys/NINJAR-lite`](https://github.com/emmbeddedboys/NINJAR-lite)

</br>

> <h1 align="center">为自己制作一个微型 Linux 控制台</h1>

<!-- 这里可以放项目的预览图 -->
<div align="center">
<img src="./assets/product_preview.png" width="900">
</div>

<div align="center">
<h2><a href="https://github.com/embeddedboys/NINJAR-lite">预定</a></h2>
</div>

<div align="left" style="clear:both;">
<h2>介绍</h2>
</div>

<!-- 有关项目的简短介绍 -->
> <h3 align="center" ><strong>NINJAR (NINJAR Is Not Just A Reader) 作为阅读器，但不止于此</strong></h3> 

<h3 align="center">隆重的向大家介绍 NINJAR-lite</h3>

<div style="font-size:16px;">
&nbsp&nbsp&nbsp&nbspNINJAR-lite 是 NINJAR 项目的成本缩减（精简）版本，本项目致力于打造一款百元以内（69？ 79？）的多功能嵌入式 Linux 学习、开发、创作设备。对于这个设备的形态，我们其实没有准确的定义，事实上，这取决于你的想法。正如演示中的那样，通过外接无线键盘，它可以当作一台迷你终端来使用。通过开源的的设计资料，这个设备的形态可以被重新定义，这得益于模块化的设计
</div>

> <div style="font-size:14px;" align="center">这个项目正处于开发阶段，我们正在尝试加入更多的创客功能</div>

## <h2 align="center">设计</h2>
<div align="center">
<img src="./assets/right_view.png" height=512>
</div>

> <div style="font-size:20px; text-align:center;">NINJAR-lite 提供了许多功能在仅仅35x43mm的尺寸上</div>

<!--  -->
<div align="right">
</br>
<h2>主板</h2>
</div>

<div style="display:flex;">
<div style="width:60%; margin-right:40px;">
<img src="./assets/NINJAR_Resources.jpg" width=400/>
</div>

<div style="float:right;">
<ul style="font-size:14px;">
<li> USB OTG </li>
<li> 用户按键 </li>
<li> 450mAh 锂电池</li>
<li> 滚轮旋转编码器 </li>
<li> ESP8089 WIFI模组</li>
<li> TP4056 充电管理IC</li>
<li> AHT20 温湿度传感器 </li>
<li> 最大256MB SPI NAND </li>
<li> 支持标准8Pin 3-wire SPI 显示屏 </li>
</ul>
</div>

</div>
<!--  -->


<!--  -->
<div align="left">
</br>
<h2>核心板</h2>
</div>

<div style="float:left; margin-right:200px;">
<ul style="font-size:14px;">
<li> F1C100s/F1C200s ARM9 @900MHz </li>
<li> SIP 32/64MB DDR1 </li>
<li> IO 管脚全部引出 </li>
<li> 分立DC-DC供电 </li>
<li> 5V 输入 </li>
</ul>
</div>

<div style="display:flex;">
<div style="width:100%;">
<img src="./assets/Core_Resources.jpg" width=500 />
</div>


</div>
<!--  -->

<!--  -->
<div align="right">
</br>
<h2>屏幕</h2>
</div>

<div style="display:flex;">
<div style="width:60%; margin-right:40px;">
<img src="./assets/screen.png" width=300 />
</div>

<div style="float:right;">
<div style="font-size:20px; text-align:center;"></div>
<ul style="font-size:14px;">
<li> OLED </li>
<li> 16灰阶 </li>
<li> 高对比度 </li>
<li> SSD1327主控 </li>
<li> uboot、linux驱动支持 </li>
</ul>
</div>

</div>
<!--  -->

<!-- <div align="center">
<img src="./assets/strcuture.gif" height="512">
</div> -->

## <h2 align="center">实机演示</h2>
<!-- 快速简短的GIF玩法展示 -->

<div align="center">
<img src="./assets/first_preview.gif" width=1024>
<img src="./assets/second_preview.gif" width=1024>
</div>

<!-- 简短的设计过程 -->

> <p align="center">器件以实物为准</p>

## <h2 align="center">技术规格</h2>

<!-- 有关设备资源的表格 -->

| 组件       | 型号                          |
|------------|-------------------------------|
| 系统       | linux、Debian 12             |
| CPU        | Allwinner F1C100s/F1C200s ARM9 @900MHz |
| 内存       | 32/64MB DDR1 256MHz              |
| 存储       | 128/256MB SPI-Nand Flash              |
| 屏幕       | 1.28英寸 16灰阶 OLED 128x128         |
| 网络       | ESP8089/RTL8723BS 2.4G WiFi AP (选配)                 |
| 定位       | GP-02 GPS/BDS 模组 （选配）           |
| 控制       | 拨轮编码器、按键               |
| 传感器     | AHT20 温湿度传感器                         |
| 电池       | 450mAh                        |
| 充放电 | TP4056 + MOSFET                    |
| 接口 | 2x5 Pin Header with I2C1,I2S, SH1.0mm 4Pin I2C0, USB 2.0 OTG |

## <h2 align="center">许可证</h2>


此项目发布于`MIT`许可证下，更多详细信息，请查看`关于->许可证`

## <h2 align="center">致谢</h2>
<!-- 对该项目做出贡献的组织或个人 -->


<p align="center">
玩得开心！</br></br>
干杯,</br>
zheng </br>

</p>

<h2 align="center">
    <img src="assets/048-boy-next.png" width="10%" alt="embeddedboys logo" /> </br>
    <img alt="Static Badge" src="https://img.shields.io/badge/🍺-embeddedboys-blue">
</h2>
<h2 align="center">
    <a href="https://embeddedboys.github.io/">embeddedboys</a> 献上
</h2>
