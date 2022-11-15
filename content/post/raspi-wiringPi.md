+++
title = "树莓派4B-WiringPi库"
draft = false
date = "2022-11-06T23:47:32+08:00"
menu = "main"
Categories = ["RasPi", "wiringPi", "I2C", "Guide"]
Description = "WiringPi是一个用C语言编写的、基于PIN的GPIO访问库。提供了GPIO、I2C、SPI、UART和PWM等库，对于树莓派的GPIO编程相当方便。"
Tags = ["RasPi", "guide", "wiringPi", "I2C", "gpio ", "开源", "guide", "raspberrypi"]
+++

## WiringPi

### 官网

[wiringpi.com](http://wiringpi.com/)

### GitHub WiringPi

[github.com/WiringPi](https://github.com/WiringPi)

目前 https://github.com/WiringPi 仓库支持的版本有

* C/C++
* Python
* PHP
* Node
* Ruby
* Perl

### WiringPi-Go

[WiringPi-Go](https://github.com/CarlosRA97/wpi)

Golang wrapped version of Gordon's Arduino-like WiringPi for the Raspberry Pi

### WiringPi库简介

WiringPi是一个用C语言编写的、基于PIN的GPIO访问库，提供了GPIO、I2C、SPI、UART和PWM等库，对于树莓派的GPIO编程相当方便。目前可在基于BCM2835、BCM2836和BCM2837的树莓派中使用。其他详细介绍请参看wiringPi官网介绍。

#### WiringPi-C 安装

截止文章发布日（2022-11-06） WiringPi  Releases Version 2.61-1  
[https://github.com/WiringPi/WiringPi/releases/tag/2.61-1](https://github.com/WiringPi/WiringPi/releases/tag/2.61-1)


方法1:  **deb方式**

```shell
cd ~/work

## 请按系统选择格式的deb包。本人系统为 64位 版本。
wget  https://github.com/WiringPi/WiringPi/releases/download/2.61-1/wiringpi-2.61-1-arm64.deb

dpkg -i wiringpi-2.61-1-arm64.deb
```

方法2:  **源码方式**

```shell
cd ~/work

wget -O wiringpi-2.61-1.tar.gz  https://github.com/WiringPi/WiringPi/archive/refs/tags/2.61-1.tar.gz

tar zxfv  wiringpi-2.61-1.tar.gz 
cd wiringpi-2.61-1
./build
```


安装成功后，输入命令：gpio -v，可以查看WiringPi的版本

![gpio -v](https://hks3.7y2.org/myimg02/images/202211062303847.png)

其他开放的 `command`  `gpio -h`

![command](https://hks3.7y2.org/myimg02/images/202211062308664.png)

对接后的OLED显示器，SSD1306 驱动

![](https://hks3.7y2.org/myimg02/images/202211062340040.png)



