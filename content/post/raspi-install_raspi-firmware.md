+++
title = "树莓派4B-安装raspi-firmware"
draft = false
date = "2022-11-15T18:32:32+08:00"
menu = "main"
Categories = ["Guide", "solutions", "RasPi"]
Description = "树莓派4B安装raspi-firmware，运行 raspi-config。"
Tags = ["RasPi", "guide", "I2C", "开源", "guide", "raspberrypi"]
+++

> 网友问我，如何在服务器版本安装 ``raspi-config`` 这个GUI工具。 就整理了这个教程出来。

- **step 1**    
  新增一个 apt 的 sources 文件 
  
  ```shell
  $ mkdir -p /etc/apt/keyrings

  $ curl -fsSL https://mirrors.tuna.tsinghua.edu.cn/raspberrypi/raspberrypi.gpg.key | gpg --dearmor -o /etc/apt/keyrings/raspi.gpg

  $ echo \
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/raspi.gpg] https://mirrors.tuna.tsinghua.edu.cn/raspberrypi \
    $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/raspi.gpg > /dev/null
  ```
  
  这里使用的 清华的源 ```https://mirrors.tuna.tsinghua.edu.cn/raspberrypi```
  
  以前的 app-key 的指令方法，现在已经升级了哦。会不成功。

- **step 2**    
  
  安装 ``raspi-firmware`` 包。
  ```shell
  $ apt update
  $ apt install raspi-firmware
  ```
- **step 3**    

  执行指令 ``raspi-config``  就会出现 GUI 界面了。

  ![](https://hks3.7y2.org/myimg02/images/202211151848749.png)
