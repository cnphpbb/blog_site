+++
date = "2017-01-16T13:18:39+08:00"
title = "VPS Debian8 删除一些没用的服务的方法"
draft = false
menu = "main"
Categories = ["vps", "Debian8","solutions"]
Description = "VPS-Debian8系统删除exim4,rpcbind等等服务的方法。"
Tags = ["vps", "linux", "debian","solutions","exim4","rpcbind","nfs"]
+++

本人喜欢使用linux的debian发行版本。也用过其他的发行版本。  
现在使用的vps是[ufovps](https://www.ufovps.com/)出售的香港云服务器。  
安装的是linux的Debian8版本。  
安装好后远程登陆vps后查看有如下几个服务已经安装并在运行。  

> 
* exim4
* rpcbind
* nfs+portmap 
 
以上这些服务我都不使用，需要关闭。方法如下：

#### 删除 `exim4` 服务

`exim4`服务  开放 `port :25` 大家都知道这个端口做啥都吧！

```
$ apt-get purge --auto-remove exim4 exim4-base exim4-config exim4-daemon-light
$ rm -rf /var/log/exim4/
```

#### 删除 `rpcbind` 服务

 `rpcbind` 服务  此服务开启`port :111` 这个端口。

```
$ apt-get purge --auto-remove rpcbind
```

#### 删除`nfs`服务

```
$ apt-get purge --auto-remove nfs-kernel-server nfs-common portmap
```

如果不删除只是关闭相关服务，可以如下执行


```
$ systemctl disable nfs-common
$ systemctl disable rpcbind
$ systemctl disable exim4
$ systemctl stop nfs-common
$ systemctl stop rpcbind
$ systemctl stop exim4
```

>PS：几句废话。  
1. debian8 已经开始用systemd来做系统的启动服务。  
2. systemd 要比sysv要先进很多。  
3. remove没用的服务也是安全保证吧。  