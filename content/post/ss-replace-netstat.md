---
title: "使用 ss 命令代替 netstat"
date: 2020-06-03T04:25:15+08:00
draft: false
menu: "main"
Categories: ["Days", "Guide", "solutions"]
Description: "Linux 使用ss命令代替 netstat"
Tags = ["linux", "debian", "solutions", "Guide", "ss", "netstat"]
---

`ss` 是Socket Statistics的缩写。

顾名思义，`ss` 命令可以用来获取socket统计信息，它可以显示和netstat类似的内容。但ss的优势在于它能够显示更多更详细的有关TCP和连接状态的信息，而且比netstat更快速更高效。

### 和netstat说再见的原因

当服务器的socket连接数量变得非常大时，无论是使用netstat命令还是直接cat /proc/net/tcp，执行速度都会很慢。可能你不会有切身的感受，但请相信我，当服务器维持的连接达到上万个的时候，使用netstat等于浪费 生命，而用ss才是节省时间。

天下武功唯快不破。ss快的秘诀在于，它利用到了TCP协议栈中tcp_diag。tcp_diag是一个用于分析统计的模块，可以获得Linux 内核中第一手的信息，这就确保了ss的快捷高效。当然，如果你的系统中没有tcp_diag，ss也可以正常运行，只是效率会变得稍慢。（但仍然比 netstat要快。）

### 用数据说话

为了让你更坚决的和netstat说再见，列举一些测试数据，以便证明ss的确名不虚传。

当服务器维持3万个socket连接，Admin需要计算具体的连接数量时，不同情况的耗时如下：

```shell
netstat -at | wc  耗时 15.60 秒
ss -atr     | wc  耗时  5.40 秒（未利用tcp_diag）
ss -atr     | wc  耗时  0.47 秒（利用tcp_diag）
```

### 好马配上好鞍

几乎所有的Linux系统都会默认包含`netstat`命令，但并非所有系统都会默认包含ss命令。`netstat` 命令是 `net-tools` 工具集中的一员：

```shell
# apt-cache show net-tools
Package: net-tools
Version: 1.60+git20180626.aebd88e-1
```

而ss命令是iproute2工具集中的一员：

```shell
# apt-cache show iproute2
Package: iproute2
Version: 4.20.0-2
```

如果你无法使用ss命令，那么可能是缺少了 `iproute2` ，需要安装一下：

```shell
# apt install -y iproute2
```

从某种意义上说，`iproute2` 工具集几乎可以替代掉 `net-tools` 工具集，具体的替代方案是这样的：

|用途|net-tool（被淘汰）|iproute2|
|---|---|---|
|路由表|route|ip route|
|邻居|arp|ip neigh|
|VLAN|vconfig|ip link|
|隧道|iptunnel|ip tunnel|
|组播|ipmaddr|ip maddr|
|统计|netstat|ss|


### ss 功能说明

想获得ss命令的帮助信息和版本信息，都非常简单，你应该可以猜到的：
```
# ss -h //获得帮助信息，直接man ss会更全面
# ss -v //取得版本号，-V也支持
ss utility, iproute2-ss190107
```


#### 常用命令

```
ss -l 显示本地打开的所有端口
ss -pl 显示每个进程具体打开的socket
ss -t -a 显示所有tcp socket
ss -u -a 显示所有的UDP Socekt
ss -o state established '( dport = :smtp or sport = :smtp )' 显示所有已建立的SMTP连接
ss -o state established '( dport = :http or sport = :http )' 显示所有已建立的HTTP连接
ss -x src /tmp/.X11-unix/* 找出所有连接X服务器的进程
ss -s 列出当前socket详细信息

```

详细的使用方法，还要在实际中多多的摸索。


参考：

[ss command: Display Linux TCP / UDP Network/Socket Information](https://www.cyberciti.biz/tips/linux-investigate-sockets-network-connections.html)